---
title: Binder详解
date: 2017-07-26 21:42:17
tags: [android, java]
categories: [Android, Java]
---
## Binder详解

Binder的目的是为了实现跨进程间通讯--IPC

### 1、Binder的框架结构
![Binder的框架结构图](/img/binder/binder1.jpg)
a、Client通过bindService()方法去绑定一个来自Service中注册的一个Service服务（其中Client和Service来自不同的进程）
b、系统收到来自Client的bindService()方法后，就调用了Service的onBind()方法进行绑定，该方法返回的是一个IBinder对象，该对象是在Service中通过Stub桩去实现的一个代理类对象
c、并把该对象返回给Client
d、Client从ServiceConnection中的onServiceConnected()方法中接收到该代理对象
e、在Client中通过该代理对象可以调用这个代理对象的方法

### 2、Binder在Service和Client中的使用
即在Service中创建一个Binder对象以给Client调用
``` bash
private AtomicBoolean mIsServiceDestroyed = new AtomicBoolean(false);
private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<>();
private RemoteCallbackList<IOnNewBookArrivedListener> mListenerList = new
            										RemoteCallbackList<>();
Binder mBinder = new BookManager.Stub(){
	@Override
    public List<Book> getBookList() throws RemoteException {
        return mBookList;
    }

    @Override
    public void addBook(Book book) throws RemoteException {
        mBookList.add(book);
    }

    @Override
    public void registerListener(IOnNewBookArrivedListener listener) throws RemoteException {
        mListenerList.register(listener);
    }

    @Override
    public void unregisterListener(IOnNewBookArrivedListener listener) throws RemoteException {
        mListenerList.unregister(listener);
    }
}
```

其中重写的方法例如getBookList()等方法，都是为了给Client调用。在Client中，调用这些方法就是跨进程通信。
这就引出了Binder的两个特性：
**完成特定的功能**
**跨进程传输**
这两个功能的完成，取决于Binder类中的关键方法：
``` bash
//伪代码（Stub）
class Binder implements IBinder{
    void attachInterface(IInterface plugs, String descriptor);
    IInterface queryLocalInterface(String descriptor);
    boolean onTransact(int code, Parcel data, Parcel reply, int type);

    final class BinderProxy implements IBinder{
        ...
    }

    ...
}
```
**系统给每一个实现IBinder接口的对象提供跨进程传输的功能。**
**attachInterface()**方法通过传入一个键值对--descriptor:plugs（Map<String, IInterface>），来将plugs（IInterface服务对象）和descriptor绑定在一起。
**queryLocalInterface()**方法通过descriptor这个key来获取plugs这个value.
看看以下的代码：
``` bash
//伪代码
public static abstract class Stub extends Binder implememnts BookManager{
    private static final String DESCRIPTOR = "BookManager";

    public Stub(){
        attachInterface(this, DESCRIPTOR);
    }

    ....

    public static BookManger asInterface(IBinder obj){
        if(obj == null){
            return null;
        }

        IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
        if((iin != null) && (iin instanceof BookManager)){
            reutrn (BookManager)iin;
        }

        return new Stub.Proxy(iin);
    }
}
```
其中，当创建Stub类的时候（在Service中创建），会调用attachInterface()方法将该Binder对象进行绑定。
在Client中通过bindService()方法绑定Service时，会调用该IBinder对象的asInterface()来获取到BookManager对象。

### 3、Binder对象在Client中的调用流程
当Client从Service中获取到一个Binder对象之后转换成BookManager(自己定义的服务)，那么就可以在Client中调用该BookManager的方法。
``` bash
//伪代码
mBookManager.addBook(book);
```
此时这个addBook()方法并不是Client这个进程中执行，而是在Service的进程中执行的。

在Service中的onBind()方法中返回了一个Binder对象，该Binder对象就是第二点中的第一段代码中所写的那个mBinder，它是一个BookManager.Stub，**但在onBind()方法中返回时已经将该BookManger对象给强制转换为IBinder对象了**。
所以在Client中调用asInterface()方法时，该方法内部所调用的queryLocalInterface()方法获取到的iin对象是一个IBinder对象，所以它已经不属于BookManager对象了，所以直接创建一个代理类。
如果是在同一个进程中，则没有通过onBind()方法，那么这个BookManger.Stub对象就不会转换为IBinder对象，那么就会执行、、、reutrn (BookManager)iin;、、、这段代码。

在跨进程中，我们已经知道了Client中返回的是一个Stub。看一下Stub的完整代码
``` bash
public static abstract class Stub extends Binder implements BookManager{
    public static final String DESCRIPTOR = "BookManager";

    public Stub(){
        attactInterface(this, DESCRIPTOR);
    }

    public IBinder asBinder(){
        return this;
    }
    public static BookManager asInterface(IBinder obj){
        if(obj == null){
            return null;
        }

        IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
        if((iin != null) && (iin instanceof BookManager)){
            return (BookManger)iin;
        }

        return new Stub.Proxy(iin);
    }

    public boolean onTransact(int code, Parcel data, Parcel reply, int type) throws RemoteException{
        switch(code){
            case INTERFACE_TRANSACTION:
                reply.writeString(DESCRIPTOR);
                return true;
            case TRANSACT_addBook:
                data.enfoceInterface(DESCRIPTOR);
                Book book;
                if(data.readInt() != 0){
                    book = Book.CREATOR.createFromParcel(data);
                } else{
                    book = null;
                }

                this.addBook(book);
                reply.writeNoException;
                return true;
            case TRANSACT_getBookList:
                data.enfoceInterface(DESCRIPTOR);
                List<Book> bookList = this.getBookList();
                reply.writeNoException();
                reply.writeTypeList(bookList);
                return true;
        }
    }

    public static class Proxy extends BookManager{
        private IBinder mRemote;
        public Proxy(IBinder remote){
            this.mRemote = remote;
        }

        public String getInterfaceDescriptor(){
            return DESCRIPTOR;
        }

        public IBinder asBinder(){
            return mRemote;
        }

        public void addBook(Book book) throws RemoteException{
            Parcel data = Parcel.obtain();
            Parcel reply = Parcel.obtain();

            try{
                data.writeInterfaceToken(DESCRIPTOR);
                if(book != null){
                    data.writeInt(1);
                    book.writeToParcel(data, 0);
                } else{
                    data.writeInt(0);
                }

                mRemote.transact(TRANSACT_addBook, data, reply, 0);
                reply.readException();
            } finally{
                data.recycle();
                reply.recycle();
            }
        }

        public List<Book> getBookList() throws RemoteException{
            Parcel data = Parcel.obtain();
            Parcel reply = Parcel.obtain();
            List<Book> result;
            try{
                data.writeInterfaceToken(DESCRIPTOR);
                mRemote.transact(TRANSACT_getBookList, data, reply, 0);
                reply.readException();
                result = reply.createTypedArrayList(Book.CREATOR);
            } finally{
                data.recycle();
                reply.recycle();
            }

            return result;
        }

        static final int TRANSACT_addBook = IBinder.FIRST_CALL_TRANSACTION + 0;
        static final int TRANSACT_getBookList = IBinder.FIRST_CALL_TRANSACTION + 1;
    }
}

public void addBook(Book book) throws RemoteException;
public List<Book> getBookList() throws RemoteException;
```
在Service中使用匿名内部类创建一个Binder对象时，必须重写addBook(Book book)和getBookList()。因为该Stub时一个抽象类。
在Client中获取到的是Stub中Proxy对象，所以在Client中调用方法时是调用了代理类中的方法。
在代理类的方法中，首先要创建两个用于进程间传递的序列化对象data和reply。然后调用mRemote对象的transact()方法（此时transact()方法会让Stub回调onTransact()方法。onTransact()方法执行完之后会把reply返回给对应的方法），根据reply获取要返回的信息并将该信息返回给调用者。
其中的重点在于回调了onTransact()方法。因为代理类在Client进程中，而执行的方法是在Service中，所以需要调用transact()。

再看看onTransact()方法，在该方法中调用了对应的方法，虽然这些方法在Stub类中都是抽象的方法，但在Service中创建Stub对象时重写了这些方法，所以在Client中调用到的就是Service中定义的方法。

### 4、BookManager服务类完整代码代码
``` bash
public interface IBookManager extends IInterface{
    public static abstract class Stub extends Binder implements IBookManager{
        private static final String DESCRITPTOR = "IBookManager";

        public Stub(){
            attactInterface(this, DESCRIPTOR);
        }

        public IBinder asBinder(){
            return this;
        }

        public static IBookManager asInterface(IBinder obj){
            if(obj == null){
                return null;
            }

            IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if((iin != null) && (iin instanceof IBookManger)){
                return (IBookManager) iin;
            }

            return new Stub.Proxy(obj);
        }

        public boolean onTransact(int code, Parcel data, Parcel reply, int type){
            switch(code){
                case INTERFACE_TRANSACTION:
                    reply.writeString(DESCRIPTOR);
                    return true;
                case TRANSACT_addBook:
                    data.enforeInterface(DESCRIPTOR);
                    Book book;
                    if(data.readInt() != 0){
                        book = Book.CREATOR.createFromParcel(data);
                    } else{
                        book = null;
                    }
                    this.addBook(book);
                    reply.writeNoException();
                    return true;
                case TRANSACT_getBookList:
                    data.enforeInterface(DESCRIPTOR);
                    List<Book> result = this.getBookList();
                    reply.writeNoException();
                    reply.writeTypeList(result);
                    return ture;
                case TRANSACT_registerListener:
                    data.enforeInterface(DESCRIPTOR);
                    IOnNewBookArrivedListener listener;
                    listener = IOnNewBookArrivedListener.Stub.asInterface(data);
                    this.registerListener(listener);
                    reply.writeNoException();
                    return true;
                case TRANSACT_unregisterListener:
                    data.enforceInterface(DESCRIPTOR);
                    IOnNewBookArrivedListener listener1 ;
                    listener1 = IOnNewBookArrivedListener.Stub.asInterface(data);
                    this.unregisterListener(listener1);
                    reply.writeNoException();
                    return true;
            }

            return super.onTransact(code, data, reply, type);
        }

        private static class Proxy implements IBookManager{
            public Binder mRemote;

            public Proxy(Binder remote){
                this.mRemote = remote;
            }

            public String getInterfaceDescription(){
                return DESCRIPTOR;
            }

            public IBinder asBinder(){
                return mRemote;
            }

            public void addBook(Book book) throws RemoteException{
                Parcel data = Parcel.obtain();
                Parcel reply = Parcel.obtain();

                try{
                    data.writeInterfaceToken(DESCRIPTOR);
                    if(book != null){
                        data.writeInt(1);
                        book.writeToParcel(data, 0);
                    } else {
                        data.writeInt(0);
                    }

                    mRemote.transact(TRANSACT_addBook, data, reply, 0);
                    reply.readException();
                } finally{
                    data.recycle();
                    reply.recycel();
                }
            }

            public List<Book> getBookList() throws RemoteException{
                Parcel data = Parcel.obtain();
                Parcel reply = Parcel.obtain();
                List<Book> result;
                try{
                    data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(TRANSACT_getBookList, data, reply, 0);
                    reply.readException();
                    result = reply.createTypedArrayList(Book.CREATOR);
                } finally{
                    data.recycle();
                    reply.recycle();
                }

                return result;
            }

            public void registerListener(IOnNewBookArrivedListener listener) throws RemoteException{
                Parcel data = Parcel.obtain();
                Parcel reply = Parcel.obtain();

                try{
                    data.writeInterfaceToken(DESCRIPTOR);
                    data.wirteStrongBinder((listener == null)? null:listener.asBinder());
                    mRemote.transact(TRANSACT_registerListener, data, reply, 0);
                    reply.readException();
                } finally{
                    data.recycle();
                    reply.recycle();
                }
            }

            public void unregisterListener(IOnNewBookArrivedListener listener) throws RemoteException{
                Parcel data = Parcel.obtain();
                Parcel reply = Parcel.obtain();

                try{
                    data.writeInterfaceToken(DESCRIPTOR);
                    data.writeStrongBinder((listener == null)? null:listener.asBinder());
                    mRemote.transact(TRANSACT_unregisterListener, data, reply, 0);
                    reply.readException();
                } finally{
                    data.recycle();
                    reply.recycle();
                }
            }

            private static final int TRANSACT_addBook = FIRST_CALL_TRANSACTION + 0;
            private static final int TRANSACT_getBookList = FIRST_CALL_TRANSACTION + 1;
            private static final int TRANSACT_registerListener = FIREST_CALL_TRANSACTION + 2;
            private static final int TRANSACT_unregisterListener = FIREST_CALL_TRANSACTION + 3;
        }
    }


    public void addBook(Book book) throws RemoteException;
    public List<Book> getBookList() throws RemoteException;
    public void registerListener(IOnNewBookArrivedListener listener) throws RemoteException;
    public void unregisterListener(IOnNewBookArrivedListener listener) throws RemoteException; 
}

```

``` bash
public interface IOnNewBookArrivedListener extends IInterface{
    public static abstract class Stub extends Binder implements IOnNewBookArrivedListener{
        private static final String DESCRIPTOR = "IOnNewBookArrivedListener";
        public Stub(){
            attachInterface(this, DESCRIPTOR);
        }

        public IBinder asBinder(){
            return this;
        }

        pubilc static IOnNewBookArrivedListener asInterface(IBinder obj){
            if(obj == null){
                reutrn null;
            }

            IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if((iin != null) && (iin instanceof IOnNewBookArrivedListener)){
                return (IOnNewBookArrivedListener)iin;
            }

            return new Stub.Proxy(obj);
        }

        public boolean onTransact(int code, Parcel data, Parcel reply, int type) throws RemoteException{
            switch(code){
                case INTERFACE_TRANSACTION:
                    data.writeString(DESCRIPTOR);
                    return true;
                case TRANSACT_onNewBookArrived:
                    data.enforceInterface(DESCRIPTOR);
                    Book book;
                    if(data.readInt() != null){
                        book = Book.CREATOR.createFromParcel(data);
                    } else{
                        book = null;
                    }

                    this.onNewBookArrived(book);
                    reply.writeNoException();
                    return true;
            }
            return super.onTransact(code, data, reply, type);
        }

        private static class Proxy implements IOnNewBookArrivedListener{
            private IBinder mRemote;
            public Proxy(IBinder obj){
                this.mRemote = obj;
            }

            public String getInterfaceDescription(){
                return DESCRIPTOR;
            }

            public IBinder asBinder(){
                return mRemote;
            }

            public void onNewBookArrived(Book book) throws RemoteException{
                Parcel data = Parcel.obtain();
                Parcel reply = Parcel.obtain();

                try{
                    data.writeInterfaceToken(DESCRIPTOR);
                    if(book != null){
                        data.writeInt(1);
                        data.writeToParcel(Book.CREATOR);
                    } else{
                        data.writeInt(0);
                    }

                    mRemote.transact(TRANSACT_onNewBookArrived, data, reply, 0);
                    reply.readException();
                } finally{
                    data.recycle();
                    reply.recycle();
                }
            }
        }

        private static final String TRANSACT_onNewBookArrived = FIRST_CALL_TRANSACTION + 0;
    }

    public void onNewBookArrived(Book book) throws RemoteException;
}
```







