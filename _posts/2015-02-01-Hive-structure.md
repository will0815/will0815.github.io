---
layout: post
title: Hive基础架构
category: Hadoop
date: 2015-02-01
share: true
duoshuo: true
---
Hive基础架构  
Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供完整的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。其优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析,Hive是建立在 Hadoop 上的数据仓库基础构架。它提供了一系列的工具，可以用来进行数据提取转化加载（ETL），这是一种可以存储、查询和分析存储在 Hadoop 中的大规模数据的机制。Hive 定义了简单的类 SQL 查询语言，称为 HQL，它允许熟悉 SQL 的用户查询数据。同时，这个语言也允许熟悉 MapReduce 开发者的开发自定义的 mapper 和 reducer 来处理内建的 mapper 和 reducer 无法完成的复杂的分析工作。   
Hive技术架构图：  
![](/image/hive1.png)  
服务端组件：   
Driver组件：该组件包括Complier、Optimizer和Executor，它的作用是将HiveQL（类SQL）语句进行解析、编译优化，生成执行计划，然后调用底层的mapreduce计算框架。   
Metastore组件：元数据服务组件，这个组件存储hive的元数据，hive的元数据存储在关系数据库里，hive支持的关系数据库有derby、mysql。元数据对于hive十分重要，因此hive支持把metastore服务独立出来，安装到远程的服务器集群里，从而解耦hive服务和metastore服务，保证hive运行的健壮性。  
Thrift服务：thrift是facebook开发的一个软件框架，它用来进行可扩展且跨语言的服务的开发，hive集成了该服务，能让不同的编程语言调用hive的接口。   
客户端组件：   
CLI：command line interface，命令行接口。   
Thrift客户端：上面的架构图里没有写上Thrift客户端，但是hive架构的许多客户端接口是建立在thrift客户端之上，包括JDBC和ODBC接口。   
WEBGUI：hive客户端提供了一种通过网页的方式访问hive所提供的服务。这个接口对应hive的hwi组件（hive web interface），使用前要启动hwi服务。   
一个Hive hsql 执行流程  
![](/image/hive2.png)  
