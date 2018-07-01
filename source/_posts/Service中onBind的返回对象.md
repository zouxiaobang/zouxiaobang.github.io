---
title: Service中onBind的返回对象
date: 2016-09-22 09:47:43
tags: [android, Service]
categories: Android
---

## onBind(Intent intent)

我们都知道这个方法返回一个IBinder对象，该对象是用于与客户端进行通信的。但是IBinder是一个抽象类，它的本质是一个AIDL对象，那么我们应该怎么去创建它呢？

Android给我们提供了三个方法
* 继承Binder类
* 使用Messager类
* 使用AIDL

### 1、继承Binder类

我在介绍Service的bindService()开启Service的时候都是使用继承Binder这个方法来与Client取得联系的，因为这个方法是最简单的。

``` bash
//Service code
private int mCount;
private MyBinder mBinder = new MyBinder();

public class MyBinder extends Binder{
	public int getCount(){
		return mCount;
	}
}

@Override
public IBinder onBind(Intent intent){
	return mBinder;
}
```

client通过bindService()与Service连接时产生的ServiceConnection对象来获取这个MyBinder对象，然后通过操作MyBinder的方法可以与Service取得通信。

``` bash
//Client code
private MyService.MyBinder mBinder;

ServiceConnection mConnection = new ServiceConnection(){
	@Override
	public void onServiceConnected(ComponentName className, IBinder service){
		mBinder = (MyService.MyBinder) service;
	}

	@Override
	public void onServiceDisconnected(ComponentName className){

	}
}

void doConnect(){
	bindService(new Intent(this, MyService.class), mConnection, Context.BIND_AUTH_CREATE);
}

int getCount(){
	return mBinder.getCount();
}

void doUnconnect(){
	unbindService(mConnection);
}

```

### 2、使用Messenger类

当我们提到IPC(跨进程通信)的时候，总会想起一个很高端的名称--AIDL，但是AIDL是一门很深很难的技术，既然太难，就会有替代品。Android使用了一个更为简单的类来代替编写整个AIDL----Messager。

Messenger的核心是有Message和Handler来进行线程间通信的。那么在Service中使用Messenger，就必须有Message和Handler的存在。即使用它的步骤如下：

* 在Service中创建一个Handler对象
* 通过Handler对象创建一个Messager对象--Messenger mMessenger = new Messenger(mHandler);
* 在onBind()中返回一个IBinder--return mMessenger.getBinder();
* 在Client中对Service进行连接
* 在ServiceConnection对象中获取来自Service的Messenger--Messenger rMessenger = new Messenger(service);
* 使用Messenger来发送数据

下面的例子是客户端和后台之间的数据传输
``` bash
public class MessengerService extends Servcie{
	private static final int MSG_SAY_HELLO = 0;

	//本地的Messenger对象
	private Messenger mMessenger;
	//来自Client的Messenger对象
	private Messenger cMessenger;

	//获取来自Client的信息并进行处理
	private Handler mHandler = new Handler(){
		public void handleMessage(Message msg){
			switch(msg.what){
				case MSG_SAY_HELLO:
					cMessenger = msg.replyTo;
					break;
			}
		}
	}

	public IBinder onBind(Intent intent){
		return mMessenger.getBinder();
	}

	public void onCreate(){
		mMessenger = new Messenger(mHandler);
		int num = 0;

		new Thread(){
			public void run(){
				while(num < 100){
					SystemClock.sleep(100);
					num ++;
					Message msg = new Message();
					msg.what = 1;
					Bundle data = new Bundle();
					data.putExtra("num", num);
					msg.setData(data);
					cMessenger.send(msg);
				}
			}
		}.start();
	}
}
```

``` bash
public class MainActivity extends Activity{
	private Messenger mMessenger;
	//来自Service的Messenger对象
	private Messenger rMessenger;

	private Handler mHandler = new Handler(){
		public void handleMessage(Message msg){
			switch(msg.what){
				case 1:
					Log.i("tag", msg.getData().getIntExtra("num") + "");
					break;
			}
		}
	};

	private ServiceConnection mConnection = new ServiceConnection(){
		public void onServiceConnected(ComponentName className, Intent service){
			rMessenger = new Messenger(service);
			mMessenger = new Messenger(mHandler);
		}

		public void onServiceDisconnected(ComponentName className){

		}
	};

	protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        bindService(new Intent(this, MessengerService.class), mConnection, Context.BIND_AUTO_CREATE);
    }

    void getCount(){
    	Message msg = new Message();
    	msg.what = 0;
    	msg.replyTo = mMessenger;
    	try{
    		rMessage.send(msg);
    	} catch(Exception ex){

    	}
    }

    public void onDestroy(){
    	super.onDestroy();
    	unbindService(mConnection);
    }
}
```

### 3、使用AIDL

由于AIDL比较复杂，所以请看下一篇博文







