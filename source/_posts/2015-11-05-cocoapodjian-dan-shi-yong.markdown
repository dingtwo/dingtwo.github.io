---
layout: post
title: "cocoaPod简单使用"
date: 2015-11-05 23:48:00 +0800
comments: true
categories: 
---
### gem-注意修改ruby源
> `gem -v` //获取gem版本信息,查看当前版本是否为最新,否则下一步更新版本
    
> `gem update --system` //可能需要管理员权限,sudo

> 如果没有更新的话,尝试以下两个命令

> `gem install rubygems-update` //可能需要管理员权限

> `update_rubygems` //

> `gem sources --remove https://rubygems.org/` //移除旧的ruby源

> `gem sources -a https://ruby.taobao.org/` //替换为淘宝的镜像

> `gem sources -l` //检查是否修改成功

> `sudo gem install cocoapods` //安装cocoaPods,需要输入密码,为电脑的登录密码

### 安装三方库文件
> `pod setup`

> 将工程文件拖到终端里,获取当前工程路径

> `touch Podfile` //创建Podfile文件, touch命令-新建

> `open Podfile` //用Xcode打开 open-打开命令

> 在podfile文件中添加

> `pod search AFNetworking` //搜索三方库

> `pod install --verbose --no-repo-update` 查看安装进度并且忽略没用的安装过程

2015年10月26日 上午11:56