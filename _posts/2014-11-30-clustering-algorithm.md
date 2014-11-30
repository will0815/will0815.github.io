---
layout: post
title: 聚类以及增量聚类的简单理解
category: hadoop
date: 2014-11-30
duoshuo: true
share: true
---
##聚类以及增量聚类的简单理解
K-Means聚类基本步骤如下：   

	Step1: 确定K值以及初始化聚类中心，选取K个初始凝聚点，做为欲形成的中心（最简单的方法就是K的值自己确定，但必须小于数据集个数，然后从数据集中选取K个数据做为初始聚类中心）；  
	Step2: 计算每个数据到K个初始凝聚点的距离，将每个数据和最近的凝聚点分到一组，形成K个初始分类；  
	Step3: 计算初始分类的重心（或均值），做为新的凝聚点，重新计算每个数据到分类重心（或均值）的距离，将每个数据和最近的凝聚点分为一组；
	Step4: 重复进行Step2和Step3，直到分类的重心或均值没有明显变化为止。

Canopy算法流程  
	
	      （1）、将数据集向量化得到一个list后放入内存，选择两个距离阈值：T1和T2，其中T1 > T2，对应上图，实线圈为T1，虚线圈为T2，T1和T2的值可以用交叉校验来确定；  
	      （2）、从list中任取一点P，用低计算成本方法快速计算点P与所有Canopy之间的距离（如果当前不存在Canopy，则把点P作为一个Canopy），如果点P与某个Canopy距离在T1以内，则将点P加入到这个Canopy；  
	      （3）、如果点P曾经与某个Canopy的距离在T2以内，则需要把点P从list中删除，这一步是认为点P此时与这个Canopy已经够近了，因此它不可以再做其它Canopy的中心了；  
	      （4）、重复步骤2、3，直到list为空结束。

单机版Canopy算法：  

	      1、从PointList中取一个Point ，寻找已经建立好的Canopy 计算这个点于所有的Canopy的距离。如果和某一个Canopy的距离小于T1，则把这个点加到Canopy中，如果没有Canopy则选择这个点为一个Canopy的中心。  
	      2、如果这个店Point和某个Canopy的距离小于T2,则把这个点从PointList中删除（这个点以后做不了其他的Canopy的中心了）。  
	      3、循环直到所有的Point都被加入进来，然后计算各个Canopy的Center和Radius。
模型MapReduce版本：

	  1、把数据整理成SequcnceFile格式（Mahout-InputMapper）作为初始化文件PointFile  
	  2、CanopyMapper阶段本机聚成小的Canopy 中间文件写成SequenceFile 这一步的T1、T2 和Reduce的T1、T2可以是不同的（ index、Canpy）  
	  3、所有的Mapper阶段的输出到1个Reducer中 然后Reduce把Map阶段中的Center点再次做聚类算法。聚出全局的Canopy。同时计算每个Canopy的CenterPoint点。写到临时文件CenterPoint中。  
	  4、针对全集合PointFile在CenterPoint上的findClosestCanopy操作（通过传入的距离算法）。然后输出一个SequenceFile。   
 


基于距离-增量聚类的实现方式

	1.
	每增加一个数据，立即(或延迟)计算它与其他对象之间
	的相异度度量值，并将其插入到R2中；检查计算出相异度
	度量值，当d(id，j)<δ时，如果id在R3中存在，则elements= 
	elements+1；否则在R3中插入一个新记录(id，1)。  
	
	2.
	数据的增加部分可以简单的加到原有聚类结果中,具体情况可以分两种: 
	(1)增加记录在原有聚类结果的某一类范围内(n维空间),这样这条记录的加入增加了这类的数据密度,因此可以直接把这条记录加入这一类当中. 
	(2)增加记录不在原来聚类结果的任何一类范围内,也就是相当于n维空间上的一个孤立的点,这种情况可暂把这条数据记录单独看成一类. 
