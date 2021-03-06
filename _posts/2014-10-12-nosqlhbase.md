---
layout: post
title: nosql&hbase
category: Hadoop
date: 2014-10-12
---
##                                NoSQL/Hbase

**NoSQL (Not Only SQL )**

一项全新的数据库革命性运动，早期就有人提出，发展至2009年趋势越发高涨。NoSQL的拥护者们提倡运用非关系型的数据存储，相对于铺天盖地的关系型数据库运用，这一概念无疑是一种全新的思维的注入。任何技术的流行都有着现实的强烈需求，NoSQL也是如此，随着互联网的移动化、大数据化，超大规模和高并发的现实需求 传统的关系型数据暴露出越来越多难以满足的缺陷。如：

	1、High performance - 对数据库高并发读写的需求
	2、Huge Storage - 对海量数据的高效率存储和访问的需求
	3、High Scalability && High Availability- 对数据库的高可扩展性和高可用性的需求

什么是NoSQL：

wiki上的定义是“NoSQL is a movement promoting a loosely defined class of non-relational data stores that break with a long history of relational databases”。其实并不存在一个叫NoSQL的产品，它是一类non-relational data stores的集合。NoSQL的重点是non-relational，而传统的数据库是relational。传统关系型数据库的最大缺陷是扩展性，虽然各个数据库厂家都有cluster的解决方案，但是不管是share storage还是share nothing的解决方案，扩展性都十分有限。目前解决数据库扩展性的思路主要有两个：第一是数据分片(sharding)或者功能分区，虽然说可以很好的解决数据库扩展性的问题，但是在实际使用过程中，一旦采用数据分片或者功能分区，必然会导致牺牲“关系型”数据库的最大优势-join，对业务局限性非常大，而数据库也退化成为一个简单的存储系统。另外一个思路是通过maser-slave复制的方式，通过读写分离技术在某种程度上解决扩展性的问题，但这种方案中，由于每个数据库节点必须保存所有的数据，这样每个存储的IO subsystem必然成为扩展的瓶颈，而且masert节点也是一个瓶颈。总的来说，传统关系型数据库的扩展能力十分有限。

NoSQL与关系型数据库在概念上的不同：

CAP：

	Consistency(一致性)，数据一致更新，所有数据变动都是同步的
	Availability(可用性)，好的响应性能
	Partition tolerance(分区容错性) 可靠性
CAP原理告诉我们，这三个因素最多只能满足两个，不可能三者兼顾。对于分布式系统来说，分区容错是基本要求，所以必然要放弃一致性。对于大型网站来说，分区容错和可用性的要求更高，所以一般都会选择适当放弃一致性。对应CAP理论，NoSQL追求的是AP，而传统数据库追求的是CA，这也可以解释为什么传统数据库的扩展能力有限的原因。

BASE：

	Basically Availble：基本可用
	Soft-state： 软状态/柔性事务
	Eventual Consistency：最终一致性
BASE模型是传统ACID模型的反面，不同与ACID，BASE强调牺牲高一致性，从而获得可用性，数据允许在一段时间内的不一致，只要保证最终一致就可以了。最终一致性是整个NoSQL中的一个核心理念。

现如今NoSQL产品分类：

	Key-Value模型：Key-Value模型是最简单，也是使用最方便的数据模型，它支持简单的key对value的键值存储和提取；Key-Value模型的一个大问题是它通常是由HashTable实现的，所以无法进行范围查询，所以有序Key-Value模型就出现了，有序Key-Value支持范围查询；
	
	Ordered Key-Value：虽然有序Key-Value模型能够解决范围查询和问题，但是其Value值依然是无结构的二进制码或纯字符串，通常我们只能在应用层去解析相应的结构。
	
	BigTable的数据模型，能够支持结构化的数据，包括列、列簇、时间戳以及版本控制等元数据的存储；
	文档型存储相对到类BigTable存储又有两个大的提升，一是其Value值支持复杂的结构定义，二是支持数据库索引的定义；全文索引模型与文档型存储的主要区别在于文档型存储的索引主要是按照字段名来组织的，而全文索引模型是按字段的具体值来组织的；
	
	图数据库模型也可以看作是从Key-Value模型发展出来的一个分支，不同的是它的数据之间有着广泛的关联，并且这种模型支持一些图结构的算法。
	他们之间的关系幽默的表示如下：
![](/image/nosql.jpg)


目前NoSQL几个分类的代表产品： 

	Key-Value 存储：Oracle Coherence、Redis、Kyoto Cabinet
	类BigTable存储：Apache HBase、Apache Cassandra
	文档数据库：MongoDB、CouchDB
	全文索引：Apache Lucene、Apache Solr
	图数据库：neo4j、FlockDB



**NoSQL 之Hbase**

Hbase是从hadoop中分离出来的apache顶级开源项目。用java实现了Google的Bigtable系统大部分特性，是一个高可靠性、高性能、面向列、可伸缩的分布式存储系统，利用HBase技术可在廉价PC Server上搭建起大规模结构化存储集群。类似Google Bigtable利用GFS作为其文件存储系统，HBase利用Hadoop HDFS作为其文件存储系统；Google Bigtable利用 Chubby作为协同服务，HBase利用Zookeeper作为对应。

Hbase在Hadoop 生态环境中的位置：
![](/image/hbase-local-in-hadooop.jpg)


如上图HBase位于Hadoop ecosystem结构化存储层，Hadoop HDFS为HBase提供了高可靠性的底层存储支持，Hadoop MapReduce为HBase提供了高性能的计算能力，Zookeeper为HBase提供了稳定服务和failover机制。此外，Pig和Hive还为HBase提供了高层语言支持，使得在HBase上进行数据统计处理变的非常简单。 Sqoop则为HBase提供了方便的RDBMS数据导入功能，使得传统数据库数据向HBase中迁移变的非常方便。与hadoop一样，Hbase目标主要依靠横向扩展，通过不断增加廉价的商用服务器，来增加计算和存储能力。

HBase中的表一般有这样的特点：

	1 大：一个表可以有上亿行，上百万列
	2 面向列:面向列(族)的存储和权限控制，列(族)独立检索。
	3 稀疏:对于为空(null)的列，并不占用存储空间，因此，表可以设计的非常稀疏。

**HBase数据模型**

Table:

	table
	  |-column family
	  	|-column
	  		|-cell
				|-rowkey
				|-timestamp
	  			|-value
HBase没有database的概念。rowkey是数据的一部分，其必须人工指定。 column family 在定义表时需指明。数据存储逻辑视图如下：

	Row Key	Timestamp	Column Family
			URI	Parser
	r1	T3	url=http://www.xxxx.com	title=xxxx
		T2	host=xxxx.com	 
		T1	 	 
	r2	T2	url=http://www.xxxx.com	content=xxxx…
		T1	host=xxxx.com	 

	Ø Row Key: 行键，Table的主键，Table中的记录按照Row Key排序
	Ø Timestamp: 时间戳，每次数据操作对应的时间戳，可以看作是数据的version number
	Ø Column Family：列簇，Table在水平方向有一个或者多个Column Family组成，
	一个Column Family中可以由任意多个Column组成，即Column Family支持动态扩展，
	无需预先定义Column的数量以及类型，所有Column均以二进制格式存储，用户需要自行进行类型转换。

Region:

	当Table随着记录数不断增加而变大后，会逐渐分裂成多份splits，成为regions，一个region由
	[startkey,endkey)表示，不同的region会被Master分配给相应的RegionServer进行管理：
    
-ROOT- && .META. Table

	HBase中有两张特殊的Table，-ROOT-和.META.
	Ø .META.：记录了用户表的Region信息，.META.可以有多个regoin
	Ø -ROOT-：记录了.META.表的Region信息，-ROOT-只有一个region
	Ø Zookeeper中记录了-ROOT-表的location

Hbase整体寻址结构如下：

     ![](/image/hbase-addre.jpg)


Client访问用户数据之前需要首先访问zookeeper，然后访问-ROOT-表，接着访问.META.表，最后才能找到用户数据的位置去访问，中间需要多次网络操作，不过client端会做cache缓存。
HBase系统架构

![](/image/hbase-structure.jpg)


Client

	HBase Client使用HBase的RPC机制与HMaster和HRegionServer进行通信，对于管理类操作，
	Client与HMaster进行RPC；对于数据读写类操作，Client与HRegionServer进行RPC
Zookeeper

	Zookeeper Quorum中除了存储了-ROOT-表的地址和HMaster的地址，HRegionServer也会把自己以
	Ephemeral方式注册到 Zookeeper中，使得HMaster可以随时感知到各个HRegionServer的健康状态
	。此外，Zookeeper也避免了HMaster的 单点问题。
HMaster

	HMaster没有单点问题，HBase中可以启动多个HMaster，通过Zookeeper的Master Election机
	制保证总有一个Master运行，HMaster在功能上主要负责Table和Region的管理工作：
	1.       管理用户对Table的增、删、改、查操作
	2.       管理HRegionServer的负载均衡，调整Region分布
	3.       在Region Split后，负责新Region的分配
	4.       在HRegionServer停机后，负责失效HRegionServer 上的Regions迁移
HRegionServer

![](/image/region-server.jpg)

HRegionServer主要负责响应用户I/O请求，向HDFS文件系统中读写数据，是HBase中最核心的模块。
HRegionServer内部管理了一系列HRegion对象，每个HRegion对应了Table中的一个Region，HRegion中由多 个HStore组成。每个HStore对应了Table中的一个Column Family的存储，可以看出每个Column Family其实就是一个集中的存储单元，因此最好将具备共同IO特性的column放在一个Column Family中，这样最高效。

HStore存储是HBase存储的核心了，其中由两部分组成，一部分是MemStore，一部分是StoreFiles。MemStore是 Sorted Memory Buffer，用户写入的数据首先会放入MemStore，当MemStore满了以后会Flush成一个StoreFile（底层实现是HFile）， 当StoreFile文件数量增长到一定阈值，会触发Compact合并操作，将多个StoreFiles合并成一个StoreFile，合并过程中会进 行版本合并和数据删除，因此可以看出HBase其实只有增加数据，所有的更新和删除操作都是在后续的compact过程中进行的，这使得用户的写操作只要 进入内存中就可以立即返回，保证了HBase I/O的高性能。当StoreFiles Compact后，会逐步形成越来越大的StoreFile，当单个StoreFile大小超过一定阈值后，会触发Split操作，同时把当前 Region Split成2个Region，父Region会下线，新Split出的2个孩子Region会被HMaster分配到相应的HRegionServer 上，使得原先1个Region的压力得以分流到2个Region上。下图描述了Compaction和Split的过程：

![](/image/region-compacte.png)

在理解了上述HStore的基本原理后，还必须了解一下HLog的功能，因为上述的HStore在系统正常工作的前提下是没有问题的，但是在分布式 系统环境中，无法避免系统出错或者宕机，因此一旦HRegionServer意外退出，MemStore中的内存数据将会丢失，这就需要引入HLog了。 每个HRegionServer中都有一个HLog对象，HLog是一个实现Write Ahead Log的类，在每次用户操作写入MemStore的同时，也会写一份数据到HLog文件中，HLog文件定期会滚动出新的，合并删除旧的文件（已持久化到StoreFile中的数据）。当HRegionServer意外终止后，HMaster会通过Zookeeper感知到，HMaster首先会处理遗留的 HLog文件，将其中不同Region的Log数据进行拆分，分别放到相应region的目录下，然后再将失效的region重新分配，领取 到这些region的HRegionServer在Load Region的过程中，会发现有历史HLog需要处理，因此会Replay HLog中的数据到MemStore中，然后flush到StoreFiles，完成数据恢复。


	HBase实践总结：
	column family 不应超过两个，因为一个column family 对应一个store，flush和compaction的触发的基本单位都是Region级别，所以当一个column family有大量的数据的时候会触发整个region里面的其他column family的memstore（其实这些memstore可能仅有少量的数据，还不需要flush的）也发生flush动作；另外compaction触发的条件是当store file的个数（不是总的store file的大小）达到一定数量的时候会发生，而flush产生的大量store file通常会导致compaction，flush/compaction会发生很多IO相关的负载，这对Hbase的整体性能有很大影响。

	一个table对应一个或多个region，在map/reduce 中对应一个map 一个column family 对应一个store。 store 中含一个menstore 和多个fileStore 当menstore 存储到达一定阀值就生成一个filestore， fileStore存放在hdfs上， 当fileStore 数量到达一定阀值就将其合并成一个大的file。
	
	在插入rowkey时，不考虑region的大小及状态，只是根据它的access 编码排序，找到对应的startkey，endkey,确定插入的位置。
	HBase两张管理表：+ .META.:管理用户region，它可以有多个region。 ROOT: 管理.META.的region，它只有一个region。 所以读写数据的流程为： client–>ROOT–>META–>regionserver 在数据的读写过程中client不会与master产生关系 master只是负责管理表结构。

	利用backup-master机制，解决master单点的问题。
	region在存储的数据到达一定阀值后将split，split的同时更形.META.和ROOT表的相关数据。 split region的过程不需要master的参与，master只负责将split后新的region分配到对应的regionserver。

	HBase的数据安全机制之一：在插入数据时先写Hlog然后在写到内存。 一个regionserver中只有一个Hlog， Hlog中只记录内存中存放的数据。 当机器挂掉后，master会根据Hlog中region将其分割，然后分配到不同的机器加载，做数据恢复。