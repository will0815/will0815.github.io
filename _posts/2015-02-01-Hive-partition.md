---
layout: post
title: Hive分区分桶
category: Hadoop
date: 2015-02-01
share: true
duoshuo: true
---
##Hive分区分桶  
Hive 分区表  
在Hive Select查询中一般会扫描整个表内容，会消耗很多时间做没必要的工作。有时候只需要扫描表中关心的一部分数据，因此建表时引入了partition概念。分区表指的是在创建表时指定的partition的分区空间。   
Hive可以对数据按照某列或者某些列进行分区管理，所谓分区我们可以拿下面的例子进行解释。 存储日志，其中必然有个属性是日志产生的日期。在产生分区时，就可以按照日志产生的日期列进行划分。把每一天的日志当作一个分区。   
将数据组织成分区，主要可以提高数据的查询速度。至于用户存储的每一条记录到底放到哪个分区，由用户决定。即用户在加载数据的时候必须显示的指定该部分数据放到哪个分区。 
1、一个表可以拥有一个或者多个分区，每个分区以文件夹的形式单独存在表文件夹的目录下。   
2、表和列名不区分大小写。   
3、分区是以字段的形式在表结构中存在，通过describe table命令可以查看到字段存在， 但是该字段不存放实际的数据内容，仅仅是分区的表示（伪列） 。  
语法  

	1. 创建一个分区表，以 ds 为分区列： 
	create table invites (id int, name string) partitioned by (ds string) row format delimited fields terminated by 't' stored as textfile; 
	2. 将数据添加到时间为 2013-08-16 这个分区中： 
	load data local inpath '/home/hadoop/Desktop/data.txt' overwrite into table invites partition (ds='2013-08-16'); 
	3. 将数据添加到时间为 2013-08-20 这个分区中： 
	load data local inpath '/home/hadoop/Desktop/data.txt' overwrite into table invites partition (ds='2013-08-20'); 
	4. 从一个分区中查询数据： 
	select * from invites where ds ='2013-08-12'; 
	5.  往一个分区表的某一个分区中添加数据： 
	insert overwrite table invites partition (ds='2013-08-12') select id,max(name) from test group by id; 
	可以查看分区的具体情况，使用命令： 
	hadoop fs -ls /home/hadoop.hive/warehouse/invites 
	或者： 
	show partitions tablename;
静态/动态分区  
（1）静态分区   

	create table if not exists sopdm.wyp2(id int,name string,tel string)
	partitioned by(age int)
	row format delimited
	fields terminated by ','
	stored as textfile;
	--overwrite是覆盖，into是追加
	insert into table sopdm.wyp2
	partition(age='25')
	select id,name,tel from sopdm.wyp;
（2）动态分区  

	--设置为true表示开启动态分区功能（默认为false）
	set hive.exec.dynamic.partition=true;
	--设置为nonstrict,表示允许所有分区都是动态的（默认为strict）
	set hive.exec.dynamic.partition.mode=nonstrict;
	--insert overwrite是覆盖，insert into是追加
	set hive.exec.dynamic.partition.mode=nonstrict;
	insert overwrite table sopdm.wyp2
	partition(age)
	select id,name,tel,age from sopdm.wyp;

hive中创建分区表没有复杂的分区类型(范围分区、列表分区、hash分区、混合分区等)。分区列也不是表中的一个实际的字段，而是一个或者多个伪列。意思是说在表的数据文件中实际上并不保存分区列的信息与数据。  
下面的语句创建了一个简单的分区表：  

	create table partition_test
	(member_id string,
	name string
	)
	partitioned by (
	stat_date string,
	province string)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
这个例子中创建了stat_date和province两个字段作为分区列。通常情况下需要先预先创建好分区，然后才能使用该分区，例如： 
 
	alter table partition_test add partition (stat_date='20110728',province='zhejiang'); 
这样就创建好了一个分区。这时我们会看到hive在HDFS存储中创建了一个相应的文件夹
每一个分区都会有一个独立的文件夹，下面是该分区所有的数据文件。在这个例子中stat_date是主层次，province是副层次，所有stat_date='20110728'，而province不同的分区都会在/user/hive/warehouse/partition_test/stat_date=20110728 下面，而stat_date不同的分区都会在/user/hive/warehouse/partition_test/ 下面.   
注意，因为分区列的值要转化为文件夹的存储路径，所以如果分区列的值中包含特殊值，如 '%', ':', '/', '#',它将会被使用%加上2字节的ASCII码进行转义，   
特别要注意，在其他数据库中，一般向分区表中插入数据时系统会校验数据是否符合该分区，如果不符合会报错。而在hive中，向某个分区中插入什么样的数据完全是由人来控制的，因为分区键是伪列，不实际存储在文件中，   
  
动态分区，    
因为按照上面的方法向分区表中插入数据，如果源数据量很大，那么针对一个分区就要写一个insert，非常麻烦。况且在之前的版本中，必须先手动创建好所有的分区后才能插入，这就更麻烦了，你必须先要知道源数据中都有什么样的数据才能创建分区。  
使用动态分区可以很好的解决上述问题。动态分区可以根据查询得到的数据自动匹配到相应的分区中去。   
使用动态分区要先设置hive.exec.dynamic.partition参数值为true，默认值为false，即不允许使用：  

	hive> 	
	hive.exec.dynamic.partition=false
	hive> set hive.exec.dynamic.partition=true;
动态分区的使用方法很简单，假设我想向stat_date='20110728'这个分区下面插入数据，至于province插入到哪个子分区下面让数据库自己来判断，那可以这样写：  

	hive> insert overwrite table partition_test partition(stat_date='20110728',province)
	> select member_id,name,province from partition_test_input where stat_date='20110728';
stat_date叫做静态分区列，province叫做动态分区列。select子句中需要把动态分区列按照分区的顺序写出来，静态分区列不用写出来。这样stat_date='20110728'的所有数据，会根据province的不同分别插入到/user/hive/warehouse/partition_test/stat_date=20110728/下面的不同的子文件夹下，如果源数据对应的province子分区不存在，则会自动创建，非常方便，而且避免了人工控制插入数据与分区的映射关系存在的潜在风险。  
注意，动态分区不允许主分区采用动态列而副分区采用静态列，这样将导致所有的主分区都要创建副分区静态列所定义的分区  
动态分区可以允许所有的分区列都是动态分区列，但是要首先设置一个参数  hive.exec.dynamic.partition.mode ：

	hive> set hive.exec.dynamic.partition.mode;
	hive.exec.dynamic.partition.mode=strict
它的默认值是strick，即不允许分区列全部是动态的，这是为了防止用户有可能原意是只在子分区内进行动态建分区，但是由于疏忽忘记为主分区列指定值了，这将导致一个dml语句在短时间内创建大量的新的分区（对应大量新的文件夹），对系统性能带来影响。  
所以我们要设置：  

	hive> set hive.exec.dynamic.partition.mode=nostrick;

再介绍3个参数：  
hive.exec.max.dynamic.partitions.pernode （缺省值100）：每一个mapreduce job允许创建的分区的最大数量，如果超过了这个数量就会报错  
hive.exec.max.dynamic.partitions （缺省值1000）：一个dml语句允许创建的所有分区的最大数量  
hive.exec.max.created.files （缺省值100000）：所有的mapreduce job允许创建的文件的最大数量  
  
  
Hive 桶  
对于每一个表（table）或者分区， Hive可以进一步组织成桶，也就是说桶是更为细粒度的数据范围划分。Hive也是 针对某一列进行桶的组织。Hive采用对列值哈希，然后除以桶的个数求余的方式决定该条记录存放在哪个桶当中。  
把表（或者分区）组织成桶（Bucket）有两个理由：  
（1）获得更高的查询处理效率。桶为表加上了额外的结构，Hive 在处理有些查询时能利用这个结构。具体而言，连接两个在（包含连接列的）相同列上划分了桶的表，可以使用 Map 端连接 （Map-side join）高效的实现。比如JOIN操作。对于JOIN操作两个表有一个相同的列，如果对这两个表都进行了桶操作。那么将保存相同列值的桶进行JOIN操作就可以，可以大大较少JOIN的数据量。  
（2）使取样（sampling）更高效。在处理大规模数据集时，在开发和修改查询的阶段，如果能在数据集的一小部分数据上试运行查询，会带来很多方便。  

创建带桶的 table:  

	create table bucketed_user(id int,name string) clustered by (id) sorted by(name) into 4 buckets row format delimited fields terminated by '\t' stored as textfile; 
	
首先，使用CLUSTERED BY 子句来指定划分桶所用的列和要划分的桶的个数： 

	CREATE TABLE bucketed_user (id INT) name STRING)  CLUSTERED BY (id) INTO 4 BUCKETS; 
在这里，我们使用用户ID来确定如何划分桶(Hive使用对值进行哈希并将结果除 以桶的个数取余数。
对于map端连接的情况，两个表以相同方式划分桶。处理左边表内某个桶的 mapper知道右边表内相匹配的行在对应的桶内。因此，mapper只需要获取那个桶 (这只是右边表内存储数据的一小部分)即可进行连接。这一优化方法并不一定要求 两个表必须桶的个数相同，两个表的桶个数是倍数关系也可以。
桶中的数据可以根据一个或多个列另外进行排序。由于这样对每个桶的连接变成了高效的归并排序(merge-sort), 因此可以进一步提升map端连接的效率。以下语法声明一个表使其使用排序桶：  
 
	CREATE TABLE bucketed_users (id INT, name STRING)  CLUSTERED BY (id) SORTED BY (id ASC) INTO 4 BUCKETS; 
我们如何保证表中的数据都划分成桶了呢？把在Hive外生成的数据加载到划分成桶的表中，当然是可以的。其实让Hive来划分桶更容易。这一操作通常针对已有的表。   
Hive并不检查数据文件中的桶是否和表定义中的桶一致(无论是对于桶 的数量或用于划分桶的列）。如果两者不匹配，在査询时可能会碰到错 误或未定义的结果。因此，建议让Hive来进行划分桶的操作。   
  
索引  
索引可以加快含有group by语句的查询的计算速度  

	create index employees_index on table employees(country)
	as  'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler'
	with deferred rebuild
	in table employees_index_table ;
