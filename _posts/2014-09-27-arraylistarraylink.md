---
layout: post
title: arraylist&arraylink
category: java
date: 2014-09-27
---
**区别**：

1. array 在定义的时候必须实例化(至少申明大小)，arraylist则可以只是申明

    - int[] array = new array[3]
    - int[] array = {1,2,3}
    - ArrayList arrayList;
    - int[] array;则不合法
2. Array只能存储同构对象，ArrayList则可以存储异构对象，当然object[]和使用泛型的arraylist不适用此条。

3. 在CRL的托管方式不一样
    Array是连续存储，Arraylist则不一定是连续存储

4. Array的大小是定义的时候指定的，Arraylist则可以动态扩容
5. Array不能随意添加或删除其中的项，ArrayList则可以随意添加删除

**相似**：

1. 都具有索引，即可以通过索引来直接添加和修改任意项
2. 所创建的对象都托管在堆中
3. 都能够对自身进行枚举（因为实现了IEnumerable接口）

**ArrayList 源码摘要：**

	1. public class ArrayList<E> extends AbstractList<E>  
	2.         implements List<E>, RandomAccess, Cloneable, java.io.Serializable  
	3. {  
	4.     ......  
	5.   
	6.     /** 
	7.      * The array buffer into which the elements of the ArrayList are stored. 
	8.      * The capacity of the ArrayList is the length of this array buffer. 
	9.      */  
	10.     private transient E[] elementData;  
	11.   
	12.   
	13.     /** 
	14.      * The size of the ArrayList (the number of elements it contains). 
	15.      * 
	16.      * @serial 
	17.      */  
	18.     private int size;  
	19.    ......  
	20. }  
	21. 



底层两个属性一个object[] 一个size

1. ArrayList底层通过object[] 复制的方式（System.arraycopy()）来处理数组的增长
2. 当ArrayList 的容量不足时，其扩充容量的方式：先将容量扩充至当前容量的1.5倍，若还不够，则将容量扩充至当前需要的数量。

**ArrayList与Vector的区别**

1） vector 是线程同步 的，所以它也是线程安全 的，而arraylist 是线程异步 的，是不安全的 。如果不考虑到线程的安全因素，一般用 arraylist效率比较高。

2） 如果集合中的元素的数目大于目前集合数组的长度时，vector 增长率为目前数组长度的100%, 而arraylist 增长率为目前数组长度的50% .如果在集合中使用数据量比较大的数据，用vector有一定的优势。

3） 如果查找一个指定位置的数据 ，vector和arraylist使用的时间是相同 的，都是O(1) ,这个时候使用vector和arraylist都可以。

而如果移动一个指定位置的数据花费的时间为O(n-i)n为总长度 ，这个时候就应该考虑到使用linklist ,因为它移动一个指定位置的数据所花费的时间为0(1),而查询一个指定位置的数据时花费的时间为0(i)。

**ArrayList和LinkedList的大致区别：**

 1. ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。
 2. 对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
 3. 对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。 
 
**ArrayList和LinkedList在性能上各有优缺点，都有各自所适用的地方，总的说来可以描述如下：**
 
1. 对ArrayList和LinkedList而言，在列表末尾增加一个元素所花的开销都是固定的。对ArrayList而言，主要是在内部数组中增加一项，指向所添加的元素，偶尔可能会导致对数组重新进行分配；而对LinkedList而言，这个开销是统一的，分配一个内部Entry对象。
2. 在ArrayList的中间插入或删除一个元素意味着这个列表中剩余的元素都会被移动；而在LinkedList的中间插入或删除一个元素的开销是固定的。
3. LinkedList不支持高效的随机元素访问。
4. ArrayList的空间浪费主要体现在在list列表的结尾预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗相当的空间

可以这样说：当操作是在一列数据的后面添加数据而不是在前面或中间,并且需要随机地访问其中的元素时,使用ArrayList会提供比较好的性能；当你的操作是在一列数据的前面或中间添加或删除数据,并且按照顺序访问其中的元素时,就应该使用LinkedList了。

