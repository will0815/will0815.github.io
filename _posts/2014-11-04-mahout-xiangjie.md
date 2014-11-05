---
layout: post
title: mahout--初学
category: hadoop
date: 2014-11-04
---
##Mahout
**What is Mahout**

官方解释：  
The Apache Mahout project's goal is to build a scalable machine learning library.  
----即一个可扩展的机器学习程序库  
Mahout是Hadoop家族中与众不同的一个成员，他是基于一个Hadoop的机器学习和数据挖掘的分布式计算框架。是一个跨学科产品，是现在Hadoop生态环境中，对于普通开发者来说最具竞争力，最难掌握，也是最值得学习的一个项目之一。它为数据分析人员，降低了大数据分析的门槛；为算法工程师，提供基础的算法库；为Hadoop开发人员，提供了数据建模的标准。   
**Mahout 0.9 Applicable Models:  **

		• User and Item based recommenders （基于用户/物品推荐）
		• Matrix factorization based recommenders （基于矩阵因数分解推荐）
		• K-Means, Fuzzy K-Means clustering（K-Means 聚类）
		• Latent Dirichlet Allocation （三层贝叶斯概率模型）
		• Singular Value Decomposition （奇异值分解）
		• Logistic regression classifier （逻辑回归 LR分类器）
		• (Complementary) Naive Bayes classifier（朴素贝叶斯分类器）
		• Random forest classifier （随机森林基于决策树的分类器）
		• High performance java collections （高性能java集合(以前的colt集合)）
		• A vibrant community （社区）

**Mahout运行/开发环境**  
运行：  
Mahout运行可以指定运行在hadoop上或是直接本地运行。 默认是在hadoop环境上运行，Mahout运行时会读取profile文件中我们配置的HADOOP_HOME和HADOOP_CONF_DIR项读取hadoop环境，从而在得以运行于hadoop上。 
但是如果少量的数据分析可能不需要hadoop的介入，单机就能分析完成而且效率会更好，这时只需要在运行Mahout时加上参数 MAHOUT_LOCAL：设置是否本地运行，如果设置这个参数就不会运行hadoop了，一旦设置这个参数，那HADOOP_CONF_DIR 和HADOOP_HOME 这两个参数的设置就自动失效了。
开发：  
Mahout以maven构建，在开发Mahout相关程序时只需要在maven工程中引入Mahout相关依赖即可。如：  

	<dependencies>
			<dependency>
				<groupId>org.apache.mahout</groupId>
				<artifactId>mahout-core</artifactId>
				<version>${mahout.version}</version>
			</dependency>
			<dependency>
				<groupId>org.apache.mahout</groupId>
				<artifactId>mahout-integration</artifactId>
				<version>${mahout.version}</version>
				<exclusions>
					<exclusion>
						<groupId>org.mortbay.jetty</groupId>
						<artifactId>jetty</artifactId>
					</exclusion>
					<exclusion>
						<groupId>org.apache.cassandra</groupId>
						<artifactId>cassandra-all</artifactId>
					</exclusion>
					<exclusion>
						<groupId>me.prettyprint</groupId>
						<artifactId>hector-core</artifactId>
					</exclusion>
				</exclusions>
			</dependency>
		</dependencies>

坑：在实际开发中如果只是像上面引用Mahout依赖，然后maven打包放到hadoop环境中去运行会出现各种问题，如jar包不兼容丢失等等。
解决办法：参考Mahout-examples中的pom.xml配置加入 job.xml文件


**Mahout学习路线图：** 
![](/image/mahout1.png)  


**聚类算法(clustering algorithm)** 
***What is clustering***   
   Clustering--聚类,即将数据对象分组成为多个类或者簇 (Cluster)，它的目标：在同一个簇中的对象之间具有较高的相似度，而不同簇中的对象差别较大。即一个簇中的数据对象应当可以被作为一个整体来对待，从而减少计算量或者提高计算质量。   
聚类分析广泛的应用在许多应用中，包括模式识别，数据分析，图像处理以及市场研究。通过聚类，人们能意识到密集和稀疏的区域，发现全局的分布模式，以及数据属性之间的相互关系。  
最被广泛使用的既是对 Web 上的文档进行分类，组织信息的发布，给用户一个有效分类的内容浏览系统（门户网站），同时可以加入时间因素，进而发现各个类内容的信息发展，最近被大家关注的主题和话题，或者分析一段时间内人们对什么样的内容比较感兴趣，这些有趣的应用都得建立在聚类的基础之上。作为一个数据挖掘的功能，聚类分析能作为独立的工具来获得数据分布的情况，观察每个簇的特点，集中对特定的某些簇做进一步的分析，此外，聚类分析还可以作为其他算法的预处理步骤，简化计算量，提高分析效率。它是数据挖掘及机器学习领域内的重点问题之一  
   聚类是在给定的数据集合中寻找同类的数据子集合，每一个子集合形成一个类簇，同类簇中的数据具有更大的相似性。聚类算法大体上可分为基于划分的方法、基于层次的方法、基于密度的方法、基于网格的方法以及基于模型的方法。选择聚类算法是需要考虑如下几个方面：   

	1.聚类结果是排他的还是可重叠的
	  一个元素，他是否可以属于聚类结果中的多个簇中，如果是，则是一个可重叠的聚类问题，如果否，那么是一个排他的聚类问题。
	2. 基于层次还是基于划分.
	  基于划分：一组对象，按照一定的原则将它们分成不同的组，这是典型的划分聚类问题
	  基于层次：顶层将对象进行大致的分组，随后每一组再被进一步的细分，也许所有路径最终都要到达一个单独实例（或者自底向上）
	3. 簇数目固定的还是无限制的聚类
	   聚类问题是在执行聚类算法前已经确定聚类的结果应该得到多少簇，还是根据数据本身的特征，由聚类算法选择合适的簇的数目。
	4. 基于距离还是基于概率分布模型
	   基于距离的聚类问题很好理解，即距离近的相似的对象聚在一起
	   基于概率分布模型的聚类问题，就是在一组对象中，找到能符合特定分布模型的点的集合，他们不一定是距离最近的或者最相似的，而是能完美的呈现出概率分布模型所描述的模型。

**Mahout clustering**   
数据模型  
Mahout 的聚类算法将对象表示成一种简单的数据模型：向量 (Vector)。在向量数据描述的基础上，可以轻松的计算两个对象的相似性.Mahout 中的向量 Vector 是一个每个域是浮点数 (double) 的复合对象，最容易想到的实现就是一个浮点数的数组。但在具体应用由于向量本身数据内容的不同，比如有些向量的值很密集，每个域都有值；有些则是很稀疏，可能只有少量域有值，所以 Mahout 提供了多个实现：  

	1. DenseVector，它的实现就是一个浮点数数组，对向量里所有域都进行存储，适合用于存储密集向量。
	2. RandomAccessSparseVector 基于浮点数的 HashMap 实现的，key 是整形 (int) 类型，value 是浮点数 (double) 类型，它只存储向量中不为空的值，并提供随机访问。
	3. SequentialAccessVector 实现为整形 (int) 类型和浮点数 (double) 类型的并行数组，它也只存储向量中不为空的值，但只提供顺序访问。

**K-Maens:**  
K-Means算法是一种得到最广泛使用的基于划分的聚类算法，把n个对象分为k个簇，以使簇内具有较高的相似度。相似度的计算根据一个簇中对象的平均值来进行。算法首先随机地选择k个对象，每个对象初始地代表了一个簇的平均值或中心。对剩余的每个对象根据其与各个簇中心的距离，将它赋给最近的簇，然后重新计算每个簇的平均值。这个过程不断重复，直到准则函数收敛。
K-Means是典型的基于距离的排他的划分方法：给定一个 n 个对象的数据集，它可以构建数据的 k 个划分，每个划分就是一个聚类，并且 k<=n，同时还需要满足两个要求：  

 1. 每个组至少包含一个对象
 2. 每个对象必须属于且仅属于一个组。
 
基本原理:  

 1. 首先创建一个初始划分，随机地选择 k 个对象，每个对象初始地代表了一个簇中心。对于其他的对象，根据其与各个簇中心的距离，将它们赋给最近的簇。
 2. 然后采用一种迭代的重定位技术，尝试通过对象在划分间移动来改进划分。所谓重定位技术，就是当有新的对象加入簇或者已有对象离开簇的时候，重新计算簇的平均值，然后对对象进行重新分配。这个过程不断重复，直到没有簇中对象的变化。
 
优缺点：  
首先原理简单，实现起来也相对简单，同时执行效率和对于大数据量的可伸缩性还是较强的。当结果簇是密集的，而且簇和簇之间的区别比较明显时，K-Means的效果比较好。  
K-Means的最大问题是要求用户必须事先给出 k 的个数，k 的选择一般都基于一些经验值和多次实验结果，对于不同的数据集，k 的取值没有可借鉴性。另外，K 均值对“噪音”和孤立点数据是敏感的，少量这类的数据就能对平均值造成极大的影响。


基于 Hadoop 的 Mahout K-Means聚类算法实现（主要步骤代码）

      //将原始数据randomData.csv，指定向量类型，转成Mahout sequence files of VectorWritable。
        InputDriver.runJob(new Path(inPath), new Path(seqFile), "org.apache.mahout.math.RandomAccessSparseVector");
        //指定聚类的簇数
        //通过随机的方法，选中kmeans的3个中心，做为初始集群
        int k = 3;
        Path seqFilePath = new Path(seqFile);
        Path clustersSeeds = new Path(seeds);
        DistanceMeasure measure = new EuclideanDistanceMeasure();
        clustersSeeds = RandomSeedGenerator.buildRandom(conf, seqFilePath, clustersSeeds, k, measure);
        //利用K-means算法实现聚类分析
        //根据迭代次数的设置，执行MapReduce，进行 10次迭代计算
        KMeansDriver.run(conf, seqFilePath, clustersSeeds, new Path(outPath), measure, 0.01, 10, true, 0.01, false);
        Path outGlobPath = new Path(outPath, "clusters-*-final");
        Path clusteredPointsPath = new Path(clusteredPoints);
        System.out.printf("Dumping out clusters from clusters: %s and clusteredPoints: %s\n", outGlobPath, clusteredPointsPath);
        //结果输出
        ClusterDumper clusterDumper = new ClusterDumper(outGlobPath, clusteredPointsPath);
        clusterDumper.printClusters(null);


对于K-Means不足的补救办法：  
对于K-Means 需要事先指定簇数的不足，我们可以结合另一个聚类算法:canopy,从而解决这个不足。  
**Canopy 聚类算法：**  
Canopy 聚类算法的基本原则是：首先应用成本低的近似的距离计算方法高效的将数据分为多个组，这里称为一个 Canopy，Canopy 之间可以有重叠的部分；然后采用严格的距离计算方式准确的计算在同一 Canopy 中的点，将他们分配与最合适的簇中。Canopy 聚类算法经常用于 K 均值聚类算法的预处理，用来找合适的 k 值和簇中心。  
  
在Mahout的examples中的kmeans算法实现就是这种解决思维：

	public static void run(Configuration conf, Path input, Path output, DistanceMeasure measure, double t1, 	double t2, double convergenceDelta, int maxIterations)throws Exception{
		Path directoryContainingConvertedInput = new Path(output,DIRECTORY_CONTAINING_CONVERTED_INPUT);
		log.info("Preparing Input");
		InputDriver.runJob(input, directoryContainingConvertedInput,"org.apache.mahout.math.RandomAccessSparseVector");
		log.info("Running Canopy to get initial clusters");
		CanopyDriver.run(conf, directoryContainingConvertedInput, output, measure,t1, t2, false, false);
		log.info("Running KMeans");
		KMeansDriver.run(conf, directoryContainingConvertedInput, new Path(output,Cluster.INITIAL_CLUSTERS_DIR), output, measure,convergenceDelta,maxIterations, true, false);
		// run ClusterDumper
		ClusterDumper clusterDumper = new ClusterDumper(finalClusterPath(conf,
		output, maxIterations), new Path(output, "clusteredPoints"));
		clusterDumper.printClusters(null);
	}
CanopyDriver.run( ) ： 即用Canopy算法确定初始簇的个数和簇的中心。
KMeansDriver.run( ) ： K-means算法进行聚类。
