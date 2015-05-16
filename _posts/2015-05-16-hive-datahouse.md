---
layout: post
title: hive 数据仓库优劣学习
category: hadoop
date: 2015-05-16
duoshuo: true
share: true
---
#####Hive数据仓库优劣  
Hive是一个基于Hadoop的数据仓库平台，通过Hive，可以方便地进行数据提取转化加载（ETL）的工作。Hive定义了一个类似于SQL的查询语言HQL，能够将用户编写的SQL转化为相应的MapReduce程序。当然，用户也可以自定义Mapper和Reducer来完成复杂的分析工作。  
基于MapReduce的Hive具有良好的扩展性和容错性。不过由于MapReduce缺乏结构化数据分析中有价值的特性，以及Hive缺乏对执行计划的充分优化，导致Hive在很场景中执行效率不高。  
  
强大的数据仓库和数据分析平台至少需要具备以下几点特性。  
灵活的存储引擎  
高效的执行引擎  
良好的可扩展性  
强大的容错机制  
多样化的可视化  

#######存储引擎
Hive数据存储没有特有的数据结构，（只是hdfs文件的映射）也没有为数据建立索引，可以自由地组织Hive中的表，只要在创建表时告诉Hive数据中的列分隔符和行分隔符，Hive就可以解析数据。Hive的元数据存储在RDBMS中，所有数据都基于HDFS存储。Hive包含Table、External Table、Partition和Bucket等数据模型。  

#######执行引擎
Hive的编译器负责编译源代码并生成最终的执行计划，包括语法分析、语义分析、目标代码生成，所做的优化并不多。Hive基于MapReduce，Hive的Sort和GroupBy都依赖MapReduce。而MapReduce相当于固化了执行算子，Map的MergeSort必须执行，GroupBy算子也只有一种模式，Reduce的Merge-Sort也必须可选。另外Hive对Join算子的支持也较少。内存拷贝和数据预处理也会影响Hive的执行效率。  
#######扩展性
Hive关注水平扩展性。简单来讲，水平扩展性指系统可以通过简单的增加资源来支持更大的数据量和负载。Hive处理的数据量是PB级的，而且每小时每天都在增长，这就使得水平扩展性成为一个非常重要的指标。Hadoop系统的水平扩展性是非常好的，而Hive基MapReduce框架，因此能够很自然地利用这点。
#######容错性
Hive有较好的容错性。Hive的执行计划在MapReduce框架上以作业的方式执行，每个作业的中间结果文件写到本地磁盘，最终输出文件写到HDFS文件系统，利用HDFS的多副本机制来保证数据的可靠性，从而达到作业的容错性。如果在作业执行过程中某节点出现故障，那么Hive执行计划基本不会受到影响。因此，基于Hive实现的数据仓库可以部署在由普通机器构建的分布式集群之上。
#######可视化
Hive的可视化界面基本属于字符终端，用户的技术水平一般比较高。面向不同的应用和用户，提供个性化的可视化展现，是Hive改进的一个重要方向。