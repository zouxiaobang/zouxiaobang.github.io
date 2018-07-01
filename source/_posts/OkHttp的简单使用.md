---
title: OkHttp的简单使用
date: 2016-08-30 11:09:50
tags: [android,框架]
categories: Android
---
## 关于OkHttp
OkHttp最近的火热我觉得就是Google在6.0版本删除HttpClient相关的API引起的。  
所以，为了多敲代码养家糊口，学习OkHttp框架已成板上钉钉的事了。  

### 下载okhttp.jar包并导包
[okhttp.jar](http://download.csdn.net/detail/gongkaii/8551425)
[okio.jar](http://download.csdn.net/detail/chiwenheng2078/9188867)

## 好了，开始使用了
记得第一步都是先创建一个Cient
``` bash
OkHttpClient mClient = new OkHttpClient();
```
当然，现在的OkHttp都升级到3.0以上了，所以以前的构造方法虽然有用，但也不是很有用
``` bash
OkHttpClient mClient = new OkHttpClient.Builder();
```

### 1、Http GET
直接上代码了
#### 同步
``` bash
Request request = new Request.Builder()
	.url("url")
	.build();
Response response = mClient.newCall(request).execute();
if(response.isSuccessful()){
	Log.i("TAG", response.body().string());
}
```
#### 异步
``` bash
Request request = new Request.Builder()
	.url("url")
	.build();
mClient.newCall(request).enqueue(new Callback(){
	@Override
	public void onFailure(Request request, IOException ex){

	}

	@Override
	public void onResponse(Response response) throws IOException{
		if(response.isSuccessful()){
			Log.i("TAG", response.body().string());
		}
	}
});
```
#### Step
1、创建请求对象Request
2、获取Response对象
3、注意...使用同步时要新开启一个线程，记得写权限

### 2、Http POST
#### 这里只举异步的例子
``` bash
//此处Body的创建与以前版本不同
RequestBody body = FormBody.Builder()
	.add("name1", "value1")
	.add("name2", "value2")
	.build();
Request request = new Request.Builder()
	.url("url")
	.post(body)
	.build();
mClient.newCall(request).enqueue(new Callback(){
	@Override
		public void onFailure(Request request, IOException ex){

		}

		@Override
		public void onResponse(Response response) throws IOException{
			if(response.isSuccessful()){
				Log.i("TAG", response.body().string());
			}
		}
});
```
#### 提交Json数据
``` bash
//已有json
RequestBody body = RequestBody.create(JSON, json);
Request request = new Request.Builder()
	.url("url")
	.post(body)
	.build();
//response...
#### 提交文件
``` bash
public static final MediaType MEDIA_TYPE_MARKDOWN
	= MediaType.parse("text/x-markdown; charset=utf-8");
Request request = new Request.Builder()
	.url("url")
	.post(RequestBody.create(MEDIA_TYPE_MARKDOWN, file))
	.build();

Response response = client.newCall(request).execute();
```
```
#### Step
1、创建RequestBody对象，在该对象中传入参数
2、创建Request对象
3、获取Response对象

### 3、Http 缓存
``` bash
File file = getExternalCacheDir().getAbsoluteFile();
int maxSize = 10 * 1024 * 1024;
Cache cache = new Cache(file, maxSize);

mClient = new OkHttpClient.Builder()
	.cache(cache)
	.build();
//Request... Response...
```
### 4、取消一个正在执行的call
使用Call.cancel()可以停止掉一个正在执行的call，但是如果该线程正在读写请求或响应的时候，就会出现IO异常
3.0之后没有了OkHttpClient.cancel(tag)的方法
#### 同步
``` bash
//...
Call call = mClient.newCall(request);
Response resp = call.execute();
//停止
call.cancel();
```
#### 异步
``` bash
//...
Call call = mClient.newCall(request);
call.enqueue(new Callback(){
	@Override
	public void onFailure(Request request, IOException ex){
		//call.cancel();
	}

	@Override
	public void onResponse(Response response) throws IOException{
		if(response.isSuccessful()){
			Log.i("TAG", response.body().string());
			//call.cancel();
		}
	}
});

```








