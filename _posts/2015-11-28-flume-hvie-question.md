---
layout: post
title: flume与hive/impala配合使用时问题记录
category: Hadoop
date: 2015-11-28
duoshuo: true
share: true
---

### 工作中日志收集以及分析  通过Flume+hive/impala实现 期间遇到一些问题  记录：  
####  问题一：
使用过程中时常会遇到.tmp文件无法找到的问题。  
impala 查询时这个问题尤为严重。 hive查询时也会时常出现这个问题。
明白Flume写文件处理方式，以及hive/impala 数据查询时的实际处理方式，就能明白为什么会出现这些问题

1. Flume日志写入操作时 会将正在写入的文件加上.tmp后缀。 当文件写入完成（一定时间没有写入/达到设置的阀值）时，会将./tmp文件重命名，去掉.tmp后缀
2. Hive查询时是根据hive元数据库中的信息进行文件扫描，即对应map/reduce的输入。
3. Impala共享hive的元数据
 
所以这时就会出现上面遇到的问题
1. 如果Hive在执行过程中，恰好Flume对写入文件进行了重命名操作，即将xxx.tmp重命名为xxx，这时Hive会报错  .tmp找不到。。。
2. impala这个错  更加平凡是因为impala同步hive元数据并不是实时操作，所以xxx.tmp被重命名掉的概率更大

###### 处理办法：  
1. impala的简单解决办法 即手动同步元数据。 执行refresh 文件对应的表， 或者直接执行 invalidate metadata 刷新元数据

2. hive的处理办法 
因为hdfs java api 读取HDFS文件时，会忽略以”.”和”_”开头的文件
类似于Linux中.xx是隐藏的一样，所以应用程序读取HDFS文件时默认也不读取.xxx和_xxx这样名称的文件
所以可以利用这一点 将Flume正在写入的.tmp文件设置为以‘.’开头，这样可以避免.tmp文件被读取到
Flume配置项：hdfs.inUsePrefix
另一种解决办法 重写hive的PathFilter

		package com.twitter.util;
		
		import java.io.IOException;
		import java.util.ArrayList;
		import java.util.List;
		import org.apache.hadoop.fs.Path;
		import org.apache.hadoop.fs.PathFilter;
		
		public class FileFilterExcludeTmpFiles implements PathFilter {
		    public boolean accept(Path p) {
		        String name = p.getName();
		        return !name.startsWith(“_”) && !name.startsWith(“.”) && !name.endsWith(“.tmp”);
		    }
		}

然后在hive-site.xml中加入：
		<property>
		    <name>mapred.input.pathFilter.class</name>
		    <value>com.twitter.util.FileFilterExcludeTmpFiles</value>
		</property>



### 问题二：
数据写入延时
日志上报端 反映日志上报到集群后 查询时有不同情况的延时问题。 但自己测试时有几乎没有延时的问题。。。
考虑如下：
1. Flume channel选择 （but 已是内存）
2. hdfs写入机制，hdfs文件按block存储，正在写入的block 对文件系统是隐藏的。
hdfs文件原则：  
创建文件，这个文件可以立即可见
写入文件的数据则不被保证可见了，哪怕是执行了刷新操作(flush/sync)。只有数据量大于1个BLOCK时，第一个BLOCK的数据才会被看到，后续的BLOCK也同样的特性。正在写入的BLOCK始终不会被其他用户看到！ 
基于以上两点  遇到的延时问题 基本上判断是第二点引起的。
###### 处理办法：
调小Flume写入文件分割的阀值：hdfs.rollInterval。
现在是按小时分割，是否可以考虑按分钟分割，
这样又将产生大量小文件的问题，那么还可以考虑对以前的每天的小时文件合并为每天一个文件！

