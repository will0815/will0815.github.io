---
layout: post
title: Java class loader
category: java
date: 2014-09-27
---
**java classloader 结构：**

1. Bootstrap classloader(启动类加载器)
主要负责jdk_home/lib目录下的核心API或-Xbootclasspath选项指定的jar包的装入工作
2. Extension Classloader(扩展类加载器)
负责jdk_home/lib/ext目录下的jar包或者-Djava.ext.dirs指定目录下的jar包装入工作
3.System classloader(系统类加载器)
负责Java -classpath/-Djava.class.path所指定的类与jar包的装入
4. User Custom classloader（用户自定义类加载器（java.lang.ClassLoader的子类））
以上每一个classloader都会维护一份自己的名称空间，同一个名称空间不能出现两个同名的类。
为了实现Java安全沙箱模型顶层的类加载器安全机制，Java默认采用 双亲委派的加载链 结构

在jvm启动时就会构建bootstrap classloader 负责Java平台核心库

**每个ClassLoader加载Class的过程是：**

1. 检测此Class是否载入过（即在cache中是否有此Class），如果有到8,如果没有到2
2. 如果parent classloader不存在（没有parent，那parent一定是bootstrap classloader了），到4
3. 请求parent classloader载入，如果成功到8，不成功到5
4. 请求jvm从bootstrap classloader中载入，如果成功到8
5. 寻找Class文件（从与此classloader相关的类路径中寻找）。如果找不到则到7.
6. 从文件中载入Class，到8.
7. 抛出ClassNotFoundException.
8. 返回Class.





