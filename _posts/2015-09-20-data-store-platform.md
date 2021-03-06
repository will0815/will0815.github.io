---
layout: post
title: 数据平台拓展
category: Nosql
date: 2015-09-20
share: true
duoshuo: true
---
###一、数据的位置  
数据存储数据存储，管理的就是数据的位置，在CPU的位置，数据相对其他数据的位置，CPU善于处理顺序性操作数据指令，即：数据预取。  
随机读取操作即成为瓶颈，预取到缓存、前端总线的数据都是无效的。传统意义上说，磁盘的存取性能要弱于内存，但是要分随机存取及顺序存取不同的场景下讨论。  
#####1、数据存储和更新  
追加写可以让我们尽量保持顺序存储文件。但是当数据要进行更新的时候，有两种选择，一种是在数据原地进行更新操作，这样我们就有了随机IO操作。另一种是把更新都放到文件末尾，然后需要读取更新数据的时候进行替换。  
#####2、读取数据  
一下子读取整个文件，也是很耗费时间的事情，例如数据库中的全表扫描。当我们读取文件中某一个字段时候，我们需要索引。索引的方式有多种，我们可以用一种简单的固定数值大小的有序数组来做索引，数组里存的是当前数据在文件中的存储偏移量。其他索引技术，如hash索引，位图索引等。  
索引相当于在数据之上又加了一层树状结构，可以迅速的读取数据。但是打破了我们前面讲的数据的追加写，这些数据都是根据索引随机写入的。在数据库上建立索引的时候都会遇到这个问题，在传统的机械式磁盘上，这个问题会造成千倍的性能差异。    
有三种方法可以解决上述问题：  
1）把索引放到内存中，可以随机存储和读取，把数据顺序存储到硬盘上。MongoDB，Cassandra都是采取这种方式。这种方式有一个弊端是存储的数据量受限于内存的大小，数据量一大，索引也增大，数据就饱和了。   

2）第二种方式是把大的索引结构，拆成很多小的索引来存储。在内存中批量进来的数据，当积累到一个预定的量，就排序然后顺序写到磁盘上，本身就是一个小的索引，数据存储完，最后加一块小的全局索引数据即可。这样读取数据的时候，要遍历一些小的索引，会有随机读取。本质是用部分小的随机读换取了整体的数据顺序存储。我们通过在内存中保存一个元索引或者Bloom filter来实现处理那些小索引的低延迟。  
日志结构的归并树（log structed merge tree）是一种典型的实现，
	有三个特征：  
	a)一组小的、不变的索引集。   
	b)只能追加写 ，合并重复的文件。   
	c)少量的内存索引消耗换来读取的性能提升。这是一种写优化索引结构。HBase、Cassandra、Bigtable都是通过这种比较小的内存开销来实现读取和存储的平衡。  

3）列式存储或者面向列的存储。纯列式存储和谷歌bigtable那种列式存储还是有所不同的，，虽然占用了同一个名字。列式存储很好理解，就是把数据按照列顺序存储到文件中，读取的时候只读需要的列。列式存储需要保持每一列数据都有相同的顺序，即行N在每一列都有相同的偏移。这很重要，因为同一查询中可能要返回多个列的数据，同时可能我们要对多列直接进行连接。每一列保持同样的顺序我们可以用非常简单的循环实现上述操作，且都是高效的CPU和缓存操作。  
列式存储的缺点是更新数据的时候需要更新每一个列文件中的相应数据，一个常用的方法就是类似LSM那种批量内存写的方式。当查询只是返回某几列数据，列式存储可以大规模减少磁盘IO。除此之外，列式存储的数据往往属于同一类型，可以进行高效的压缩，一些低延迟，高压缩率的扫描宽度、位填充算法都试用。即使对于未压缩的数据流，同时可以进行针对其编码格式的预取。   
列式存储尤其适用于大表扫描，求均值、最大最小值、分组等聚合查询场景。列式存储天然的保持了一列中数据的顺序性，方便两列数据进行关联，而heap-file index结构关联时候，一份数据可以按顺序读取，则另一份数据就会有随机读取了。  
典型优势总结：  
1）列式压缩，低IO   
2）列中每行数据保持顺序，可以按照行id进行关联合并  
3）压缩后的数据依然可以进行预取  
4）数据延迟序列化  
通过heap-file结构把索引存储在内存，是很多NoSQL数据库及一些关系型数据库的首选，例如Riak，CouchBase和MongoDB，模型简单并且运行良好。   
要处理更大量的数据，LSM技术应用更为广泛，提供了同时满足高效存储和读取效率的基于磁盘的存取结构。HBase、Cassandra、RocksDB, LevelDB，甚至MongoDB最新版也支持这种技术。   

列式存储在MPP数据库里面应用广泛，例如RedShift、Vertica及hadoop上的Parquet等。这种结构适合需要大表扫描的数据处理问题，数据聚合类操作（最大最小值）更是他的主战场。  

###二、并行化  
把数据放到分布式集群中运算，有两点最为重要：分区（partition）和副本（replication）。
分区又被称为sharding，在随机访问和暴力扫描任务下都表现不错。通过hash函数把数据分布到多个机器上，很像单机上使用的hashtable，只不过这儿每一个桶都被放到了不同的机器上。这样可以通过hash函数直接去存储数据的机器上把数据取出来，这种模式有很强的扩展性，也是唯一可以根据客户端请求数线性扩展的模式。请求会被独立分发到某一机器上单独处理。    
我们通过分区可以实现批量任务的并行化，例如聚合函数或者更复杂的聚类或者其他机器学习算法，我们通过广播的方式在所有机器上使任务同时执行。我们还可以运行分治策略来使得高计算的任务在一个更短的时间内解决。批处理系统处理大型的计算问题有不错的效果，但是它的并发性不好，因为执行任务的时候会非常消耗集群的资源。所以分区方式在两个极端情况非常简单：1）直接hash访问 2）广播，然后分而治之，在这两种情况之间还有中间地带，那就是在NoSQL数据库中常用的二级索引技术。

二级索引是指不是构建在主键上的索引，意味着数据不会因为索引的值而进行分区。不能直接通过hash函数去路由到数据本身。我们必须把请求广播到所有节点上，这样会限制了并发性，每一个请求都会卷入所有的节点。因此好多基于key-value的数据库拒绝引入二级索引，虽然它很有价值，例如Hbase和Voldemort。也有些数据库系统包含它了，因为它有用，例如Cassandra、MongoDB、Riak等。重要的是我们要理解好他的效益及他对并发性所造成的影响。          
解决上述并发性瓶颈的一个途径是数据副本，例如异步从数据库和Cassandra、MongoDB中的数据副本。实际上副本数据可以是透明的（只是数据恢复时候使用）、只读的（增加读的并发性），可读写的（增加分区容错性）。这些选择都会对系统的一致性造成影响。 

这些对一致性的折中，给我们带来一个值得思考的问题？一致性到底有什么用？实现一致性的代价非常昂贵。在数据库中是用串行化来保证ACID的。他的基本保证是所有操作都是顺序排列的。这样实现起来的代价非常昂贵，所以好多关系型数据库也不把他当成默认选项。所以说要想在包含分布式写操作的系统上实现强一致性，如同坠入深渊。
解决一致性问题的方案也很简单，避免他。假如不能避免它把他隔离到尽可能少的写入和尽可能少的机器上。  
当然有些业务场景是必须要保证数据一致性的，例如银行转账时候。有些业务场景感觉上是必须保持一致性的，但实际上不是，例如标记一个交易是否有潜在的欺诈，我们可以先把它更新到一个新的字段里面，另外再用一条单独的记录数据去关联最开始的那个交易。所以对一个数据平台来说有效的方式是去避免或者孤立需要一致性的请求，一种孤立的方法是采取单一写入者的策略，Datamic就是典型的例子。另一种物理隔离的方法就是去区分请求中可变和不可变的字段，分别查询。Bloom/CALM把这种理念走的更远，默认的配置选项就是无序执行的策略，只有在必要的时候才启用顺序执行读写语句。
