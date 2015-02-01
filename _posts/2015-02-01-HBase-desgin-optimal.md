---
layout: post
title: HBase设计优化
category: Hadoop
date: 2015-02-01
share: true
duoshuo: true
---
##HBase设计优化
一、Table  
1. 创建表时预分区  
Hbase在创建表时默认只有一个region，这样的话在大量数据导入时会频繁发生region的split，从而导致region 暂停对外服务。同时大量client向同一个region server写数据也会导致集群负载不均衡，影响导入数据。   
一种可以加快批量写入速度的方法是通过预先创建一些空的regions，这样当数据写入HBase时，会按照region分区情况，在集群内做数据的负载均衡。   
参考：http://hbase.apache.org/book.html#precreate.regions  
简单实现：  

	public static byte[][] getHexSplits(String startKey, String endKey, int numRegions) {
	  byte[][] splits = new byte[numRegions-1][];
	  BigInteger lowestKey = new BigInteger(startKey, 16);
	  BigInteger highestKey = new BigInteger(endKey, 16);
	  BigInteger range = highestKey.subtract(lowestKey);
	  BigInteger regionIncrement = range.divide(BigInteger.valueOf(numRegions));
	  lowestKey = lowestKey.add(regionIncrement);
	  for(int i=0; i < numRegions-1;i++) {
	    BigInteger key = lowestKey.add(regionIncrement.multiply(BigInteger.valueOf(i)));
	    byte[] b = String.format("%016x", key).getBytes();
	    splits[i] = b;
	  }
	  return splits;
	}

2. Row key  
Row key 即为Hbase数据的天然索引，通过Row key可以实现：  
通过单个row key访问：即按照某个row key键值进行get操作；  
通过row key的range进行scan：即通过设置startRowKey和endRowKey，在这个范围内进行扫描；  
  
row key可以是任意字符串，最大长度64KB，因为row key在数据中是冗余存在的所以设计row key时也要考虑数据的冗余尤其是海量数据时如果row key设计过长可能导致数据成倍增长，实际应用中一般为10~100bytes，存为byte[]字节数组， 一般设计成定长的 。  
row key是按照字典序存储，因此，设计row key时，也需要考虑这个问题。同时需要注意避免数据热点问题等  
3. Column Family  
不要在一张表里定义太多的 column family 。目前Hbase并不能很好的处理超过2~3个column family的表。因为某个column family在flush的时候，它邻近的column family也会因关联效应被触发flush，最终导致系统产生更多的I/O。（未经验证，不是很理解）。.  
4. In Memory  
创建表的时候，可以通过HColumnDescriptor.setInMemory(true)将表放到RegionServer的缓存中，保证在读取的时候被cache命中。  
5. Max Version  
创建表的时候，可以通过HColumnDescriptor.setMaxVersions(int maxVersions)设置表中数据的最大版本，如果只需要保存最新版本的数据，那么可以设置setMaxVersions(1)。  
6. Time To Live  
创建表的时候，可以通过HColumnDescriptor.setTimeToLive(int timeToLive)设置表中数据的存储生命期，过期数据将自动被删除，例如如果只需要存储最近两天的数据，那么可以设置setTimeToLive(2 * 24 * 60 * 60)。  
7. Compact & Split  
在HBase中，数据在更新时首先写入WAL 日志(HLog)（如果数据安全不是很重要可以关闭先写日志的功能：putrow.setWriteToWAL(false);）和内存(MemStore)中，MemStore中的数据是排序的，当MemStore累计到一定阈值时，就会创建一个新的MemStore，并且将老的MemStore添加到flush队列，由单独的线程flush到磁盘上，成为一个StoreFile。于此同时， 系统会在zookeeper中记录一个redo point，表示这个时刻之前的变更已经持久化了 (minor compact) 。   
StoreFile是只读的，一旦创建后就不可以再修改。因此Hbase的更新其实是不断追加的操作。当一个Store中的StoreFile达到一定的阈值后，就会进行一次合并 (major compact) ，将对同一个key的修改合并到一起，形成一个大的StoreFile，当StoreFile的大小达到一定阈值后，又会对 StoreFile进行分割 (split) ，等分为两个StoreFile。 由于对表的更新是不断追加的，处理读请求时，需要访问Store中全部的StoreFile和MemStore，将它们按照row key进行合并，由于StoreFile和MemStore都是经过排序的，并且StoreFile带有内存中索引，通常合并过程还是比较快的。  
  
二、Hbase client HTable  
写：  
1. Auto Flush  
通过调用HTable.setAutoFlush(false)方法可以将HTable写客户端的自动flush关闭，这样可以批量写入数据到HBase，而不是有一条put就执行一次更新，只有当put填满客户端写缓存时，才实际向HBase服务端发起写请求。默认情况下auto flush是开启的。  
2. Write Buffer  
通过调用HTable.setWriteBufferSize(writeBufferSize)方法可以设置HTable客户端的写buffer大小，如果新设置的buffer小于当前写buffer中的数据时，buffer将会被flush到服务端。其中，writeBufferSize的单位是byte字节数，可以根据实际写入数据量的多少来设置该值。  
3. WAL Flag  
在HBae中，客户端向集群中的RegionServer提交数据时（Put/Delete操作），首先会先写WAL（Write Ahead Log）日志（即HLog，一个RegionServer上的所有Region共享一个HLog），只有当WAL日志写成功后，再接着写MemStore，然后客户端被通知提交数据成功；如果写WAL日志失败，客户端则被通知提交失败。这样做的好处是可以做到RegionServer宕机后的数据恢复。因此，对于相对不太重要的数据，可以在Put/Delete操作时，通过调用Put.setWriteToWAL(false)或Delete.setWriteToWAL(false)函数，放弃写WAL日志，从而提高数据写入的性能。  
4. 批量写  
通过调用HTable.put(Put)方法可以将一个指定的row key记录写入HBase，同样HBase提供了另一个方法：通过调用HTable.put(List<Put>)方法可以将指定的row key列表，批量写入多行记录，这样做的好处是批量执行，只需要一次网络I/O开销，这对于对数据实时性要求高，网络传输RTT高的情景下可能带来明显的性能提升。  
读：  
1. Scanner Caching  
通过调用HTable.setScannerCaching(int scannerCaching)可以设置HBase scanner一次从服务端抓取的数据条数，默认情况下一次一条。通过将此值设置成一个合理的值，可以减少scan过程中next()的时间开销，代价是scanner需要通过客户端的内存来维持这些被cache的行记录。  
2. Scan Attribute Selection  
scan时指定需要的Column Family，可以减少网络传输数据量，否则默认scan操作会返回整行所有Column Family的数据  
3. Close ResultScanner   
通过scan取完数据后，记得要关闭ResultScanner  
4. 批量读  
通过调用HTable.get(Get)方法可以根据一个指定的row key获取一行记录，同样HBase提供了另一个方法：通过调用HTable.get(List)方法可以根据一个指定的row key列表，批量获取多行记录，这样做的好处是批量执行，只需要一次网络I/O开销，这对于对数据实时性要求高而且网络传输RTT高的情景下可能带来明显的性能提升。  
5. Blockcache  
HBase上Regionserver的内存分为两个部分，一部分作为Memstore，主要用来写；另外一部分作为BlockCache，主要用于读。  
写请求会先写入Memstore，Regionserver会给每个region提供一个Memstore，当Memstore满64MB以后，会启动 flush刷新到磁盘。当Memstore的总大小超过限制时（heapsize * hbase.regionserver.global.memstore.upperLimit * 0.9），会强行启动flush进程，从最大的Memstore开始flush直到低于限制。  
读请求先到Memstore中查数据，查不到就到BlockCache中查，再查不到就会到磁盘上读，并把读的结果放入BlockCache。由于BlockCache采用的是LRU策略，因此BlockCache达到上限(heapsize * hfile.block.cache.size * 0.85)后，会启动淘汰机制，淘汰掉最老的一批数据。  
一个Regionserver上有一个BlockCache和N个Memstore，它们的大小之和不能大于等于heapsize * 0.8，否则HBase不能启动。默认BlockCache为0.2，而Memstore为0.4。对于注重读响应时间的系统，可以将 BlockCache设大些，比如设置BlockCache=0.4，Memstore=0.39，以加大缓存的命中率。  
  
服务端计算  
Coprocessor运行于HBase RegionServer服务端，各个Regions保持对与其相关的coprocessor实现类的引用，coprocessor类可以通过RegionServer上classpath中的本地jar或HDFS的classloader进行加载。  
目前，已提供有几种coprocessor：  
Coprocessor：提供对于region管理的钩子，例如region的open/close/split/flush/compact等； 
RegionObserver：提供用于从客户端监控表相关操作的钩子，例如表的get/put/scan/delete等； 
Endpoint：提供可以在region上执行任意函数的命令触发器。  
  
写端计算（不了解）  
	计数  
HBase本身可以看作是一个可以水平扩展的Key-Value存储系统，但是其本身的计算能力有限  （Coprocessor可以提供一定的服务端计算），因此，使用HBase时，往往需要从写端或者读端进行计算，然后将最终的计算结果返回给调用者。举两个简单的例子：  
PV计算：通过在HBase写端内存中，累加计数，维护PV值的更新，同时为了做到持久化，定期（如1秒）将PV计算结果同步到HBase中，这样查询端最多会有1秒钟的延迟，能看到秒级延迟的PV结果。   
分钟PV计算：与上面提到的PV计算方法相结合，每分钟将当前的累计PV值，按照rowkey + minute作为新的rowkey写入HBase中，然后在查询端通过scan得到当天各个分钟以前的累计PV值，然后顺次将前后两分钟的累计PV值相减，就得到了当前一分钟内的PV值，从而最终也就得到当天各个分钟内的PV值。   
	去重  
对于UV的计算，就是个去重计算的例子。分两种情况：  
如果内存可以容纳，那么可以在Hash表中维护所有已经存在的UV标识，每当新来一个标识时，通过快速查找Hash确定是否是一个新的UV，若是则UV值加1，否则UV值不变。另外，为了做到持久化或提供给查询接口使用，可以定期（如1秒）将UV计算结果同步到HBase中。     
如果内存不能容纳，可以考虑采用Bloom Filter来实现，从而尽可能的减少内存的占用情况。除了UV的计算外，判断URL是否存在也是个典型的应用场景。   
	读端计算  
如果对于响应时间要求比较苛刻的情况（如单次http请求要在毫秒级时间内返回），个人觉得读端不宜做过多复杂的计算逻辑，尽量做到读端功能单一化：即从HBase RegionServer读到数据（scan或get方式）后，按照数据格式进行简单的拼接，直接返回给前端使用。当然，如果对于响应时间要求一般，或者业务特点需要，也可以在读端进行一些计算逻辑。  

