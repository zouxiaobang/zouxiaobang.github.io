---
title: AIDL学习
date: 2016-09-24 13:15:50
tags: [android, aidl]
categories: [Android, Java]
---

## AIDL介绍

Android Interface Definition Language，android接口定义语言。

你可以定义能够让Client和Service之间建立起通信的程序接口。在Android里面，进程与进程间是不能通信和访问各自的内存。而AIDL的存在，则是为了解决这个问题。

![官方文档](/img/aidl/aidl1.png)

什么时候需要AIDL，这是个问题，因为AIDL的编写比较复杂，所以你应该在需要的时候才去使用它，例如当其他的App通过跨进程通信来访问你的Service而且你的Service需要处理多个线程。不然的话你就使用继承Binder或使用Messenger的方法。


## AIDL的使用

### 1、创建bean类

首先我们要创建一个实现了Parcelable的Javabean类

``` bash
public class Book implements Parcelable{
	private String name;
	private int price;

	public Book(){}

	protected Book(Parcel in){
		name = in.readString();
		price = in.readInt();
	}

	public static final Creator<Book> CREATOR = new Creator<Book>(){
		public Book createFromParcel(Parcel in){
			return new Book(in);
		}

		public Book[] newArray(int size){
			return new Book[size];
		}
	}

	public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int describeContents(){
    	return 0;
    }

    public void writeToParcel(Parcel parcel, int i){
    	parcel.writeString(name);
    	parcel.writeInt(price);
    }

    /**
    * 自己写的
    */
    public void readFromParcel(Parcel dest){
    	//这里的顺序需要和writeToParce()中的顺序一致
    	name = dest.readString();
    	price = dest.readInt();
    }
}
```

其中需要注意的是readFromParce()这个方法是自己写上去的，是为了在.aidl文件中可以使用out或inout来定向tag。

### 2、创建.aidl文件

怎么新建AIDL文件呢？当然，AndroidStudio已经帮你领到家门口了。直接右键new --> AIDL --> AIDL File

这里如果你填写的AIDL名字和上面的javabean类名相同的会是不行的，你需要先写一个其他的名字，然后再reName一下，改为javabean的类名。

OK，然后我们再来看看AIDL文件的内容：
``` bash
package com.example.zouxiaobang.aidlex.aidl;

parcelabel Book;
```

这个文件的作用就是为了引入一个序列化对象Book，只有这样Book对象才能被其他的AIDL文件所使用。
这里的包名应该和Book.java的包名一致。

``` bash
package com.example.zouxiaobang.aidlex.aidl;

import com.example.zouxiaobang.aidlex.aidl.Book;

interface BookManager {
    List<Book> getBooks();

    void addBook(in Book book);
}
```

如果需要使用到非默认的类型的时候，就需要import了，例如上面的Book。而在传入参数的时候，需要使用到定向tag。需要记住的是aidl文件其实就是一个接口类，所以你的那些方法也就是将来使用与进程间通信所需要的方法。
写完之后clean一下project，如果没有错就对了。这时系统会默认给我们生成对应的.java文件，所以我们才可以使用该接口里的方法。

### 3、服务端的代码

我们刚刚所做的只是建立了接口，我们都知道，接口的方法是不能调用，这就需要我们去实现这些方法了。这个步骤就在我们的服务端来完成。(这里我们的服务端就是Service)

``` bash
public class AIDLService extends Service {
    private List<Book> mBooks = new ArrayList<>();

    private final BookManager.Stub mBookManager = new BookManager.Stub() {
        @Override
        public List<Book> getBooks() throws RemoteException {
            if (mBooks == null){
                return new ArrayList<Book>();
            }
            return mBooks;
        }

        @Override
        public void addBook(Book book) throws RemoteException {
            synchronized (AIDLService.class){
                if (mBooks == null){
                    mBooks = new ArrayList<>();
                }
                if (book == null){
                    book = new Book();
                    book.setPrice(12);
                }

                if (!mBooks.contains(book)){
                    mBooks.add(book);
                }
            }
        }
    };

    @Override
    public void onCreate() {
        super.onCreate();
        Book book = new Book();
        book.setName("Android");
        book.setPrice(75);
        mBooks.add(book);
    }

    @Override
    public IBinder onBind(Intent intent) {
        return mBookManager;
    }
}
```

这里最重要的就是对BookManager这个aidl接口的实现，注意它的实现并不是普通interface那样直接使用接口名，而是BookManger.Stub。我们在Stub实现类中将方法进行逻辑实现。而我们以前说过onBind()方法返回的IBinder对象也是一个AIDL接口，所以我们可以直接让onBind()方法返回AIDL的实现类，即mBookManager。

好了，我们服务端的代码写完了，就先开启它。可以在Activity中使用startService()来启动。

### 4、客户端的代码

新建一个App作为客户端，这才能够体现出AIDL的跨进程通信的功能。将服务端的aidl文件都复制到客户端中，记得包名也得是相同的，而且Book这个bean也要复制过去，包名一样得相同。

``` bash
public class MainActivity extends Activity {
    private List<Book> mBooks;
    private BookManager mBookManager;
    private boolean isBound;

    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder service) {
            mBookManager = BookManager.Stub.asInterface(service);
            isBound = true;

            if (mBookManager != null){
                try {
                    mBooks = mBookManager.getBooks();
                    Log.i("books", mBooks.toString());
                } catch (RemoteException e) {
                    e.printStackTrace();
                }

            }
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {
            isBound = false;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    /**
     * 点击事件:点击添加
     * @param view
     */
    public void addBook(View view){
        if (!isBound){
            Intent intent = new Intent();
            intent.setAction("com.example.zouxiaobang.aidlex.service");
            intent.setPackage("com.example.zouxiaobang.aidlex");
            bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
        }

        if (mBookManager == null)
            return;

        Book book = new Book();
        book.setName("App");
        book.setPrice(12);

        try {
            mBookManager.addBook(book);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(mConnection);
    }
}
```

此时同时打开两个APP，点击客户端的按钮时，即可看到通信的内容。














