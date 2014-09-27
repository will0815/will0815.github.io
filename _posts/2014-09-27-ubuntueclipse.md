---
layout: post
title: ubuntu下eclipse菜单栏无法展开
category: Linux
date: 2014-09-27
---
因为ubuntu13下菜单栏被代理到状态栏下，下载安装eclipse3.8，打开后菜单栏点击无法打开，网上认为可能是unity的bug

解决办法去掉eclipse的菜单代理：

1.在终端中进入快捷方式的目录，然后运行

sudo gedit eclipse.desktop

2.修改Exec的值，将原有的Exec=eclipse修改为：

Exec=env UBUNTU_MENUPROXY=0  eclipse

------问题解决

顺带学习ubuntu 程序快捷方式

进入 /usr/share/applications 目录下，这里有所有程序的图形化的快捷方式，
你可以直接复制到桌面，即可

搜索ls *程序名*

方法一：进入 /usr/share/applications 目录下，这里有所有程序的图形化的快捷方式，你可以直接复制到桌面，即可

方法二：使用命令的方式： gnome-desktop-item-edit



 	 
 