---
layout: post
title: Map/Reduce第一次开发记录
category: Hadoop
date: 2014-09-27
---
1.main()
作用：配置job, 提交job
main()运行于JobTracker

2. map/reduce运行流程：

		map(k1, v1)-->(k2, v2)-->combiner(k2, v2)-->(k3, v3)-->reduce(k3, v3)-->(k4, v4)
		InputFormat  程序逻辑          map的输出               combiner的输出  最终输出

3. 不要在map类中自己实现combiner或reduce的功能。可能导致内存不足。
context.write();方法在当写入量超过一定值后，会写入文件。

4. setNumReduce() 设置reduce的个数，默认为1. 另外当map/reduce运行在本地时此方法的设置无效。
对于reduce的个数一般由metadata来决定。


5. map/reduce 程序的运行方式：

	a: 将写好的程序，打成jar包，将其和其依赖的jar包放置于 hadoop安装环境上 然后在hadoop上以 hadoop -jar /path/***.jar
	b：在eclipse中run
	c: 编写servlet, 在servlet 中调用a方法中的命令 运行程序。

5. Combiner

	在map端实现对key的聚合，然后再有reduce从聚合以后的数据中获取，从而减少map于reduce之间的数据传输，提高程序效率。
	另外使用Combiner,应考虑使用combiner对数据的聚合程度。如果聚合量太少，是没有必要的。

6. Partitioner  (shuffle)
	Partitioner 实现将map或是combiner的输出发送到对应的reduce的功能.
	可以自定义实现。默认为HashPartitioner

7. configuration：作用：实现对job的配置
	可以作为jobtracker,map,reduce之间的参数传递，configuration对象在设置后，在程序的任何地方都能获取。

8. 在本地run map/redcue 程序时 需要修改源码中FileUtil类如下：

		private static void checkReturnValue(boolean rv, File p,
		FsPermission permission
		) throws IOException {
		// if (!rv) {
		// throw new IOException("Failed to set permissions of path: " + p +
		// " to " +
		// String.format("%04o", permission.toShort()));
		// }
		}

可以把这个类copy到你eclipse工程中的源程序中，包名和类名都不变。
在eclipse中运行时将优先使用你copy过来修改的那个类。
没有必要重新编译整个hadoop的类。