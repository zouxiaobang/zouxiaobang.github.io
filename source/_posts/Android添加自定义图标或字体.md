---
title: Android添加自定义图标或字体
date: 2017-12-24 19:46:41
tags: [android,字体库]
categories: Android
---
## 依赖
使用自定义字体库，就是使用joanzapata提供的iconify项目，可进入该github：
[https://github.com/JoanZapata/android-iconify](https://github.com/JoanZapata/android-iconify)
可以看到该readme给出了下载的方法：
	compile 'com.joanzapata.iconify:android-iconify-fontawesome:2.2.2' // (v4.5)
    compile 'com.joanzapata.iconify:android-iconify-entypo:2.2.2' // (v3,2015)
    compile 'com.joanzapata.iconify:android-iconify-typicons:2.2.2' // (v2.0.7)
    compile 'com.joanzapata.iconify:android-iconify-material:2.2.2' // (v2.0.0)
    compile 'com.joanzapata.iconify:android-iconify-material-community:2.2.2' // (v1.4.57)
    compile 'com.joanzapata.iconify:android-iconify-meteocons:2.2.2' // (latest)
    compile 'com.joanzapata.iconify:android-iconify-weathericons:2.2.2' // (v2.0)
    compile 'com.joanzapata.iconify:android-iconify-simplelineicons:2.2.2' // (v1.0.0)
    compile 'com.joanzapata.iconify:android-iconify-ionicons:2.2.2' // (v2.0.1)
上面这些依赖都是一些有名的图标库，例如前端常用的fontawesome，天气图标weathericons等等。
我们只需要添加我们自己需要的库即可，以上一个或多个。

### 问题：并没有看到上面哪个依赖是我们需要使用的iconify
这是因以上的这些依赖中，都会默认去依赖到iconify，这个可以进去github的项目中去看：
```java
dependencies {
    compile project(':android-iconify')
}
```

## 具体使用
### 1、创建自己的Application类
自定义继承Application的类
```java
class MyApplication: Application(){
	override fun onCreate(){
		//该例子使用的是android-iconify-fontawesome:2.2.2这个依赖
		Iconify.with(FontAwesomeModule())
	}
}

```
在Application中调用Iconify的静态方法with()来添加字体库。
#### 问题：为什么要在Application中添加，而不是MainActivity中？
因为在Application中添加，那么整个项目都可以使用到该字体库。
#### 注意：在AndroidManifest.xml中注册该Application的名字为MyApplication

### 2、在xml文件中使用字体库
<com.joanzapata.iconify.widget.IconTextView
	android:text="{fa-search} spin"
	android:textSize="18sp"
	android:textColor="#ff0000"/>

"spin"：如果该字体或图标是动态的，那么就动态来显示该图标
#### 注意：这里的fa-search的“-”而不是 “_”


## 自定义图标
如果在一些网站上看到比较好的图标，例如在fontawesome网站或阿里巴巴矢量图标库等，想要在项目中使用，那么，就需要自定义了。

### 1、下载
到fontawesome网站或阿里巴巴矢量图标库中下载：
fa开头	-- FontAwesome图标库
icon开头 -- 阿里巴巴矢量图标库
以阿里巴巴矢量图标库为例，下载完之后会有一个文件夹，该文件夹下有一些文件：
```java
demo_fontclass.html
demo_symbol.html
demo_unicode.html
demo.css
iconfont.css
iconfont.eot
iconfont.js
iconfont.svg
iconfont.ttf
iconfont.woff
```
在这里面，最主要的就是demo_unicode.html和iconfont.ttf这两个文件。
iconfont.ttf：这是下载下来的字体库，我们需要的就是这个文件
demo_unicode.html：用于查看每个图标对应的字符码

### 2、自定义Icon类和IconFontDescriptor类
```java
enum class MyFontIcons(val character: Char): Icon{
	icon_back('\ue624'),
	icon_close('\ue625'); 

	override fun key(): String{
		return name().replace('_', "-")
	}

	override fun character(): Char{
		return character
	}
}
```
这里的\ue624就是从demo_unicode.html文件中查到的，在对应的图标下面有对应的字符码，例如“&#xe624”，我们取去后4位，然后填上\u即可(\u表示unicode编码)

```java
class MyFontModule: IconFontDescriptor{
	override fun ttfFileName(): String{
		//返回该ttf文件的文件名
		return "iconfont.ttf"
	}

	override fun characters(): Array<MyFontIcons>{
		return MyFontIcons.value()
	}
}
```
这里的characters()方法返回的是一个Icon类型的对象，但是要注意的是，在java中，返回值参数可以写为Icon[]，但在Kotlin中不能写为Array<Icon>，而是写为Array<MyFontIcons>

### 3、在Application添加字体库
```java
Iconify.with(FontAwesomeMudule()).with(MyFontModule())
```

### 4、xml使用
```java
<com.joanzapata.iconify.widget.IconTextView
	android:text="{icon-back}"
	android:textSize="18sp"
	android:textColor="#ff0000"/>
```

Ok，到这里，字体图标库就合入到Android项目中，并且可以使用了。