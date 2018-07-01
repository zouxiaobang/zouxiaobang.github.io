---
title: Glide图片加载(高配版)
date: 2016-09-11 18:29:45
tags: [android,框架,Glide]
categories: Android
---
## 承接上文
上一篇主要介绍的是Glide的基本使用，而它的更多优点，则在此篇进行简单说明。

### 1、强制修改图片大小
如果你想下载的图片实在太大了呢，那下下来就太占内存了，那么就需要对它的大小进行压缩
``` bash
Glide.with(context)
	.load("url")
	.override(200, 200)
	.into(imageView);
```
这个比较少用，因为有可能会失真，毕竟你不知道图片的比例。

### 2、不失真修改图片大小
不失真，那就用Glide提供的两种操作
	* fitCenter()
	* centerCrop()

根据名字就能猜到这两种操作的区别了：fitCenter是将图片缩小至长边能放进imageView中，有可能没法填满imageView；而centerCrop则是以短边能放进imageView中，而将长边又中间填满，剪掉多出来的部分。
很难理解？上图：
![fitCenter](/img/glide/fitCenter.jpg)		![centerCrop](/img/glide/centerCrop.jpg)

### 3、加载gif格式图片
这个可是glide的重头戏，就连picasso都没有的功能
但它对gif的加载也是很简单，load()方法和加载普通图片是一样的

``` bash
Glide.with(context)
	.load("url")
	.asGif()
	.error(R.drawable.error_pic)
	into(imageView);
```
加载完后在imageView中会自动播放gif
而asGif()是为了检测该url是否是一个gif，如果不是，就直接显示error();
而不加这个方法，则当该url不是一个gif而是一个图片是也加载;

【--坑】
问题：有时会有gif图加载特别慢或加载不出
解决：使用缓存策略
``` bash
Glide.with(context)
	.load("url")
	.asGif()
	.diskCacheStrategy(DiskCacheStrategy.SOURCE)
	.into(imageView);
```
Glide提供了4个缓存策略（ALL, NONE, SOURCE, RESULT）
其中适合gif的智勇NONE和SOURCE；

问题：gif只想显示1次
解决：下面代码
``` bash
Glide.with(context)
	.load("url")
	.asGif()
	.diskCacheStrategy(DiskCacheStrategy.SOURCE)
	.into(new GlideDrawableImageView(imageView, 1));
```
其中的1为次数，可以自己更改

问题：只想显示gif的第一帧图片
解决：asBitmap()方法
``` bash
Glide.with(context)
	.load("url")
	.asBitmap()
	.error(R.drawable.error_pic)
	into(imageView);
```

### 4、显示视频
是不是感觉又高级了，这个连视频都可以显示
``` bash
String mp4path = "/storage/emulated/0/Pictures/test.mp4";
Glide.with(context)
	.load(Uri.fromFile(new File(mp4path)))
	.into(imageView);
```
但是，很遗憾得告诉你，术业有专攻，Glide在视频这方面也是乏力了，它只能播放本地的视频，而网络上的，那就不是它力所能及的了。

### 5、Glide的缓存
以前说过Glide是不用自己去设置缓存的，它自己会自动去设置。
那时你可能像我一样，心中总有一种不大对劲的感觉。对的，它的智能引发了一个问题：如果你的图片经常变化，那你每次都从缓存中取出的图片就没法变化了。这时，我们就需要对它的缓存进行更改了。
就像前面所说的，Glide会首选使用内存缓存
```
.skipMemoryCache(true)
```
跳过内存缓存，这样，Glide就会将图片放在磁盘缓存中（如此.skipMemoryCache(false)就很没有必要了）
```
.diskCacheStrategy(DiskCacheStrategy.NONE)
```
将磁盘缓存的策略改为NONE，如果只想用网络方式，那就将上面两个都禁掉。
既然磁盘缓存为策略型，那么它就具有不同的枚举：
* DiskCacheStrategy.NONE：不采取缓存
* DiskCacheStrategy.ALL：缓存所有的图片(default)
* DiskCacheStrategy.SOURCE：缓存原图片（分辨率不变）
* DiskCacheStrategy.RESULT：缓存最终的图片（压缩过的分辨率）

### 6、缩略图
缩略图的使用环境：
例如qq头像，没有点击时是一个比较小的图片（例如为100 * 100），点击之后会出现一个模糊的大图片（1000 * 1000），但是过了一会之后这个大图片就变清晰了。

``` bash
Glide.with(context)
	.load("url")
	.thumbnail(0.1f)
	.into(imageView);
```
当然，缩略图也是从网络上加载的，如果原图特别大，那即使你使用了它的10分之一（0.1f），也是很慢的，所以，Glide提供了另一种方法
``` bash
DrawableRequestBuilder<String> thumbnailRequest = Glide
	.with(context)
	.load("url");
Glide.with(context)
	.load("url")
	.thumbnail(thumbnailRequest)
	.into(imageView);
```
此时，缩略图和原图除了长得像，一毛钱关系都没有。

### 7、请求的优先策略
优先策略，根据我们以前学过的Priority，就知道它只是一个策略，并不是准则。
也就是说，它也有可能罢工。那么，在哪种情况下就不遵守了呢。
例如有两个图片，将第一个设置优先等级为HIGH,而第二个设置为LOW，一般情况下是第一个先加载完再加载第二个，但是当第一个大到特别大，第二个小到特别小的时候，当第一个还没加载完成，第二个已经加载结束了，那么就会先显示第二个图片。

Priority总共有4种枚举：
* Priority.LOW
* Priority.NORMAL
* Priority.HIGH
* Priority.IMMEDIATE

``` bash
.priority(Priority.HIGH)
```

### 8、动画
当我们在网络上加载图片时，有一个默认图片，图片加载完成后才是网络图片，那么两个图片转化的过程就是Glide提供的动画。
(为什么不提从缓存中获取图片，因为从缓存中获取速度快，基本不需要用到默认图片，就没有动画这么一说了)
```
.animate(R.anim.slide)
```
该方法可以传入android本身拥有的动画，也可以是自己编写的动画
但是如果该imageView是自定义的组件呢，这个方法就没用了，就必须实现一个ViewPropertyAnimatioin.Animator接口的类
``` bash
ViewPropertyAnimation.Animator animationObject = new ViewPropertyAnimation.Animator() {  
    @Override
    public void animate(View view) {
        
        view.setAlpha(0f);

        ObjectAnimator fadeAnim = ObjectAnimator.ofFloat(view, "alpha", 0f, 1f);
        fadeAnim.setDuration(2500);
        fadeAnim.start();
    }
};

Glide  
    .with(context)
    .load("url")
    .animate(animationObject)
    .into(imageView);
```

### 9、圆形图片
可以使用Glide自身提供的方法
``` bash
		Glide.with(this)
                .load("http://bmob-cdn-5947.b0.upaiyun.com/2016/09/20/042b683ff289436eae48f38ad1df519a.jpg")
                .asBitmap()
                .centerCrop()
                .into(new BitmapImageViewTarget(mIvTest) {
            		@Override
            		protected void setResource(Bitmap resource) {
             			   RoundedBitmapDrawable circularBitmapDrawable =
            	            RoundedBitmapDrawableFactory.create(MainActivity.this.getResources(), resource);
           		    circularBitmapDrawable.setCircular(true);
          		    mIvTest.setImageDrawable(circularBitmapDrawable);
            }
        });
```
使用该方法时需要注意必须有.asBitmap()方法。亲测很坑。
或者使用第二种方法
``` bash
Glide.with(this).load(URL).transform(new CircleTransform(context)).into(imageView);

public static class CircleTransform extends BitmapTransformation {
    public CircleTransform(Context context) {
        super(context);
    }

    @Override protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        return circleCrop(pool, toTransform);
    }

    private static Bitmap circleCrop(BitmapPool pool, Bitmap source) {
        if (source == null) return null;

        int size = Math.min(source.getWidth(), source.getHeight());
        int x = (source.getWidth() - size) / 2;
        int y = (source.getHeight() - size) / 2;

        // TODO this could be acquired from the pool too
        Bitmap squared = Bitmap.createBitmap(source, x, y, size, size);

        Bitmap result = pool.get(size, size, Bitmap.Config.ARGB_8888);
        if (result == null) {
            result = Bitmap.createBitmap(size, size, Bitmap.Config.ARGB_8888);
        }

        Canvas canvas = new Canvas(result);
        Paint paint = new Paint();
        paint.setShader(new BitmapShader(squared, BitmapShader.TileMode.CLAMP, BitmapShader.TileMode.CLAMP));
        paint.setAntiAlias(true);
        float r = size / 2f;
        canvas.drawCircle(r, r, r, paint);
        return result;
    }

    @Override public String getId() {
        return getClass().getName();
    }
} 
```

