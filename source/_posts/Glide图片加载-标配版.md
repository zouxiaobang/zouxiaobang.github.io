---
title: Glide图片加载(标配版)
date: 2016-09-10 15:11:18
tags: [android,框架,Glide]
categories: Android
---
## 前言
前面，我已经有一篇介绍图片加载（[Volley](https://zouxiaobang.github.io/2016/08/26/Volley的简单使用/)）的博客了，为什么还要再写一篇呢？
原因就是Volley对于网络加载那是杠杠的不用说，但它的缺点也局限了它的使用，那就是Volley并不能加载本地的图片。

先说说Glide的好处吧：
	* 能异步加载网络图片和本地图片
	* 支持Memory和Disk缓存
	* 对图片的滚动列表尽可能的流畅
	* 可以使用BitmapPool对Bitmap进行复用
	* 支持gif和webp格式的图片

最重要的一点，是google推荐使用它。

## 使用
### 准备
#### 1、下载glide的jar包
[glide-3.6.1.jar](http://download.csdn.net/download/u011241872/9257813)
#### 2、导入support-v4包
如果你以为只导入glide包就结束了，那就太天真了。没错，glide需要与support-v4合作，所以需要再导入该包。
在app的build.gradle中修改
``` bash
dependencies{
	complie 'com.android.support:appcompat-v7:24.2.1'
}
```

### 使用
#### 1、最普通的应用
``` bash
Glide.with(context)
	.load("url")
	.into(imageView);
```
Glide最大的亮点在于没有多余的东西，很多东西都给你封装了，加载图片，只需调用一句话。

分析：
```
with(context)
```
该方法可传入的参数有--Actvity，--Fragement，--FragementActivity，--Context
为什么要这么多的参数呢，Picasso只能传入Context。但Glide却推荐使用Activity和Fragement，为什么呢？因为这样Glide的生死将取决于该Activity或Fragement的生命周期。例如当Activity为pause时，Glide暂停加载，当Activity为resume时，Glide恢复加载。
```
load("url")
```
该方法能传入的url（String类型或int类型）就多了，有一下几种
	* 从资源中加载：R.drawable.pic / R.mipmap.ic;
	* 从文件中加载：File file = new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES), "Running.jpg");
	* 从Uri加载：Uri uri = Uri.parse(ANDROID_RESOURCE + context.getPackageName() + FOREWARD_SLASH + R.mipmap.ic);
	* 从网络中加载："http://..."

#### 2、Glide的缓存
在使用列表展示图片时，我们往往需要使用到缓存来存储图片(用太久的手机就没办法保证加载很快了，比较卡)，我们也不用去考虑该缓存空间的大小，Glide是很智能的，它隐藏提供了一个缓存的大小。
缓存的来源：
	* 内存--Memory
	* 磁盘--Disk
	* 网络--Net
	速度从快到慢排列

#### 3、加载中图片和加载失败图片
这个基本是每个网络加载图片的标配了
``` bash
Glide.with(context)
	.load("url")
	.placeholder(R.drawable.default_pic)
	.error(R.drawable.error_pic)
	.into(imageView);
```
其中placeholder()方法加载的是加载中图片
error()方法加载的是加载失败的图片

### 以上为标配

高配请阅读下篇
[Glide图片加载-高配版](https://zouxiaobang.github.io/2016/09/11/Glide图片加载-高配版/)

