---
layout: post
title: hbase import export
category: Hadoop
date: 2014-09-27
---
1. 导出HFlie
使用org.apache.hadoop.hbase.mapreduce.Export类
参数: MR配置参数 需导出的表 导出路径 时间戳版本（默认为1）时间戳起始结束时间 可加正则的RowFilter或PrefixFilter

		Export [-D <property=value>]* <tablename> <outputdir> [<versions> [<starttime> [<endtime>]] [^[regex pattern] or [Prefix] to filter]]
		Note: -D properties will be applied to the conf used.
		For example:
		-D mapred.output.compress=true
		-D mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec
		-D mapred.output.compression.type=BLOCK
		Additionally, the following SCAN properties can be specified
		to control/limit what is exported..
		-D hbase.mapreduce.scan.column.family=<familyName>
		-D hbase.mapreduce.include.deleted.rows=true
		For performance consider the following properties:
		-Dhbase.client.scanner.caching=100
		-Dmapred.map.tasks.speculative.execution=false
		-Dmapred.reduce.tasks.speculative.execution=false
		可直接运行也可以利用Driver如：
		hbase org.apache.hadoop.hbase.mapreduce.Driver export <hbase 表> <导出路径>
		hbase org.apache.hadoop.hbase.mapreduce.Driver export my_table /tmp/my_file

2. 导入HFlie

		使用org.apache.hadoop.hbase.mapreduce.Import类
		参数: Import [options] <tablename> <inputdir>
		By default Import will load data directly into HBase. To instead generate
		HFiles of data to prepare for a bulk data load, pass the option:
		-Dimport.bulk.output=/path/for/output
		For performance consider the following options:
		-Dmapred.map.tasks.speculative.execution=false
		-Dmapred.reduce.tasks.speculative.execution=false
		可直接运行也可以用Driver如：
		hbase org.apache.hadoop.hbase.mapreduce.Driver import <hbase 表> <导入路径>
		hbase org.apache.hadoop.hbase.mapreduce.Driver import my_table /tmp/my_file

3. 批量导入HFlie（导入到已存在的表）

		org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles
		参数: completebulkload </path/to/hfileoutputformat-output> <tablename>
		可直接运行也可以用Driver如：
		hbase org.apache.hadoop.hbase.mapreduce.Driver completebulkload <导入路径> <hbase 表>
		hbase org.apache.hadoop.hbase.mapreduce.Driver completebulkload /tmp/my_file my_table

	后面几个工具MR没有测试：

4. 导入TSV文件

		org.apache.hadoop.hbase.mapreduce.ImportTsv类
		参数: importtsv -Dimporttsv.columns=a,b,c <tablename> <inputdir>
		必须用-Dimporttsv.columns指定导入的TSV文件的列，列与列之间用逗号隔开
		一个列对应hbase的columnfamily，

		By default importtsv will load data directly into HBase. To instead generate
		HFiles of data to prepare for a bulk data load, pass the option:
		-Dimporttsv.bulk.output=/path/for/output
		Note: if you do not use this option, then the target table must already exist in HBase
		Other options that may be specified with -D include:
		-Dimporttsv.skip.bad.lines=false - fail if encountering an invalid line
		'-Dimporttsv.separator=|' - eg separate on pipes instead of tabs
		-Dimporttsv.timestamp=currentTimeAsLong - use the specified timestamp for the import
		-Dimporttsv.mapper.class=my.Mapper - A user-defined Mapper to use instead of org.apache.hadoop.hbase.mapreduce.TsvImporterMapper
		For performance consider the following options:
		-Dmapred.map.tasks.speculative.execution=false
		-Dmapred.reduce.tasks.speculative.execution=false
		可直接运行也可以用Driver如：
		hbase org.apache.hadoop.hbase.mapreduce.Driver importtsv Dimporttsv.columns=name,age <hbase 表> <导入路径>
		hbase org.apache.hadoop.hbase.mapreduce.Driver importtsv Dimporttsv.columns=name,age my_table /tmp/my_tsvFile

5. 表复制

	org.apache.hadoop.hbase.mapreduce.CopyTable类

	Usage: CopyTable [general options] [--starttime=X] [--endtime=Y] [--new.name=NEW] [--peer.adr=ADR] <tablename>
	Options:
	rs.class hbase.regionserver.class of the peer cluster
	specify if different from current cluster
	rs.impl hbase.regionserver.impl of the peer cluster
	starttime beginning of the time range (unixtime in millis)
	without endtime means from starttime to forever
	endtime end of the time range. Ignored if no starttime specified.
	versions number of cell versions to copy
	new.name new table's name
	peer.adr Address of the peer cluster given in the format
	hbase.zookeeer.quorum:hbase.zookeeper.client.port:zookeeper.znode.parent
	families comma-separated list of families to copy
	To copy from cf1 to cf2, give sourceCfName:destCfName.
	To keep the same name, just give "cfName"
	all.cells also copy delete markers and deleted cells
	Args:
	tablename Name of the table to copy
	Examples:
	To copy 'TestTable' to a cluster that uses replication for a 1 hour window:
	$ bin/hbase org.apache.hadoop.hbase.mapreduce.CopyTable
	--starttime=1265875194289
	--endtime=1265878794289
	--peer.adr=server1,server2,server3:2181:/hbase
	--families=myOldCf:myNewCf,cf2,cf3 TestTable
	For performance consider the following general options:
	-Dhbase.client.scanner.caching=100
	-Dmapred.map.tasks.speculative.execution=false

6. Compares the data

	org.apache.hadoop.hbase.mapreduce.replication.VerifyReplication类
	Usage: verifyrep [--starttime=X] [--stoptime=Y] [--families=A] <peerid> <tablename>
	Options:
	starttime beginning of the time range
	without endtime means from starttime to forever
	stoptime end of the time range
	families comma-separated list of families to copy
	Args:
	peerid Id of the peer used for verification, must match the one given for replication
	tablename Name of the table to verify
	Examples:
	To verify the data replicated from TestTable for a 1 hour window with peer #5
	$ bin/hbase org.apache.hadoop.hbase.mapreduce.replication.VerifyReplication --starttime=1265875194289 --stoptime=1265878794289 5 TestTable

 

实例记录：

1. Export Production HBase table as following:
hbase org.apache.hadoop.hbase.mapreduce.Export 'mytable' /hbase_bak/mytable
2. Copy HDFS files to local:
hadoop fs -copyToLocal /hbase_bak/mytable /home/user/hbase_bak/mytable

3. Copy the local files from production to production extension server

4. Copy local file to HDFS:
hadoop fs -copyFromLocal /home/user/hbase_bak/mytable /hbase_bak/mytable
5. Create table in production extension
create 'mytable', {NAME=>'log', VERSION=> 1}
6. Import the HDFS files into HBase:
hbase org.apache.hadoop.hbase.mapreduce.Import 'mytable' /hbase_bak/mytable
7. Verify the result:
hbase shell
count mytable

Find both the row count are same in production and production extension.

 

 

 