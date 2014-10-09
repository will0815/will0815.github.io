---
layout: post
title: Tuning Hadoop
category: Hadoop
date: 2014-10-09
---
##Hadoop集群性能调优##

**mapred-site.xml**

1. mapred.child.java.opts

	配置每个map或reduce使用的内存数量。默认的是200M。网友推荐：如果内存是8G，CPU有8个核，那么就设置成1G就可以了。实际上，在map和reduce的过程中对内存的消耗并不大，但是如果配置的太小，则有可能出现”无可分配内存”的错误。对于Hadoop环境独占的机器可以考虑：总内存 / (Map/Reduce 并发数) * 80%

2. dfs.block.size 

	生的map来综合考量的。一般来说，文件大，集群数量少，还是建议将block size设置大一些的好。
3. mapred.compress.map.output 

	几乎所有会产生大量map output的hadoop作业，都能够通过将中间文件用Lzo进行压缩来提升性能。尽管lzo会使得cpu的load有所增高，但是reduce在shuffle阶段的disk IO通常都能节省不少时间，对job的性能有好处。
	当采用map中间结果压缩的情况下，用户还可以选择压缩时采用另外几种压缩格式进行压缩，现在hadoop支持的压缩格式 有：GzipCodec，LzoCodec，BZip2Codec，LzmaCodec等压缩格式。通常来说，想要达到比较平衡的cpu和磁盘压缩 比，LzoCodec比较适合。但也要取决于job的具体情况。

		<property>
		    <name>mapred.compress.map.output</name>
		    <value>true</value>
		  </property>
		<property> 
		    <name>mapred.map.output.compression.codec</name> 
		    <value>com.hadoop.compression.lzo.LzoCodec</value> 
		</property>  

4. mapred.tasktracker.map.tasks.maximum

	The maximum number of map tasks that will be run simultaneously by a task tracker.

	一个tasktracker最多可以同时运行的map任务数量
	默认值：2
	mapred.tasktracker.map.tasks.maximum = cpu数量
	Sanders: 应该设置成为等于或略小于CPU总核数
	cpu数量 = 服务器CPU总核数 / 每个CPU的核数
	服务器CPU总核数 = more  /proc/cpuinfo | grep 'processor' | wc -l
	每个CPU的核数 = more  /proc/cpuinfo | grep 'cpu cores'
5. mapred.map.tasks

	The default number of map tasks per job
	一个Job会使用task tracker的map任务槽数量，这个值 ≤ mapred.tasktracker.map.tasks.maximum
	默认值：2
	优化值：
	CPU数量 （我们目前的实践值）
	(CPU数量 > 2) ? (CPU数量 * 0.75) : 1  （mapr的官方建议）
6. io.sort.mb    
 
	每一个map都会对应存在一个内存 buffer， map会将已经产生的部分结果先写入到该buffer中，这个buffer默认是100MB大小. 当map的产生数据非常大时，并且把io.sort.mb调 大，那么map在整个计算过程中spill的次数就势必会降低，map task对磁盘的操作就会变少，如果map tasks的瓶颈在磁盘上，这样调整就会大大提高map的计算性能。
7. io.sort.spill.percent    
	这个值就是上述buffer的阈值，默认是0.8，既80%，当buffer中的数据达到这个阈值，后台线程会起来对buffer中已有的数据进行排序，然后写入磁盘，此时map输出的数据继续往剩余的20% buffer写数据，如果buffer的剩余20%写满，排序还没结束，map task被block等待。
8. io.sort.factor     
	当一个map task执行完之后，本地磁盘上(mapred.local.dir)有若干个spill文件，map task最后做的一件事就是执行merge sort，把这些spill文件合成一个文件（partition），有时候我们会自定义partition函数，就是在这个时候被调用。

	当map的中间结果非常大，调大io.sort.factor，有利于减少merge次数，进而减少 map对磁盘的读写频率，有可能达到优化作业的目的。

	默认是10

**Reducer**

1. mapred.tasktracker.reduce.tasks.maximum

	The maximum number of reduce tasks that will be run simultaneously by a task tracker.
	一个task tracker最多可以同时运行的reduce任务数量
	默认值：2
	优化值： (CPU数量 > 2) ? (CPU数量 * 0.50): 1 （mapr的官方建议）

	如果使用fair的调度模式，设置成相同，应该是可以的，但是如果是FIFO模式，我个人认为在map或是reduce阶段，CPU的核数没有得到充分的利用，有些可惜，所以，FIFO模式下，还是尽量配置的map并发数量多于redcue并发数量
2. mapred.reduce.tasks
	The default number of reduce tasks per job. Typically set to 99% of the cluster's reduce capacity, so that if a node fails the reduces can  still be executed in a single wave.

		一个Job会使用task tracker的reduce任务槽数量
		默认值：1
		优化值：
		1.0.95 * mapred.tasktracker.tasks.maximum
		理由：启用95%的reduce任务槽运行task, recude task运行一轮就可以完成。剩余5%的任务槽永远失败任务，重新执行
		2.1.75 * mapred.tasktracker.tasks.maximum
		理由：因为reduce task数量超过reduce槽数，所以需要两轮才能完成所有reduce task。具体快的原理还没有完全理解.
	io.sort.mb/io.sort.factor/ io.sort.spill.percent
	和Map类似

3. mapred.reduce.parallel.copies    

	Reduce copy数据的线程数量，默认值是5

	Reduce task在做shuffle时，实际上就是从不同的已经完成的map上去下载属于自己这个reduce的部分数据，由于map通常有许多个，所以对一个reduce来说，下载也可以是并行的从多个map下载
	Reduce到每个完成的Map Task copy数据（通过RPC调用），默认同时启动5个线程到map节点取数据。这个配置还是很关键的，如果你的map输出数据很大，有时候会发现map早就100%了，reduce一直在1% 2%。。。。。。缓慢的变化，那就是copy数据太慢了，5个线程copy 10G的数据，确实会很慢，这时就要调整这个参数了，但是调整的太大，又会事倍功半，容易造成集群拥堵，所以 Job tuning的同时，也是个权衡的过程，你要熟悉你的数据
4. mapred.job.shuffle.input.buffer.percent

	Reduce在shuffle阶段对下载来的map数据，并不是立刻就写入磁盘的，而是会先缓存在内存中，然后当使用内存达到一定量的时候才刷入磁盘。当指定了JVM的堆内存最大值以后，上面这个配置项就是Reduce用来存放从Map节点取过来的数据所用的内存占堆内存的比例，默认是70%
5. mapred.job.shuffle.merge.percent 

	同Map的io.sort.spill.percent 一样，当shuffle时不是等到buffer满后再在flush到磁盘，而是达到一定阈值后写入磁盘，其默认值是0.66
6. mapred.job.reduce.input.buffer.percent 

	当reduce task真正进入reduce函数的计算阶段的时候，在读取reduce需要的数据时需要多 少的内存百分比来作为reduce读已经sort好的数据的buffer百分比。默认情况下为0，也就是说，默认情况下，reduce是全部从磁盘开始读 处理数据。如果这个参数大于0，那么就会有一定量的数据被缓存在内存并输送给reduce，当reduce计算逻辑消耗内存很小时，可以分一部分内存用来 缓存数据
7. mapred.inmem.merge.threshold

	从map节点取过来的文件个数，当达到这个个数之后，也进行merger sort，然后写到reduce节点的本地磁盘；mapred.job.shuffle.merge.percent优先判断.默认值1000，完全取决于map输出数据的大小
	默认值1000