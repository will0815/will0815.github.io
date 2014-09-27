---
layout: post
title: InputFormat实现流程
category: Hadoop
date: 2014-09-27
---
自定义InputFormat类步骤：

1. 继承InputFormat抽象类，InputFormat<K, V>

2. 根据具体需求逻辑，实现自定义key,value.
自定义key,value类，需要实现WritableComparable接口，
实现其中的readFields, write, compareTo 方法
readFields，write 分别实现对读写的序列化和反序列化
注：对于实现类的属性，在实现readFields和write方法是必须保持顺序一致。
compareTo方法，根据具体逻辑实现对实现对象的比较，用于map/reduce过程中的排序。

3. 在继承InputFormat抽象类后，需要实现其中的两个抽象方法。

		List<InputSplit> getSplits(JobContext context)
		RecordReader<K,V> createRecordReader(InputSplit split, TaskAttemptContext context )
	3.1

	List<InputSplit> getSplits(JobContext context)：
	在该方法中我们需要实现对待处理文件按需求进行分割，方法返回值List的大小即为map的个数。
	若需要我们自定义我们的分割方法我们需要需要继承InputSplit抽象类，该抽象类中拥有

		long getLength()        返回split的大小
		String[] getLocations() 返回获取数据的位置。
		注：在继承InputSplit类时，必须实现Writable接口，否则此split无法在hadoop中传递。
		实现Writable接口： 需要实现其中的 readFields，write 分别实现对读写的序列化和反序列化

	3.2

	RecordReader<K,V> createRecordReader(InputSplit split, TaskAttemptContext context )
	在该方法中我们需要实现根据split对数据的读取，返回值RecordReader<K, V> 即map的输入键值。
	由此我们需要继承RecodReader<K, V>抽象类，来实现我们的读取逻辑。
	该抽象类的中方法：

		abstract void initialize(InputSplit split, TaskAttemptContext context ) ：初始化方法，一般在此读取数据
		abstract boolean nextKeyValue()     ：返回是否还有下一个键值对，并对其赋值。
		abstract KEYIN getCurrentKey()      ：Get the current key
		abstract  VALUEIN getCurrentValue() ：Get the current value.
		abstract float getProgress()        ：获取处理进度
		abstract void close()               ：结束方法， 销毁资源之类的操作