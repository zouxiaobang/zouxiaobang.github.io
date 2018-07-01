---
title: 圆形进度条CircleProgress(下)
date: 2016-09-05 22:27:05
tags: [android,自定义组件]
categories: Android
---
## 设置动画

### 同样，不说话，先上图
![拥有波浪形的进度](/img/circleProgress2/CircleProgress.png)

### 波浪形的来历
这是著名的贝塞尔曲线
	贝塞尔曲线(Bézier curve)，又称贝兹曲线或贝济埃曲线，是应用于二维图形应用程序的数学曲线。一般的矢量图形软件通过它来精确画出曲线，贝兹曲线由线段与节点组成，节点是可拖动的支点，线段像可伸缩的皮筋，我们在绘图工具上看到的钢笔工具就是来做这种矢量曲线的。
[赛贝尔曲线](http://baike.baidu.com/link?url=LjgifDUq4km2QVT6SySubWYTAQzxCR46hv5cssKt-F3s3iKL0zj-HJurd-IB-1Xh_9dCcnqD9a4MHobmhiwUGNhgBqh9_4oZhRATuftZofZhNpnuaee2aZJkoNgPp0fBlIBmfBB57wqLNCzlubscpa)
好了，这个就不多说了，都是高数。只要记得Android是有配套的API可以调用的

### 必备参数
maxProgress:最大进度
currentProgress：现在进度

### 开始绘制

#### 1、初始化画笔
``` bash
mProgressPaint = new Paint();
mProgressPaint.setAntiAlias(true);
mProgressPaint.setColor(mProgressColor);
//取两层绘制的交集,取上层
mProgressPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
```

#### 2、在缓冲区画布上画
``` bash
mBitmap = Bitmap.createBitmap((int) mBallRadius*2, (int)mBallRadius*2, Bitmap.Config.ARGB_8888);
mCanvas = new Canvas(mBitmap);
```
要根据图层顺序地画图形
1 背景圆
2 波浪形
3 文本
``` bash
		int width = getMeasuredWidth();
        int height = getMeasuredHeight();

        //绘制圆
        mCanvas.drawCircle(width/2, height/2, mBallRadius, mRoundPaint);
        //开始绘制波形
        mPath.reset();
        //波形的个数
        int count = (int)(mBallRadius + 1)*2 / space;
        //波浪高度
        float y1 = (1 - (float)currentProgress/maxProgress)*mBallRadius*2 + height/2 - mBallRadius;
        mPath.moveTo(-width+y1, y1);
        float d = (1-(float)currentProgress/maxProgress) * space;
        for (int i = 0;i < count;i ++){
        	//贝赛尔曲线
            mPath.rQuadTo(space, -d, space*2, 0);
            mPath.rQuadTo(space, d, space*2, 0);
        }
        mPath.lineTo(width, y1);
        mPath.lineTo(width, height);
        mPath.lineTo(0, height);
        //实心
        mPath.close();
        mCanvas.drawPath(mPath, mProgressPaint);

        //写字
        mCenterText = currentProgress + " %";
        float textWidth = mFontPaint.measureText(mCenterText);
        Paint.FontMetrics fontMetrics = new Paint.FontMetrics();
        float dy = -(fontMetrics.descent + fontMetrics.ascent)/2;
        float x = width/2 - textWidth/2;
        float y = height/2 + dy;
        mCanvas.drawText(mCenterText, x, y, mFontPaint);

        canvas.drawBitmap(mBitmap, 0, 0, null);
```
不知道space是什么？来，直接上图
![](/img/circleProgress2/640.png)
space：每个波形的宽度的一半
y1为图中的y：波浪到达的高度
path：一个矩形
d：波浪的高度

#### 3、提供向外接口，设置进度
``` bash
public void setProgress(int progress){
    currentProgress = progress;
    invalidate();
}
```

### 至此整个绘图结束，只剩调用
``` bash
	<com.example.zouxiaobang.circleprogress.userdefinedview.CircleProgress
        android:id="@+id/cp_test"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:centerText="进度"
        app:ballRadius="100dp"
        app:ballColor="#00bcd4"
        app:centerTextColor="#000000"
        app:centerTextSize="16sp"
        app:progressColor="#00796b"/>
```





