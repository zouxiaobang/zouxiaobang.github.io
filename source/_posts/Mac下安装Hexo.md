---
title: Mac下安装Hexo
date: 2016-08-24 01:40:47
tags: [安装,Hexo]
categories: Hexo
---
这是我的第一篇文章，现在已经是深夜1点多，我还在笔耕不辍。因为构建这个hexo博客，可是弄了我一整夜的时间。

虽然这个博客的style比较丑，但也没办法，以后再改。

## 步骤

### 1、下载git
  
使用下面的指令可查看git是否下载成功

``` bash
$ git --version
```

### 2、下载Node.js

使用下面指令可查看node是否下载成功

``` bash
$ node -v
$ npm -v
```

### 3、（重点）下载Hexo

记得sudo，不然就各种错误

``` bash
sudo npm install -g hexo-cli
```

### 4、初始化站点

要先自己创建一个文件夹，例如hexo，然后进入该文件夹，再执行代码

``` bash
$ init hexo
$ npm install
```

### 5、ok，可以进行本地测试了


``` bash
$ hexo server
```  
我的博客(http://localhost:4000)

### 6、部署到GitHub中

先在GitHub中创建一个repository，名称为用户名.github.io  
修改hexo根目录下的_config.yml文件，先修改其中最后面的部分：  

``` bash
deploy:
	type: git
	repository: git@github.com:yourname/yourname.github.io
	branch: master
```  
（记得在：后面要有一个空格）

### 7、将文章转换为静态文本


``` bash
$ hexo g
```  
该网页生成到public文件夹中

### 8、提交到github中


``` bash
$ npm install hexo-deployer-git --save
$ hexo d
```  
到此结束

### 打开博客
(https://zouxiaobang.github.io)

