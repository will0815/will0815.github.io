---
layout: post
title: hbase time集群时间不一致无法启动
category: Hadoop
date: 2014-09-27
---
hbase 分布式时可能因为各个机器的时间不一致导致无法启动
两种解决方案：

1. 联网更新时间：

		sudo tzconfig，
如果命令不存在请使用 dpkg-reconfigure tzdata
然后按照提示选择 Asia对应的序号，选完后会显示一堆新的提示—输入城市名，
如Shanghai或Chongqing，最后再用 sudo date -s “” 来修改本地时间。
按照提示进行选择时区，然后： www.2cto.com
sudo cp /usr/share/zoneinfo/Asia/ShangHai /etc/localtime
上面的命令是防止系统重启后时区改变。

网上同步时间

1. 安装ntpdate工具

		# sudo apt-get install ntpdate
2. 设置系统时间与网络时间同步
 
		# ntpdate cn.pool.ntp.org
3. 将系统时间写入硬件时间
 
	# hwclock –systohc

2. 修改hbase的时间检查， 其默认的时间为30000ms

		hbase.master.maxclockskew
		180000