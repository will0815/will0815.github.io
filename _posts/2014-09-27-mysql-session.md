---
layout: post
title: mysql 性能session
category: DB
date: 2014-09-27
---
mysql

主从：

外围app只能从主数据库中写入数据，从从数据库中读取数据，从而减小数据压力

1. 要实现主从数据库之间的同步复制，需要用户授权，使得从数据库可以登录主数据库并拷贝数据
grant all "." to rob@172.20.29.29 identified by "123"

2. bin_log 日志(保存数据库的增删改数据操作，用作数据恢复)

	A:binlog日志打开方法

		在my.cnf这个文件中加一行（Windows为my.ini）。
	
			#vi /etc/my.cnf
		[mysqld]
		log-bin=mysqlbin-log #添加这一行就ok了=号后面的名字自己定义吧
		然后我们可以对数据库做简单的操作后到mysql数据文件所在的目录来看binlog文件。

	B:关于bin-log日志的相关命令：

		查看当前bin_log情况 show BINARY logs
		查看当前bin_log情况 show master logs
		查看log_bin相关配置 show variables like ‘%log_bin%’
		切换bin_log日志（生成一个新的日志文件） flush logs
		删除mysqlbin.000002之前的bin_log，并修改index中相关数据 PURGE BINARY LOGS TO ‘mysqlbin.000002′
		删除2011-07-09 12:40:26之前的bin_log，并修改index中相关数据 PURGE BINARY LOGS BEFORE ’2011-07-09 12:40:26′
		删除2011-07-09之前的bin_log，并修改index中相关数据 PURGE BINARY LOGS BEFORE ’2011-07-09′
		设置bin_log过期日期 set global expire_logs_days=5;
		重设bin_log日志，以前的所有日志将被删除并且重设index中的数据 reset master;

	C:利用bin-log日志恢复数据：

		mysqlbinlog --no-defaults --start-position="102" --stop-position="201" mysql_bin.000001 |mysql -uroot -p123456
		亦可导出为sql文件，再导入至数据库中：
		mysqlbinlog --no-defaults --start-date="2012-10-15 16:30:00" --stop-date="2012-10-15 17:00:00" mysql_bin.000001 >d:\1.sql

3. mysql 数据备份
	mysqldump -uroot -p123456 test -l -F > "/tmp/test.sql"
	备份数据库两个主要方法是用 mysqldump 程序或直接拷贝数据库文件（如用 cp、cpio 或 tar 等）。每种方法都有其优缺点：

mysqldump 与 MySQL 服务器协同操作。直接拷贝方法在服务器外部进行，并且你必须采取措施保证没有客户正在修改你将拷贝的表。

如果你想用文件系统备份来备份数据库，也会发生同样的问题：如果数据库表在文件系统备份过程中被修改，
进入备份的表文件主语不一致的状态，而对以后的恢复表将失去意义。文件系统备份与直接拷贝文件的区别是对后者你完全控制了
备份过程，这样你能采取措施确保服务器让表不受干扰。
mysqldump 比直接拷贝要慢些。
mysqldump 生成能够移植到其它机器的文本文件，甚至那些有不同硬件结构的机器上。直接拷贝文件不能移植到其它机器上，
除非你正在拷贝的表使用 MyISAM 存储格式。ISAM 表只能在相似的硬件结构的机器上拷贝。在 MySQL 3.23 中引入的 MyISAM 表
存储格式解决了该问题，因为该格式是机器无关的，所以直接拷贝文件可以移植到具有不同硬件结构的机器上。只要满足两个条件
：另一台机器必须也运行 MySQL 3.23 或以后版本，而且文件必须以 MyISAM 格式表示，而不是 ISAM 格式。

1 使用 mysqldump 备份和拷贝数据库

	当你使用 mysqldumo 程序产生数据库备份文件时，缺省地，文件内容包含创建正在倾倒的表的
	CREATE 语句和包含表中行数据的 INSERT 语句。换句话说，mysqldump 产生的输出可在以后用
	作 mysql 的输入来重建数据库。
	你可以将整个数据库倾倒进一个单独的文本文件中，如下：
	%mysqldump samp_db >/usr/archives/mysql/samp_db.1999-10-02

4. 主从复制的配置

		A:保证所有主从的server id 唯一
		B:主服务器只需要保证开启bin-log，并且配置server id
		C:从服务器需要配置：bin-log,server id, master-host,master-user,master-password,master-port

5. 分表， 分区

		RANGE 分区：基于属于一个给定连续区间的列值，把多行分配给分区。
		LIST 分 区：类似于按RANGE分区，区别在于LIST分区是基于列值匹配一个离散值集合中的某个值来进行选择。
		HASH分区：基于用户定义的表达式的返 回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含MySQL 中 有效的、产生非负整数值的任何表达式。
		KEY 分 区：类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且MySQL 服务器 提供其自身的哈希函数。必须有一列或多列包含整数值。
		CREATE TABLE employees (
		id INT NOT NULL,
		fname VARCHAR(30),
		lname VARCHAR(30),
		hired DATE NOT NULL DEFAULT '1970-01-01',
		separated DATE NOT NULL DEFAULT '9999-12-31',
		job_code INT NOT NULL,
		store_id INT NOT NULL
		)
		PARTITION BY RANGE (store_id) (
		PARTITION p0 VALUES LESS THAN (6),
		PARTITION p1 VALUES LESS THAN (11),
		PARTITION p2 VALUES LESS THAN (16),
		PARTITION p3 VALUES LESS THAN (21)
		)；
		CREATE TABLE employees (
		id INT NOT NULL,
		fname VARCHAR(30),
		lname VARCHAR(30),
		hired DATE NOT NULL DEFAULT '1970-01-01',
		separated DATE NOT NULL DEFAULT '9999-12-31',
		job_code INT,
		store_id INT
		)
		PARTITION BY LIST(store_id)
		PARTITION pNorth VALUES IN (3,5,6,9,17),
		PARTITION pEast VALUES IN (1,2,10,11,19,20),
		PARTITION pWest VALUES IN (4,12,13,14,18),
		PARTITION pCentral VALUES IN (7,8,15,16)
		)；
		CREATE TABLE employees (
		id INT NOT NULL,
		fname VARCHAR(30),
		lname VARCHAR(30),
		hired DATE NOT NULL DEFAULT '1970-01-01',
		separated DATE NOT NULL DEFAULT '9999-12-31',
		job_code INT,
		store_id INT
		)
		PARTITION BY HASH(store_id)
		PARTITIONS 4；

5. sql 优化

		A：查看slow log
		B: 判断是否需要index
		
		优化目标
		减少 IO 次数
		IO永远是数据库最容易瓶颈的地方，这是由数据库的职责所决定的，大部分数据库操作中超过90%的时间都是 IO 操作所占用的，
		减少 IO 次数是 SQL 优化中需要第一优先考虑，当然，也是收效最明显的优化手段。
		降低 CPU 计算
		除了 IO 瓶颈之外，SQL优化中需要考虑的就是 CPU 运算量的优化了。order by, group by,distinct …
		都是消耗 CPU 的大户（这些操作基本上都是 CPU 处理内存中的数据比较运算）。当我们的 IO 优化做到一定阶段之后，
		降低 CPU 计算也就成为了我们 SQL 优化的重要目标
		
		优化方法
		改变 SQL 执行计划
		明确了优化目标之后，我们需要确定达到我们目标的方法。对于 SQL 语句来说，达到上述2个目标的方法其实只有一个，
		那就是改变 SQL 的执行计划，让他尽量“少走弯路”，尽量通过各种“捷径”来找到我们需要的数据，
		以达到 “减少 IO 次数” 和 “降低 CPU 计算” 的目标

常见误区

	count(1)和count(primary_key) 优于 count(*)
	很多人为了统计记录条数，就使用 count(1) 和 count
	(primary_key) 而不是 count(*) ，他们认为这样性能更好，
	其实这是一个误区。对于有些场景，这样做可能性能会更差，应为数据库对 count(*) 计数操作做了一些特别的优化。
	count(column) 和 count(*) 是一样的
	这个误区甚至在很多的资深工程师或者是 DBA 中都普遍存在，很多人都会认为这是理所当然的。
	实际上，count(column) 和 count(*) 是一个完全不一样的操作，所代表的意义也完全不一样。
	count(column) 是表示结果集中有多少个column字段不为空的记录
	count(*) 是表示整个结果集有多少条记录
	select a,b from … 比 select a,b,c from … 可以让数据库访问更少的数据量
	这个误区主要存在于大量的开发人员中，主要原因是对数据库的存储原理不是太了解。
	实际上，大多数关系型数据库都是按照行（row）的方式存储，而数据存取操作都是以一个固定大小的IO单元
	（被称作 block 或者 page）为单位，一般为4KB，8KB… 大多数时候，每个IO单元中存储了多行，
	每行都是存储了该行的所有字段（lob等特殊类型字段除外）。
	所以，我们是取一个字段还是多个字段，实际上数据库在表中需要访问的数据量其实是一样的。
	当然，也有例外情况，那就是我们的这个查询在索引中就可以完成，也就是说当只取 a,b两个字段的时候，
	不需要回表，而c这个字段不在使用的索引中，需要回表取得其数据。在这样的情况下，二者的IO量会有较大差异。
	order by 一定需要排序操作
	我们知道索引数据实际上是有序的，如果我们的需要的数据和某个索引的顺序一致，而且我们的查询又通过这个索引来执行，
	那么数据库一般会省略排序操作，而直接将数据返回，因为数据库知道数据已经满足我们的排序需求了。
	实际上，利用索引来优化有排序需求的 SQL，是一个非常重要的优化手段
	延伸阅读：MySQL ORDER BY 的实现分析 ，MySQL 中 GROUP BY 基本实现原理 以及 MySQL DISTINCT 的基本实现原理
	执行计划中有 filesort 就会进行磁盘文件排序
	有这个误区其实并不能怪我们，而是因为 MySQL 开发者在用词方面的问题。
	filesort 是我们在使用 explain 命令查看一条 SQL 的执行计划的时候可能会看到在 “Extra” 一列显示的信息。
	实际上，只要一条 SQL 语句需要进行排序操作，都会显示“Using filesort”，这并不表示就会有文件排序操作。
	延伸阅读：理解 MySQL Explain 命令输出中的filesort，我在这里有更为详细的介绍

基本原则

尽量少 join

MySQL 的优势在于简单，但这在某些方面其实也是其劣势。MySQL 优化器效率高，但是由于其统计信息的量有限，
优化器工作过程出现偏差的可能性也就更多。对于复杂的多表 Join，一方面由于其优化器受限，
再者在 Join 这方面所下的功夫还不够，所以性能表现离 Oracle 等关系型数据库前辈还是有一定距离。
但如果是简单的单表查询，这一差距就会极小甚至在有些场景下要优于这些数据库前辈。

尽量少排序

排序操作会消耗较多的 CPU 资源，所以减少排序可以在缓存命中率高等 IO 能力足够的场景下会较大影响 SQL 的响应时间。
对于MySQL来说，减少排序有多种办法，比如：
上面误区中提到的通过利用索引来排序的方式进行优化
减少参与排序的记录条数
非必要不对数据进行排序
…

尽量避免 select *

很多人看到这一点后觉得比较难理解，上面不是在误区中刚刚说 select 子句中字段的多少并不会影响到读取的数据吗？
是的，大多数时候并不会影响到 IO 量，但是当我们还存在 order by 操作的时候，
select 子句中的字段多少会在很大程度上影响到我们的排序效率，这一点可以通过我之前一篇介绍 MySQL ORDER BY
的实现分析 的文章中有较为详细的介绍。
此外，上面误区中不是也说了，只是大多数时候是不会影响到 IO 量，当我们的查询结果仅仅只需要在索引中就能找到的时候，
还是会极大减少 IO 量的。

尽量用 join 代替子查询

虽然 Join 性能并不佳，但是和 MySQL 的子查询比起来还是有非常大的性能优势。
MySQL 的子查询执行计划一直存在较大的问题，虽然这个问题已经存在多年，
但是到目前已经发布的所有稳定版本中都普遍存在，一直没有太大改善。虽然官方也在很早就承认这一问题，
并且承诺尽快解决，但是至少到目前为止我们还没有看到哪一个版本较好的解决了这一问题。

尽量少 or

当 where 子句中存在多个条件以“或”并存的时候，MySQL 的优化器并没有很好的解决其执行计划优化问题，
再加上 MySQL 特有的 SQL 与 Storage 分层架构方式，造成了其性能比较低下，很多时候使用 union all
或者是union（必要的时候）的方式来代替“or”会得到更好的效果。
尽量用 union all 代替 union
union 和 union all 的差异主要是前者需要将两个（或者多个）结果集合并后再进行唯一性过滤操作，
这就会涉及到排序，增加大量的 CPU 运算，加大资源消耗及延迟。所以当我们可以确认不可能出现重复结果集或者
不在乎重复结果集的时候，尽量使用 union all 而不是 union。

尽量早过滤

这一优化策略其实最常见于索引的优化设计中（将过滤性更好的字段放得更靠前）。
在 SQL 编写中同样可以使用这一原则来优化一些 Join 的 SQL。比如我们在多个表进行分页数据查询的时候，
我们最好是能够在一个表上先过滤好数据分好页，然后再用分好页的结果集与另外的表 Join，
这样可以尽可能多的减少不必要的 IO 操作，大大节省 IO 操作所消耗的时间。

避免类型转换

这里所说的“类型转换”是指 where 子句中出现 column 字段的类型和传入的参数类型不一致的时候发生的类型转换：

人为在column_name 上通过转换函数进行转换
直接导致 MySQL（实际上其他数据库也会有同样的问题）无法使用索引，如果非要转换，应该在传入的参数上进行转换
由数据库自己进行转换
如果我们传入的数据类型和字段类型不一致，同时我们又没有做任何类型转换处理，MySQL
可能会自己对我们的数据进行类型转换操作，也可能不进行处理而交由存储引擎去处理，这样一来，
就会出现索引无法使用的情况而造成执行计划问题。
优先优化高并发的 SQL，而不是执行频率低某些“大”SQL
对于破坏性来说，高并发的 SQL 总是会比低频率的来得大，因为高并发的 SQL 一旦出现问题，
甚至不会给我们任何喘息的机会就会将系统压跨。而对于一些虽然需要消耗大量 IO 而且响应很慢的 SQL，
由于频率低，即使遇到，最多就是让整个系统响应慢一点，但至少可能撑一会儿，让我们有缓冲的机会。
从全局出发优化，而不是片面调整
SQL 优化不能是单独针对某一个进行，而应充分考虑系统中所有的 SQL，尤其是在通过调整索引优化 SQL
的执行计划的时候，千万不能顾此失彼，因小失大。
尽可能对每一条运行在数据库中的SQL进行 explain


优化 SQL，需要做到心中有数，知道 SQL 的执行计划才能判断是否有优化余地，才能判断是否存在执行计划问题。
在对数据库中运行的 SQL 进行了一段时间的优化之后，很明显的问题 SQL 可能已经很少了，大多都需要去发掘，
这时候就需要进行大量的 explain 操作收集执行计划，并判断是否需要进行优化。