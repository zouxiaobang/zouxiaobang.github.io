---
title: Otto的最简单用法
date: 2016-09-17 11:24:31
tags: [Otto,android,框架]
categories: Android
---
## Otto简介
	Otto是一个事件总线的框架，简单地说就是它就是用于事件的分发，从而简化了组件之间的通讯。

最经典的组件通讯有下面三种
* Activity <==> Activity : startActivityForResult  onActivityResult
* Activity <==> Fragment : interface
* Fragment <==> Fragment : interface

我们经常会使用到一个场景，就是界面A打开界面B，在B中进行操作后返回A时要求A界面的内容进行更新。例如两个界面都是Activity，我们以前都是使用startActivityForResult()方法来实现这个功能，在A界面通过重写onActivityResult()方法来判断状态和进行更新，因为需要进行状态的判断和逻辑的判断，那么代码量就会随之增大，bug的几率也会悄悄增长。而Otto完美地解决了这个问题，具体怎么解决的下面再说。
先给个otto的jar包
[otto-1.3.8.jar](http://download.csdn.net/download/zhaihaohao1/9500768)

## Otto的简单使用

### 1、获取Bus对象
Bus是整个事件分发机制的重心，顾名思义，Bus就像一辆列车，在几条道路上来回穿梭，到了每一个不同的站都有人要下车。
Ok，它是真的像一辆列车，在一条铁轨上，只能有一辆列车的存在，不然就容易发生碰撞。Bus也一样，那么单例设计就是很有必要的了。
原先：
```
Bus bus = new Bus();
```
单例设计模式
``` bash
public final class BusProvider{
	private static Bus mBus;
	private BusProvider(){}

	public static Bus getInstance(){
		if(mBus == null){
			synchronized(BusProvider.class){
				if(mBus == null){
					mBus = new Bus();
				}
			}
		}
		return mBus;
	}
}
```

### 2、创建一个参数类
这个类就像一个bean类，里面装着我们需要在不同组件间传递的参数。
例如我们要从B中返回给A的是个人的信息
``` bash
public class Person{
	private String name;
	private String sex;
	private int age;

	public void setName(String name){
		this.name = name;
	}
	public String getName(){
		return name;
	}

	//getter and setter...
}
```

### 3、注册Otto
当然，就像其他框架一样，要使用它，就得先注册它。
在需要获得信息或需要刷新的组件注册，例如在A界面注册
``` bash
//将Bus注册到组件中
BusProvider.getInstance().register(this);
//注销Bus
BusProvider.getInstance().unregister(this);
```
当然不是什么地方都可以进行注册和注销的，例如如果在onCreate()方法中注册，则需要在onDestroy()方法中注销，如果在onResume()方法中注册，就需要在onPause()方法中注销。如果不是，那么会出现异常，例如在onResume中注册而不知onPause对其注销，那么从B返回A时会出现异常，提示已经注册过异常。

### 4、接收信息
接收信息就像乘客下车，在上面的例子中，我们就是要在A中接收B关闭时的信息并更新A中的内容，所以接收信息是在A中进行的
``` bash
@Subscribe
public void getTheMessage(Person person){
	Log.i("TAG", person.getName() + " : " + person.getAge());
}
```
使用@Subscribe表示订阅该信息，该标记必须在该组件(即Activity或Fragment)已经注册了Bus的前提下才有效。
方法名称可以自定义，但参数必须是你需要获得的信息的对象，可以是bean，也可以是基本对象或String。例如你的参数是Person，那么你就是订阅了Person这个信息，当这个信息发布出来的时候，这个方法就会被回调。
【特别注意】而该方法有一点需要注意的，就是必须是public，记得，public。这是个沉痛的教训。
你是不是很好奇这样就可以获取信息了吗？别急，接下去看

### 5、发布信息
发布信息就像乘客上车，在这个例子中，我们要在B中将信息发送给A，所以发布信息应该在B中进行
``` bash
btn_send.setOnClickListener(new OnClickListener(){
	@Override
	public void onClick(View view){
		Person person = new Person();
		person.setName("John");
		person.setAge(12);
		person.setSex("Boy");

		//发送Person信息
		BusProvider.getInstance().post(person);
		B.this.finish();
	}
});
```
这是一种发布的操作，是最简单的方法，不用在B中对Bus进行注册，调用post()方法就直接将信息发布出去。
还有一种方法，比较麻烦，叫生产者方法
``` bash
@Produce 
public Person sendPerson(){
	Person person = new Person();
	person.setName("John");
	person.setAge(12);
	person.setSex("Boy");

	return person;
}

btn_send.setOnClickListener(new OnClickListener(){
	@Override
	public void onClick(View view){
		//发送Person信息
		BusProvider.getInstance().register(B.this);
		B.this.finish();
	}
});
```
看代码就知道这个比较麻烦了，使用@Produce来创建一个方法，该方法返回需要发布的信息。
当我们需要发布出去的时候，就注册它。
那你是不是又有疑问了，那如果我在B中@Produce创造多个方法，返回多种对象呢？那A怎么接收？
当然，这也算是个问题，我也认真思考了一下：
因为@Subscribe接收信息的根据参数接收的，所以你需要接收那个对象，就在需要的地方设置该接收方法就好。

## 总结
1 记得方法要public
2 @Subscribe的是订阅者，@Produce的是发布者。
3 register注册在订阅前是必须的，在发布时可以使用不需注册的，若选择需要注册的，则在发布的时候才register




