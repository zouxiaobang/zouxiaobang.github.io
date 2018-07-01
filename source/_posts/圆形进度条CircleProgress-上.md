---
title: 圆形进度条CircleProgress(上)
date: 2016-09-04 19:17:26
tags: [android,自定义组件]
categories: Android
---
## github项目
[波浪形圆形进度条](https://github.com/zouxiaobang/CircleProgress)
## 画图

### 先上图
![圆形与文字](/img/circleProgress1/circleProgress1.png)

### 现在开始使用代码来画图
#### 1、自定义属性
在value文件夹中新建一个attrs.xml配置文件
（注意value文件夹不是什么value21什么的，曾被坑得找不着北）
``` python
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="CircleProgress">
        <attr name="ballRadius" format="dimension"/>
        <attr name="ballColor" format="color"/>
        <attr name="centerText" format="string"/>
        <attr name="centerTextColor" format="color"/>
        <attr name="centerTextSize" format="dimension"/>
    </declare-styleable>
</resources>
```
此处介绍一下：
圆形
* 半径：ballRadius
* 颜色：ballColor

文字
* 文本：centerText
* 颜色：centerTextColor
* 大小：centerTextSize

#### 2、继承View
继承并重新它的前三个构造方法
``` bash
	public CircleProgress(Context context) {
        this(context, null);
    }

    public CircleProgress(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CircleProgress(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.CircleProgress);
        mBallRadius = typedArray.getDimension(R.styleable.CircleProgress_ballRadius, 260f);
        mBallColor = typedArray.getColor(R.styleable.CircleProgress_ballColor, Color.argb(255, 33, 150, 243));
        mCenterText = typedArray.getString(R.styleable.CircleProgress_centerText);
        mCenterTextSize = typedArray.getDimension(R.styleable.CircleProgress_centerTextSize, 20f);
        mCenterTextColor = typedArray.getColor(R.styleable.CircleProgress_centerTextColor, Color.argb(255, 48, 63, 159));
        typedArray.recycle();
    }
```
通过TypedArray对象获取xml文件中的设置参数
记得最后要recycle()回收

#### 3、初始化画笔
这里需要画两种东西，即圆形和文本，所以需要使用到两种画笔
``` bash
mRoundPaint = new Paint();
mRoundPaint.setColor(mBallColor);
mRoundPaint.setAntiAlias(true);

mFontPaint = new Paint();
mFontPaint.setColor(mCenterTextColor);
mFontPaint.setAntiAlias(true);
mFontPaint.setTextSize(mCenterTextSize);
mFontPaint.setFakeBoldText(true);
```

#### 4、对view的测量
必须测量，不然默认的话就只有match_parent这个尺寸了，不管怎么设置都是这个尺寸
``` bash
	@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(measure(widthMeasureSpec), measure(heightMeasureSpec));
    }

    /**
     * 对宽高进行测量
     * @param measureSpec
     * @return
     */
    private int measure(int measureSpec) {
        int result = 0;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        if (specMode == MeasureSpec.EXACTLY){
            result = specSize;
        } else {
            result = (int) mBallRadius * 2;
            if (specMode == MeasureSpec.AT_MOST){
                result = result<specSize?result:specSize;
            }
        }

        return result;
    }
```
这时，当我们使用wrap_content的时候宽高是这个圆形的直径
（注意：result = (int) mBallRadius * 2; 乘以2才是直径）

#### 5、开始画画
画个圆之前需要先获得大小
``` bash
int width = getMeasuredWidth();
int height = getMeasuredHeight();
```
先画个圆
``` bash
canvas.drawCircle(width/2, height/2, mBallRadius, mRoundPaint);
```

#### 6、写字
``` bash
float textWidth = mFontPaint.measureText(mCenterText);
Paint.FontMetrics fontMetrics = new Paint.FontMetrics();
float dy = -(fontMetrics.descent + fontMetrics.ascent)/2;
float x = width/2 - textWidth/2;
float y = height/2 + dy;
canvas.drawText(mCenterText, x, y, mFontPaint);
```
其中，使用到FontMetrics对象，通过该对象可以测量字体粗体时的大小

### 画图结束
先在layout的xml文件中使用看看有没有该效果
``` python
	<com.example.zouxiaobang.circleprogress.userdefinedview.CircleProgress
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:centerText="进度"
        app:ballRadius="100dp"
        app:ballColor="#00bcd4"
        app:centerTextColor="#000000"
        app:centerTextSize="16sp"/>
```





