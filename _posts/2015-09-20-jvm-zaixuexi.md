---
layout: post
title: JVM笔记
category: Java
date: 2015-09-20
share: true
duoshuo: true
---


JVM根据实现不同结构有所不同，大多数将内存分为：  
Method Area                        方法区  
Heap			                     堆
Program Counter register               程序计数器  
Java method stack                     Java方法栈   
native method stack                   本地方法栈 （Hot Spot 中将Java method和native合称为方法区）  
direct memory                       直接内存区（此区域并不归JVM管理）    

指令 数据 方法 实例 属性。。。  
方法是指令操作码的一部分保存在stack中  
方法内部变量作为指令的操作数部分，跟在指令的操作码之后，保存在Stack中（实际上是简单类型保存在Stack中，对象类型在Stack中保存地址，在Heap 中保存值）；指令操作码和指令操作数构成了完整的Java 指令。对象实例包括其属性值作为数据，保存在数据区Heap 中  
非静态的对象属性作为对象实例的一部分保存在Heap 中，而对象实例必须通过Stack中保存的地址指针才能访问到。因此能否访问到对象实例以及它的非静态属性值完全取决于能否获得对象实例在Stack中的地址指针。   

静态/非静态方法   
非静态方法有一个隐含的传入参数，该参数是JVM给它的，和我们怎么写代码无关，这个隐含的参数就是对象实例在Stack中的地址指针。So我们在调用前都要new,获得Stack中的地址指针。   
静态方法无此隐含参数，因此也不需要new对象，只要class文件被ClassLoader load进入JVM的Stack，该静态方法即可被调用。当然此时静态方法是存取不到Heap 中的对象属性的。   

静/动态属性   
实例以及动态属性都是保存在Heap 中的， Heap 必须通过Stack中的地址指针才能够被指令（类的方法）访问到。   
静态属性是保存在Stack中的，而不同于动态属性保存在Heap 中。正因为都是在Stack中，而Stack中指令和数据都是定长的，因此很容易算出偏移量，也因此不管什么指令，都可以访问到类的静态属性。也正因为静态属性被保存在Stack中，所以具有了全局属性。   
在JVM中，静态属性保存在Stack指令内存区，动态属性保存在Heap数据内存区。   


栈  
Stack（栈）是JVM的内存指令区。Stack的速度很快，管理很简单，并且每次操作的数据或者指令字节长度是已知的。所以Java 基本数据类型，Java 指令代码，常量都保存在Stack中。  
是 Java 程序的运行区，是在线程创建时创建，它的生命期是跟随线程的生命期，线程结束栈内存也就释放，对于栈来说不存在垃圾回收问题。  
栈中数据都是以栈帧（stack frame）的形式存在，栈帧是一个内存块一个有关方法和运行期数据的数据集，遵循 先进后出原则（方法 A 被调用时就产生了一个栈帧 F1，并被压入到栈中，A 方法又调用了 B 方法，于是产生栈帧 F2 也被压入栈，执行完毕后，先弹出 F2栈帧，再弹出 F1 栈帧）
栈帧中主要保存三类数据：  
local variable 本地变量（输入参数和输出参数以及方法内的变量）  
operand stack栈操作  （出栈进栈记录）  
frame data栈帧数据   （类文件 方法等）  


Heap  
JVM的内存数据区 管理复杂，用于保存对象的实例   
Heap 中分配一定的内存来保存对象实例，实际上也只是保存对象实例的属性值，属性的类型和对象本身的类型标记等，并不保存对象的方法（方法是指令，保存在Stack中）,在Heap 中分配一定的内存保存对象实例和对象的序列化比较类似。而对象实例在Heap 中分配好以后，需要在Stack中保存一个4字节的Heap 内存地址，用来定位该对象实例在Heap 中的位置，便于找到该对象实例。   

Java中堆是由所有的线程共享的一块内存区域。   
Heap 中分配一定的内存来保存对象实例，实际上也只是保存对象实例的属性值，属性的类型和对象本身的类型标记等，并不保存对象的方法（方法是指令，保存在Stack中）,在Heap 中分配一定的内存保存对象实例和对象的序列化比较类似。而对象实例在Heap 中分配好以后，需要在Stack中保存一个4字节的Heap 内存地址，用来定位该对象实例在Heap 中的位置，便于找到该对象实例。  
Java中堆是由所有的线程共享的一块内存区域。   

Perm   
Perm代主要保存class,method,filed对象，这部门的空间一般不会溢出，除非一次性加载了很多的类，不过在涉及到热部署的应用服务器的时候，有时候会遇到java.lang.OutOfMemoryError : PermGen space 的错误，造成这个错误的很大原因就有可能是每次都重新部署，但是重新部署后，类的class没有被卸载掉，这样就造成了大量的class对象保存在了perm中，这种情况下，一般重新启动应用服务器可以解决问题。    

Tenured   
Tenured区主要保存生命周期长的对象，一般是一些老的对象，当一些对象在Young复制转移一定的次数以后，对象就会被转移到Tenured区，一般如果系统中用了application级别的缓存，缓存中的对象往往会被转移到这一区间。  

Young  
Young区被划分为三部分，Eden区和两个大小严格相同的Survivor区，其中Survivor区间中，某一时刻只有其中一个是被使用的，另外一个留做垃圾收集时复制对象用，在Young区间变满的时候，minor GC就会将存活的对象移到空闲的Survivor区间中，根据JVM的策略，在经过几次垃圾收集后，任然存活于Survivor的对象将被移动到Tenured区间。  

The pc Register 程序计数器寄存器   
JVM支持多个线程同时运行。每个JVM都有自己的程序计数器。在任何一个点，每个JVM线程执行单个方法的代码，这个方法是线程的当前方法。如果方法不是native的，程序计数器寄存器包含了当前执行的JVM指令的地址，如果方法是 native的，程序计数器寄存器的值不会被定义。 JVM的程序计数器寄存器的宽度足够保证可以持有一个返回地址或者native的指针。  

Method Area 方法区  
Object Class Data(类定义数据) 是存储在方法区的。除此之外，常量、静态变量、JIT 编译后的代码也都在方法区。正因为方法区所存储的数据与堆有一种类比关系，所以它还被称为 Non-Heap。方法区也可以是内存不连续的区域组成的，并且可设置为固定大小，也可以设置为可扩展的，这点与堆一样。
方法区内部有一个非常重要的区域，叫做运行时常量池（Runtime Constant Pool，简称 RCP）。在字节码文件中有常量池（Constant Pool Table），用于存储编译器产生的字面量和符号引用。每个字节码文件中的常量池在类被加载后，都会存储到方法区中。值得注意的是，运行时产生的新常量也可以被放入常量池中，比如 String 类中的 intern() 方法产生的常量。    

内存分配：  
1、对象优先在EDEN分配  
2、大对象直接进入老年代   
3、长期存活的对象将进入老年代   
4、适龄对象也可能进入老年代：动态对象年龄判断  
动态对象年龄判断：  
虚拟机并不总是要求对象的年龄必须达到MaxTenuringThreshold才能晋升到老年代，当Survivor空间的相同年龄的所有对象大小总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无需等到MaxTenuringThreshold中指定的年龄  
  
1、对象优先在Eden分配，这里大部分对象具有朝生夕灭的特征，Minor GC主要清理该处  
2、大对象（占内存大）、老对象（使用频繁）  
3、Survivor无法容纳的对象，将进入老年代，Full GC的主要清理该处  

JVM的GC机制  
第一个线程负责回收Heap的Young区  
第二个线程在Heap不足时，遍历Heap，将Young 区升级为Older区  
Older区的大小等于-Xmx减去-Xmn，不能将-Xms的值设的过大，因为第二个线程被迫运行会降低JVM的性能。   
堆内存GC  
       JVM(采用分代回收的策略)，用较高的频率对年轻的对象(young generation)进行YGC，而对老对象(tenured generation)较少(tenured generation 满了后才进行)进行Full GC。这样就不需要每次GC都将内存中所有对象都检查一遍。  
非堆内存不GC  
      GC不会在主程序运行期对PermGen Space进行清理，所以如果你的应用中有很多CLASS(特别是动态生成类，当然permgen space存放的内容不仅限于类)的话,就很可能出现PermGen Space错误。  
内存申请过程   
1.JVM会试图为相关Java对象在Eden中初始化一块内存区域；  
2.当Eden空间足够时，内存申请结束。否则到下一步；  
3.JVM试图释放在Eden中所有不活跃的对象（minor collection），释放后若Eden空间仍然不足以放入新对象，则试图将部分Eden中活跃对象放入Survivor区；  
4.Survivor区被用来作为Eden及old的中间交换区域，当OLD区空间足够时，Survivor区的对象会被移到Old区，否则会被保留在Survivor区；  
5.当old区空间不够时，JVM会在old区进行major collection；  
6.完全垃圾收集后，若Survivor及old区仍然无法存放从Eden复制过来的部分对象，导致JVM无法在Eden区为新对象创建内存区域，则出现"Out of memory错误"；  

对象衰老过程   
1.新创建的对象的内存都分配自eden。Minor collection的过程就是将eden和在用survivor space中的活对象copy到空闲survivor space中。对象在young generation里经历了一定次数(可以通过参数配置)的minor collection后，就会被移到old generation中，称为tenuring。  
![](/image/jvm-memery.png)


类的加载方式   
1）：本地编译好的class中直接加载  
2）：网络加载：java.net.URLClassLoader可以加载url指定的类  
3）：从jar、zip等等压缩文件加载类，自动解析jar文件找到class文件去加载util类  
4）：从java源代码文件动态编译成为class文件  
 
类加载的时机  
1. 类加载的 生命周期 ：加载（Loading）-->验证（Verification）-->准备（Preparation）-->解析（Resolution）-->初始化（Initialization）-->使用（Using）-->卸载（Unloading）  
2. 加载：这有虚拟机自行决定。  
3. 初始化阶段：  
a) 遇到new、getstatic、putstatic、invokestatic这4个字节码指令时，如果类没有进行过初始化，出发初始化操作。   
b) 使用java.lang.reflect包的方法对类进行反射调用时。  
c) 当初始化一个类的时候，如果发现其父类还没有执行初始化则进行初始化。  
d) 虚拟机启动时用户需要指定一个需要执行的主类，虚拟机首先初始化这个主类。  
注意：接口与类的初始化规则在第三点不同，接口不要气所有的父接口都进行初始化。   




















双亲委派机制   
JVM在加载类时默认采用的是 双亲委派 机制。通俗的讲，就是某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才自己去加载。   
