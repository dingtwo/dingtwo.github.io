---
title: linux下安装git遇到的小坑
date: 2016-09-29 14:10:01
tags: linux
---
前几天买了个[搬瓦工](bwh1.net)的服务器, 做ss的备用, 为了方便装了[WDCP](http://www.wdlinux.cn/wdcp/)做web服务器的集成环境, 考虑用github的webhook来同步代码, 所以需要在linux下装git.
 使用yum工具, 使用方式和mac下的homebrew类似, 包管理器, 可以方便的安装各种软件, 并且自动安装依赖包.
  
  <!-- more -->
  
在使用

    yum install git-core
    
结果
    
    Error: Cannot retrieve repository metadata (repomd.xml) for repository: RPMForge. Please verify its path and try again
    
RPMForge是一个软件库, 可能是找不到这个包的路径, 然后找到了把这个包的源改成国内[网易源](http://mirrors.163.com/.help/centos.html)的方法

    # wget http://mirrors.163.com/.help/CentOS6-Base-163.repo -O /etc/yum.repos.d/CentOS-Base.repo
    # yum makecache
  
我的系统是CentOS-6, 如果是CentOS-5的话把链接里的6改成5即可, 安装之后可以查看```/etc/yum.repos.d/rpmforge.repo```文件中的配置

    # CentOS-Base.repo
    #
    # The mirror system uses the connecting IP address of the client and the
    # update status of each mirror to pick mirrors that are updated to and
    # geographically close to the client.  You should use this for CentOS updates
    # unless you are manually picking other mirrors.
    #
    # If the mirrorlist= does not work for you, as a fall back you can try the
    # remarked out baseurl= line instead.
    #
    #
    
    [base]
    name=CentOS-$releasever - Base - 163.com
    baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/
    #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
    gpgcheck=1
    gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6
    
    #released updates
    [updates]
    name=CentOS-$releasever - Updates - 163.com
    baseurl=http://mirrors.163.com/centos/$releasever/updates/$basearch/
    #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
    gpgcheck=1
    gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6
    
    #additional packages that may be useful
    [extras]
    name=CentOS-$releasever - Extras - 163.com
    baseurl=http://mirrors.163.com/centos/$releasever/extras/$basearch/
    #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
    gpgcheck=1
    gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6
    
    #additional packages that extend functionality of existing packages
    [centosplus]
    name=CentOS-$releasever - Plus - 163.com
    baseurl=http://mirrors.163.com/centos/$releasever/centosplus/$basearch/
    #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
    gpgcheck=1
    enabled=0
    gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6
    
    #contrib - packages by Centos Users
    [contrib]
    name=CentOS-$releasever - Contrib - 163.com
    baseurl=http://mirrors.163.com/centos/$releasever/contrib/$basearch/
    #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib
    gpgcheck=1
    enabled=0
    gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6
    
修改之后又需要下一个问题

    initscripts-9.03.46-1.el6.centos.1.i686 has missing requires of udev >= ('0', '125', '1')
    util-linux-ng-2.17.2-12.18.el6.i686 has missing requires of udev
    vzdev-1.0-7.swsoft.noarch has missing requires of udev
    
可以看出是缺少了udev, 所以

    yum install udev
    
安装成功之后又到了下一个坑里

    错误：Package: git-1.7.12.4-1.el5.rf.i386 (rpmforge)
              Requires: libcurl.so.3
     You could try using --skip-broken to work around the problem
     You could try running: rpm -Va --nofiles --nodigest

google找到CentOS官网的一个[问题](https://www.centos.org/forums/viewtopic.php?t=5028), 大神一语道破

> The package that you are installing now comes from one of those and is pulling in the libcurl-devel from the other repo when it expects to find the one from the same repo as itself.

可能是从RPMForge安装的包使用了CentOS-Base中的依赖库, 建议使用yum-priorities插件来管理安装时选择仓库的顺序, 这个插件用来保证安装软件时候软件仓库先后次序（priority优先权），一般是默认先从官方base或者镜像安装，然后从社区用户contribute的软件中安装，再从第三方软件仓库中安装。当然这个次序可以自己更改，为了安全和稳定还是依照这个次序吧。

1. 安装priorities插件

        yum install yum-priorities

2. 确认插件开启
    查看/etc/yum/pluginconf.d/priorities.conf文件, 确认文件中```enabled=1```.
    
3. 设置默认仓库优先级
    在/etc/yum.repos.d/CentOS-Base.repo文件中, [base]、[updates]、[addons]、[extras]最后分别设置priority=1，[centosplus]、[contrib]最后分别设置priority=2，其他第三的软件源设置priority=N（推荐N>10）。
    
4. 设置优先级
    设置/etc/yum.repos.d/rpmforge.repo文件, [rpmforge]、[rpmforge-extras]、[rpmforge-testing]最后分别设置priority=11.
    
终于, 最后看见了
    
    已安装:
      git.i686 0:1.7.1-4.el6_7.1
    
    作为依赖被安装:
      perl-Error.noarch 1:0.17015-4.el6                  perl-Git.noarch 0:1.7.1-4.el6_7.1
    
    完毕！
    
参考资料:
http://www.live-in.org/archives/998.html
https://www.centos.org/forums/viewtopic.php?t=5028
http://www.dabu.info/centos-163-source-and-automatically-match-the-source-plug-yum-fastestmirror.html