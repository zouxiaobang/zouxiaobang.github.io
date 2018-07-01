---
title: Volley的简单使用
date: 2016-08-26 23:56:40
tags: [android,框架]
categories: Android
---

## 对Volley的自我理解

	Volley和OkHttp这两个框架，我刚开始是使用OkHttp的，但是，后来在做到需要缓存图片的项目的时候，才开始学习使用Volley。
	好了，废话不多说，这篇文章，并没有牵扯到很复杂的东西，只是简单介绍Volley的入门

### 下载volley.jar包并导包  
[volley.jar下载](http://download.csdn.net/detail/zhangphil/9053413)

## 使用Volley的几种请求方式  
不管哪种方式，都要有一个请求队列
	``` bash
	RequestQueue mQueue = Volley.newRequestQueue(mContext);
	```

### 1、StringRequest
普通的请求

GET请求方式
	``` bash
	StringRequest stringRequest = new StringRequest("http://www.baidu.com",
		new Response.Listener<String>(){
			@Override
			public void onResponse(String response){
				Lod.d("TAG", response);
			}
		},
		new Response.ErrorListener(){
			@Override
			public void onErrorResponse(VolleyError error){
				Lod.e("TAG", error.getMessage(), error);
			}
		});
	```
3个参数：
	1、URL地址
	2、服务器响应成功的回调方法
	3、服务器响应失败的回调方法

POST请求方式
	``` bash
	StringRequest stringRequest = new StringRequest(Method.POST, url, listener, errorListener);
	```
但是Volley并没有为POST方法提供参数的设置
但是我们可以调用StringRequest的父类--Request的getParams(), 所以我们可以在创建StringRequest时使用匿名内部类
	``` bash
	StringRequest stringRequest = new StringRequest(Method.POST, url, listener, errorListener){
		@Override
		protected Map<String, String> getParams throws AuthFailureError{
			Map<String, String> map = new Map<>();
			map.put("name1", "value1");
			map.put("name2", "value2");
			return map;
		}
	};
	```

将它添加到队列中
	``` bash
	mQueue.add(stringRequest);
	```

### 2、JsonRequest
JsonRequest是一个抽象类，但它有两个子类
	--JsonObjectRequest
	--JsonArrayRequest

我就只介绍第一个了
	``` bash
	JsonObjectRequest jsonRequest = new JsonObjectRequest("url", null,
		new Response.Listener{
			Override
			public void onResponse<JSONObject>(JSONObject response){
				Log.d("TAG", response.toString());
			}
		},
		new Response.ErrorListener{
			@Override
			public void onErrorResponse<VolleyError error>(){
				Log.e("TAG", error.getMessage(), error);
			}
		});
	```

记得添加到队列中
	``` bash
	mQueue.add(jsonRequest);
	```

### 3、ImageRequest
ImageRequest是一个下载图片的对象

	``` bash
	ImageRequest imageRequest = new ImageRequest("http://developer.android.com/images/home/aw_dac.png",
		new Response.Listener<Bitmap>(){
			@Override
			public void onResponse<Bitmap response>(){
				//设置图片
				imageView.setImageBitmap(response);
			}
		},
		0,
		0,
		Config.RGB_565,
		new Response.ErrorListener{
			@Override
			public void onErrorResponse(VolleyError error){
				imageView.setImageResource(R.drawable.default_image);
			}
		});
	```
中间3个参数：
	3和4：允许下载的图片的最大宽度和高度，0为不限制。如果图片大于设置的值，则进行压缩
	5：图片的颜色属性

### 4、ImageLoader
当下载图片需要使用到缓存的时候，就需要这个对象

	``` bash
	BitmapCache mCache = new BitmapCache();
	ImageLoader loader = new ImageLoader(mQueue, 
							mCache);
	ImageListener listener = ImageLoader.getImageListener(mImageView, R.drawable.default_pic, R.drawable.failure_pic);
	loader.get("url", listener, 200, 200);

	public class BitmapCache implements ImageCache{
		private LruCache<String, Bitmap> mCache;

		public BitmapCache(){
			int maxSize = 10 * 1024 * 1024;
			mCache = new LruCache<String, Bitmap>(maxSize){
				@Override
				protected int sizeOf(String key, Bitmap bitmap){
					return bitmap.getRowBytes() * bitmap.getHeight();
				}
			};
		}

		@Override
		public Bitmap getBitmap(String url){
			return mCache.get(url);
		}

		@Override
		public void putBitmap(String url, Bitmap bitmap){
			mCache.put(url, bitmap);
		}
	}
	```


