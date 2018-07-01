---
title: hexo命令及markdown语法
date: 2016-09-01 16:53:50
tags: [Hexo]
categories: Hexo
---
## hexo命令
### 1、新建
	hexo new "my blog article"
### 2、编译
	hexo generate
或
	hexo g
将md文件编译成html文件，存在public文件夹中
### 3、开启本地服务
	hexo server
或
	hexo s
开启后可通过该网址预览(http://localhost:4000)
### 4、部署到github中
	hexo deploy
或
	hexo d
### 5、清除public
	hexo clean
当source文件夹中的资源有改动时就需要用到它

## Markdown语法
### 1、先来一份语法
分段
	```
	eg 两个回车
	```
换行
	```
	eg 回车
	```
标题
	```
	eg # ~ ###### 表示几级标题，总共有六级
	```
引用
	```
	eg >
	```
列表
	```
	eg * + - 1.  注意使用时后面有空格
	```
代码区块
	```
	eg 开头4个空格， 或转换键
	```
链接
	```
	eg [文字]  (链接地址)   中间没有空格
	```
图片
	```
	eg ![图片说明]  （图片地址） 中间没有空格，图片可以是本地的，也可以是网络地址
	```
强调
	```
	eg 两个星号中间文字，或两个下划线中间文字，或四个星号中间文字，或四个下划线中间文字
	```
代码
	```
	eg 三个``` 开头，再三个结尾。
	```
### 2、好了，开一篇范文吧
[范文](https://www.zybuluo.com/mdeditor)









