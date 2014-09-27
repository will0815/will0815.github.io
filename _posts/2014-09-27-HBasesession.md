---
layout: post
title: HBase第一次session记录
category: Hadoop
date: 2014-09-27
---
1. 数据本地化(Data Localization)
数据存在当前JVM 或是当前磁盘
2. HBase的架构 实现数据与操作的分离，没有本地数据。
其数据存放于hadoop上，所以集群能具有很强的健壮性：
hase写过程：先写Hlog
3. HBase的没有指定master，在哪台机器上启动HBase集群，该台机就是master。
4. rowkey是数据的一部分，其必须人工指定。
column family 在定义表时需指明。
5. HBase没有database的概念。
6. HBase表结构：

		table
		|-column family
		|-column
		|-cell
		|-timestamp
		|-value
7. HBase最佳实践：column family 不应超过两个，因为一个column family 对应一个store，flush和compaction的触发的基本单位都是Region级别，所以当一个column family有大量的数据的时候会触发整个region里面的其他column family的memstore（其实这些memstore可能仅有少量的数据，还不需要flush的）也发生flush动作；另外compaction触发的条件是当store file的个数（不是总的store file的大小）达到一定数量的时候会发生，而flush产生的大量store file通常会导致compaction，flush/compaction会发生很多IO相关的负载，这对Hbase的整体性能有很大影响。
8. 一个table对应一个或多个region，在map/reduce 中对应一个map
一个column family 对应一个store。 store 中含一个menstore 和多个fileStore
当menstore 存储到达一定阀值就生成一个filestore， fileStore存放在hdfs上，
当fileStore 数量到达一定阀值就将其合并成一个大的file。
9. qualifier对应column
10. 在插入rowkey时，不考虑region的大小及状态，只是根据它的access 编码排序，找到对应的startkey，endkey,确定插入的位置。
11. HBase两张管理表：+
.META.:管理用户region，它可以有多个region。
ROOT: 管理.META.的region，它只有一个region。
所以读写数据的流程为：
client-->ROOT-->META-->regionserver
在数据的读写过程中client不会与master产生关系
master只是负责管理表结构。
12. 利用backup-master机制，解决master单点的问题。
13. region在存储的数据到达一定阀值后将split，split的同时更形.META.和ROOT表的相关数据。
split region的过程不需要master的参与，master只负责将split后新的region分配到对应的regionserver。
14. HBase的数据安全机制之一：在插入数据时先写Hlog然后在写到内存。
一个regionserver中只有一个Hlog， Hlog中只记录内存中存放的数据。
当机器挂掉后，master会根据Hlog中region将其分割，然后分配到不同的机器加载，做数据恢复。