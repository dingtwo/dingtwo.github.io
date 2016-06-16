title: mac下使用WireShark抓包
date: 2016-03-17 00:54:49
tags: mac
---
文章来自:[Ding](http://dingtwo.github.io/2016/03/17/mac下使用WireShark抓包/)
常用的手机网络抓包方式在mac下可以用Charles或者paros,手机上可以用Replica,但是这两种方式都有一些缺点,手机抓包的缺点是使用不方便,只适用用在一些简单的环境,Charles等软件的缺点是每次都需要手动修改手机的服务器和端口号,而且只能抓取wifi状态下的网络包,无法在移动网络环境下(公司渣网速 **PK** 4G)使用,因此重新找了一个其他的方法,操作简单,而且支持手机各种网络状态.
<!-- more -->

[WireShark](https://zh.wikipedia.org/wiki/Wireshark)是一个免费开源的网络数据包分析软件。

1. 安装Wireshark[官网下载](https://www.wireshark.org/download.html)
	这一步我在安装成功之后一直卡在主页显示配置首选项,![](http://7xr09w.com1.z0.glb.clouddn.com/wireshark_waiting.png)
重启电脑之后顺利进入.
这一步如果提示需要X11环境的话进行下一步,否则跳过直接到3,因为WireShark旧版本需要X11环境,新版本可以直接使用.

2. 安装XQuartz[官网下载](http://www.xquartz.org/).

	安装完成会提示注销,然后重新登录,就可以使XQuartz作为默认的X11Server
	![](http://img.blog.csdn.net/20140815211812328)

3. 使用苹果在 iOS 5 中新引入了“远程虚拟接口（Remote Virtual Interface,RVI）”的特性，可以在 Mac 中建立一个虚拟网络接口来作为 iOS 设备的网络栈，这样所有经过 iOS 设备的流量都会经过此虚拟接口。此虚拟接口只是监听 iOS 设备本身的协议栈（但并没有将网络流量中转到 Mac 本身的网络连接上），所有网络连接都是 iOS 设备本身的，与 Mac 电脑本身联不联网或者联网类型无关。iOS设备本身可以为任意网络类型（WiFi/xG），这样在 Mac 电脑上使用任意抓包工具（tcpdump、Wireshark、CPA）抓取 RVI 接口上的数据包就实现了对 iPhone 的抓包。

	通过iTunes手机的UDID:
	![通过iTunes获取UDID](http://7xr09w.com1.z0.glb.clouddn.com/wiresharkitunes_UDID.png)

	或在Xcode的菜单Window->Devices中获取:
	![通过Xcode获取UDID](http://7xr09w.com1.z0.glb.clouddn.com/wiresharkxcode_udid.png)

4. 在终端使用rvictl命令创建RVI接口(remote virtual interface),使用iPhone的UDID作为参数。

		$ rvictl -s UDID
	![](http://7xr09w.com1.z0.glb.clouddn.com/wiresharkstart.png)


5. 终端输入ifconfig -l可以看到多了一个rvi0的虚拟接口

6. 打开WireShark,点击设置按钮,可以看到所有的虚拟接口,选择rvi0,点击开始
	![](http://7xr09w.com1.z0.glb.clouddn.com/wiresharkwireShark_set.png)

7. 接下来就可以看到当前连接手机的所有网络活动,可以在菜单栏下的过滤器里选择HTTP即可抓取所有HTTP协议的请求了
	![](http://7xr09w.com1.z0.glb.clouddn.com/http.png)

8. 在抓包完成之后记得需要移除创建的虚拟端口

		$ rvictl -x UDID
