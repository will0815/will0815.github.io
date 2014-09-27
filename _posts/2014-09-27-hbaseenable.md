---
layout: post
title: hbase表无法enable
category: Hadoop
date: 2014-09-27
---
在enable表时，发现耗时很长都未结束，就ctrl+c退出hbase shell，再进入继续enable表，但此时出现如下错误：

	ERROR: org.apache.hadoop.hbase.TableNotDisabledException:

错误说表未禁用，但进行disable表操作时，又出现如下错误：
ERROR: org.apache.hadoop.hbase.TableNotEnabledException
那肯定是刚才强行退出导致出问题了，根据大家的经验，指出这是因为zookeeper保持了要enable的这个表信息，只需要登录zk删除该记录就行。
登录到zookeeper也有两种方式，一是到zk目录运行脚本连接，而是直接在hbase安装目录bin下运行：

	hbase zkcli
之后就能连接上zookeeper了，跟着删除/hbase/table下对应的表项就行：

	delete /hbase/table/user_video_recommend2
之后就可以成功的enable表了。
如果此时仍然失败，则可以运行一下命了修复META：

	hbase hbck -fixMeta -fixAssignments