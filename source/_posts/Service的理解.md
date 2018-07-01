---
title: Service的理解
date: 2016-09-21 16:31:10
tags: [android, Service]
categories: Android
---

## 初识Service
对于Service的使用，应该有很多人都对它很有心得，但我却惊奇地发现，每个人的对Service的理解总是有那么些不同，那么，就说说我的心得吧。

首先，看看官方文档对Service的介绍

![官文第一段](/img/service/service1.png)


Service是一个app组件，用于在处理长时间的操作。所有的Service都必须在AndroidManifest.xml中进行注册。开启Service的方法有两个

* Context.startService();
* Context.bindService();

这两种方法稍下进行详细地介绍，下面继续看官方文档

![官文第二段](/img/service/service2.png)

Service默认是运行在主线程中，如果你想启动例如后台播放的MP3，获取网络之类的操作呢，你就得乖乖地自己开启一个新的线程(PS:spawn为产卵的意思，根据我14年英语生涯的理解，还是解释成开启比较好)。当然，你要是选择了IntentService来作为后台的话，那就更好了，因为它自己本身就拥有一个线程，拥有异步操作的优势。

我百度了一下，Service中能否进行耗时的操作，很多都直接说，不能。但是，官方文档第一句就说它就是为了出来长时间操作而来的，那不是自相矛盾。其实不然，Service确实可以进行耗时操作，只是必须直接创建一个新线程，因为Service默认是在main线程中执行的，那么长时间的操作会阻碍主线程的更新，就会产生异常(ANR)。而对于IntentService则没有这个痛苦，稍后也会对它进行介绍。

## Service的启动

### 1、startService()启动Service

在Java代码中启动Service：

``` 
context.startService(new Intent(context, MyService.class));
```

在AndroidManifest.xml文件中注册Service：

```
<service
    android:name=".MyService"
    android:process=":remote">
</service>
```

其中的android:process=":remote"是新建一个新的线程，线程名叫remote。

接下来我们看看这种方式启动的Service的生命周期

![Unbounded](/img/service/service3.png)

* onCreate()：只会调用一次，即使是多次使用了startService()来开启Service。可以在这个方法里面做一些初始化操作
* onStartCommand(Intent intent, int flag, int startId)：当其它组件通过startService()来开启Service时调用它。这个方法的作用很好地说明了一旦你第一次开启了Service，那么你没有消灭它之前，它是一直存在的，直到你退出整个App。
* onDestroy()：这个不用多说了吧，当你调用stopService()方法来停止后台运行时，就会调用它。

停止后台的代码(两种)

在Activity等组件中调用
```
context.stopService(Intent name); 
```

在Service自身中调用
```
stopSelf();
//stopSelf(int startId);
```

### 2、bindService()绑定Service

![Bounded](/img/service/service4.png)


使用这种方法绑定Service，必须在Service类中重写onBind()方法，而onBind()返回一个IBinder对象。该IBinder对象的作用就是用于与其它组件进行通信的。

在Java中的代码
``` bash
private MyService.MyBinder mBinder;
private boolean mIsBound;

private ServiceConnection mConnection = new ServiceConnection(){
	public void onServiceConnected(ComponentName className, IBinder service){
		mBinder = (MyService.MyBinder)service;
		//TODO Binder class in MyService ----fuction
		
	}

	public void onServiceDisconnected(ComponentName className){

	}
};

void doBindService(){
	bindService(new Intent(this, MyService.class), mConnection, Context.BIND_AUTO_CREATE);
	mIsBound = true;
}

void doUnbindService(){
	if(mIsBound){
		unbindService(mConnection);
		mIsBound = false;
	}
}

@Override
protected void onDestroy(){
	super.onDestroy();
	doUnbindService();
}
```

其中我们需要创建一个连接--ServiceConnection，将该连接对象传入到bind中，此时如果与Service绑定成，则这个ServiceConnection就会回调onServiceConnected()方法，否则回调onServiceDisconnected()，成功绑定后，我们可以通过onServiceConnected()的参数service来获取后台的IBinder对象，从而操作后台创建的IBinder类中的动作。

现在我们来看看这个Service类

``` bash
public class MyService extends Service{
	private int mCount = 0;
	public class MyBinder extends Binder{
		public int getCount(){
			return mCount;
		}
	}
	private MyBinder mBinder = new MyBinder();

	@Override
	public IBinder onBind(Intent intent){
		return mBinder;
	}

	@Override
	public void onCreate(){
		super.onCreate();

		new Thread{
			public void run(){
				while(true){
					count ++;
					SystemClock.sleep(100);
				}
			}
		}.start();
	}
}
```

注意，当使用bindService()方法绑定Service时，不能在AndroidManifest.xml文件中写入android:process=":remote"，而只能在代码中new一个Thread。

这样之后我们可以在Activity中通过mBinder的getCount来获取Service中的信息，而Service也可以通过onBind()方法中的Intent参数来获取Activity传递过来的参数。


[下一篇](https://zouxiaobang.github.io/2016/09/21/IntentService-拥有独立线程的后台/)我将介绍google更加推荐的IntentService。






