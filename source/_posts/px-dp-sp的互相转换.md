---
title: 'px,dp,sp的互相转换'
date: 2016-09-09 11:34:53
tags: [android,utils]
categories: Android
---
## 认识
对于android的各个尺寸，是言之不尽道之不明的。百度就可以搜到各种对他们的理解
下面是一个类，用于对px，dp，sp之间的互换
``` bash
import android.content.Context;

/**
 * Created by zouxiaobang on 16/9/9.
 */
public class DisplayUtil {

    public static int px2dip(Context context, float pxValue){
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int)(pxValue/scale+0.5f);
    }

    public static int dip2px(Context context, float dipValue){
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int)(dipValue*scale + 0.5f);
    }

    public static int px2sp(Context context, float pxValue){
        final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
        return (int)(pxValue/fontScale + 0.5f);
    }

    public static int sp2px(Context context, float spValue){
        final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
        return (int)(spValue*fontScale + 0.5f);
    }
}
```