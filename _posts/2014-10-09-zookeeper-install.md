---
layout: post
title: zookeeper install
category: Hadoop
date: 2014-10-09
---
**Zookeeper集群安装配置**

1. 修改zookeeper配置文件zoo.cfg

	在Linux系统下解压zookeeper安装包zookeeper-3.4.3.tar.gz ，进入到conf目录，将zoo_sample.cfg拷贝一份命名为zoo.cfg（Zookeeper 在启动时会找这个文件作为默认配置文件），打开该文件进行修改为以下格式

		dataDir=/home/hadoop/temp/zookeeper/data
		server.23=192.168.6.23:2888:3888
		server.24=192.168.6.24:2888:3888
		server.25=192.168.6.25:2888:3888
2. 新建目录、新建并编辑myid文件
（本次配置myid文件放在/home/hadoop/temp/zookeeper/data目录下）

		mkdir /home/hadoop/will/zookeeper/data	//dataDir目录
		vi /home/hadoop/temp/zookeeper/data/myid
	注意myid文件中的内容为：Master中为0，Slave1中为1，Slave2中为2，分别与zoo.cfg中对应起来。
3. 同步安装包

	将解压修改后的zookeeper-3.4.3文件夹分别拷贝到Master、Slave1、Slave2的相同zookeeper安装路径下。注意：myid文件的内容不是一样的，各服务器中分别是对应zoo.cfg中的设置。
4. 启动zookeeper
	Zookeeper的启动与hadoop不一样，需要每个节点都执行，分别进入3个节点的zookeeper-3.4.3目录，启动zookeeper：

		bin/zkServer.sh start
	注意：此时如果报错先不理会，需要继续在另两台服务器中执行相同操作。 

5. 检查zookeeper是否配置成功
	待3台服务器均启动后，如果过程正确的话zookeeper应该已经自动选好leader，进入每台服务器的zookeeper-3.4.3目录，执行以下操作查看zookeeper启动状态：

		bin/zkServer.sh status
	如果出现以下代码表示安装成功了。

		[java] view plaincopy
		JMX enabled by default  
		Using config: /home/hadoop/zookeeper-3.4.3/bin/../conf/zoo.cfg  
		Mode: follower	//或者有且只有一个leader


2 - 解读zookeeper的配置项
-----
zookeeper的默认配置文件为zookeeper/conf/zoo_sample.cfg，需要将其修改为zoo.cfg。其中各配置项的含义，解释如下：

1. tickTime：CS通信心跳数

	Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。tickTime以毫秒为单位。
	1.tickTime=2000  

2. initLimit：LF初始通信时限

	集群中的follower服务器(F)与leader服务器(L)之间初始连接时能容忍的最多心跳数（tickTime的数量）。
	1.initLimit=5  

3. syncLimit：LF同步通信时限

	集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数（tickTime的数量）。
	1.syncLimit=2  
 
4. dataDir：数据文件目录

	Zookeeper保存数据的目录，默认情况下，Zookeeper将写数据的日志文件也保存在这个目录里。
	1.dataDir=/home/michael/opt/zookeeper/data  

5. dataLogDir：日志文件目录

	Zookeeper保存日志文件的目录。
	1.dataLogDir=/home/michael/opt/zookeeper/log  

6. clientPort：客户端连接端口

	客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
	1.clientPort=2333  

7. 服务器名称与地址：集群信息（服务器编号，服务器地址，LF通信端口，选举端口）

	这个配置项的书写格式比较特殊，规则如下：
	1.server.N=YYY:A:B  

	其中N表示服务器编号，YYY表示服务器的IP地址，A为LF通信端口，表示该服务器与集群中的leader交换的信息的端口。B为选举端口，表示选举新leader时服务器间相互通信的端口（当leader挂掉时，其余服务器会相互通信，选择出新的leader）。一般来说，集群中每个服务器的A端口都是一样，每个服务器的B端口也是一样。但是当所采用的为伪集群时，IP地址都一样，只能时A端口和B端口不一样。
	如：

		1.server.0=192.168.6.23:2008:6008  
		2.server.1=192.168.6.24:2008:6008  
		3.server.2=192.168.6.25:2008:6008  
		4.server.3=192.168.6.26:2008:6008  