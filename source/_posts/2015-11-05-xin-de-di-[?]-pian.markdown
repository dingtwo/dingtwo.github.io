---
layout: post
title: "新的第一篇"
date: 2015-11-05 23:25:46 +0800
comments: true
categories: 无关紧要
---

#### 重新配置了博客

##### 因为终端git使用不熟，导致掉进好多坑，简单整理一下这个博客的创建过程。

#### 1.首先安装git，安装ruby。

安装ruby可以通过RVM

`curl -L https://get.rvm.io | bash -s stable --ruby//安装RVM的同时安装Ruby`

为保证ruby版本，可以使用以下命令

``` 
rvm install 2.0.0
rvm use 2.0.0
rvm rubygems latest
ruby --version //查看当前ruby版本
```

#### 2.在github上创建repository

在github上创建一个repository，name格式为name.github.io或者name.github.com,这里注意不要勾选readme文件（因为pull的时候没有问题，push的时候因为这个readme不在本地推不上去）。

#### 3.本地安装Octopress

终端输入：

``` 
git clone git://github.com/imathis/octopress.git octopress //从远程仓库克隆octopress
cd octopress //进入octopress文件夹
```

安装octopress依赖项

``` 
gem install bundler
bundle install
```

​	

这里可能会出现几个问题：

* 路径不允许
  
  ​	
  
  ``` 
  ERROR:  While executing gem ... 
  	(Errno::EPERM)
  Operation not permitted - /usr/bin/bundler
  ```
  
  不允许在这个路径安装，所以使用如下命令安装：

``` 
sudo gem install -n /usr/local/bin bundler
```

* 在执行bundle install时需要安装各种依赖包，修改Gemfile文件中的ruby源为淘宝镜像

安装完成后

``` 
rake install //ruby make
rake preview //浏览器输入http://localhost:4000/预览博客
```

#### 4.本地octopress与github pages关联

``` 
cd octopress
rake setup_github_pages	
```

需要输入博客的git地址

``` 
rake generate//生成页面
rake deploy//部署页面并发布
```

​	

提交到source分支下，在git的远程仓库中两个分支，master是博客内容，source是octopress的设置

所以每次修改主题或者博客设置之后需要提交到source

``` 
git add .
git commit -m "备注内容"
git push origin source
```

​	

#### 5.发布文章

``` 
rake new_post["Post Title"]
```

此命令会在octopress/source/_posts目录下生成一个markdown后缀的文件，打开后内容如下：

``` 
layout: post  

title: "Post Title"  

date: yyyy-mm-dd hh:mm:ss  

comments: true  

categories: ""

```

分别是当前日志的标题，发表时间，是否支持评论，分类等，在之后书写日志内容。完成后：

`rake gen_deploy//generate + deploy`

运行成功的话日志就发布成功了，在name.github.io上可以看到当前的博客