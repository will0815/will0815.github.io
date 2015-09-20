---
layout: post
title: Sqoop密码和SA用户问题记录
category: Hadoop
date: 2015-09-20
share: true
duoshuo: true
---
sqoop 定时job 运行时需要输入密码问题
1.可以使用except 脚本，实现交互输入密码  
2. 配置sqoop，将密码保存至sqoop的metadata中   
sqoop-site.xml：  

	<property>  
	     <name>sqoop.metastore.client.record.password</name>  
	     <value>true</value>  
	     <description>If true, allow saved passwords in the metastore. </description>
	</property>  
  
2. sqoop 出现Caused by: java.sql.SQLException: User not found: SA  错误  
  
未知问题，看起来像是并发执行sqoop引起的问题  导致无法连接sqoop的metadata库  
不知如何解决， 可以采取的办法  
1.换个用户执行 貌似可以（。。。。。）  
2.更改metadata数据库  如修改成MySQL   如下：  

	<property>  
	    <name>sqoop.metastore.client.autoconnect.url</name>  
	    <value>jdbc:mysql://192.168.117.7:3306/sqoop</value>  
	</property>  
	<property>  
	    <name>sqoop.metastore.client.autoconnect.username</name>
	    <value>scm987</value>  
	</property>  
	<property>  
	    <name>sqoop.metastore.client.autoconnect.password</name>
	    <value>scm258</value>  
	</property>  
	
	
	<property>
	     <name>sqoop.metastore.client.enable.autoconnect</name>
	     <value>true</value>
	</property>
