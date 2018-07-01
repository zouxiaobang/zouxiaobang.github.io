---
title: IntentService--拥有独立线程的后台
date: 2016-09-21 19:42:03
tags: [android,Service]
categories: Android
---

## IntentService
同样地，我们先来看看官方对IntentService的介绍

![](/img/service/intentservice1.png)

1、IntentService继承自Service，但它是用来处理有异步请求的
2、客户端也就是其他的组件通过startService(Intent)方法来发送请求，然后它就会根据这个请求来决定是否开始处理
3、使用一个线程处理所有的Intent(当然，它并不是同时进行的)，但它工作完了之后就会自己关闭掉自己。

![](/img/service/intentservice3.png)

所有的请求都在一个线程中进行，也许有些请求会花很长的时间，但每次只能处理一个请求。
这就是IntentService的缺点，它虽然是异步的，但每次都只能够对一个请求进行处理(工作队列的模式)。但这其实满足了很多对于网络的请求。

## IntentService的使用

创建一个继承自IntentService的类
``` bash
public class MyIntentService extends IntentService{
	public MyIntentService(){
		super("MyIntentService");
		//...
	}

	public void onHandleIntent(Intent intent){
		//TODO handle your requests
	}
}
```

这是最简单的IntentService的类范例

![](/img/service/intentservice4.png)

构造方法：在官方文档中的构造方法也是有一个String的参数，这是一个很奇怪的参数，如果你的构造方法有这个参数，那么你就需要有一个构造器。如果没有构造器，那么你的IntentService就会出错。所以一般按照上面的方法来写。super()里面的参数是IntentService开启的线程的线程名。

![](/img/service/intentservice5.png)

onHandleIntent(Intent intent)：这是一个抽象方法，所以在编写自己的IntentService类的时候需要重写该方法。在该方法中对Intent请求进行处理，但有可能其中某个请求需要消耗很多的时间，这会拖延了其它请求的进行(当然不会拖延其它Service的请求啦)。而且在这个方法中，当你将所有的请求都处理完了，IntentService会自动stop自己，而不用去调用stopSelf()方法。
其中的Intent参数是startService(Intent)方法中的参数传过来的，所以它带来的是其它组件传过来的意图。


其实，IntentService还有几个方法，只是在封装的时候都已经被实现了，所以不需要我们进行重写。但还是有一个方法需要注意的，因为它涉及一个问题，你可能看这篇文章的时候一直有一个问题，就是看了这么久，都不知道IntentService是怎么和Activity等的组件进行交互的。在Service中我们通过绑定来解决这个问题，那么IntentService行不行呢？官方文档第一句话就已经说行了。哪里，我怎么没看到？你看，它说啊，IntentService是继承自Service的，那父类有的public方法，子类自然也有。所以重写onBind(Intent)方法来与之进行数据的交流。

onBind()的用法和Service的用法一致，这里就不再贴代码了，如果不清楚可以去看我的上一篇博文。
这里要注意的有两个方面：
* IntentService的开启方式就是startService()方法，而bindService()也就仅仅只是绑定而已，并没有开启，所以每次要是IntentService具有交流性，都要将这两个方法都使用。因为bindService()方法并没有去调用onHandleIntent()方法。
* 在onHandleIntent()方法中，请求的处理是一件接一件的，例如你在Activity中同时发送两个Intent给IntentService，那么onHandleIntent()就收到两个Intent，它根据Intent携带的信息来判别不同的请求，然后对这些请求进行处理：intent1 start --> intent1 end --> intent2 start --> intent2 end 也就是说第一个请求还没处理好，第二个请求你就别想动。




























