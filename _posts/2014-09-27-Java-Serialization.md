---
layout: post
title: Java Serialization
category: java
date: 2014-09-27
---
目的：

保存对象的状态，并且可以把保存的对象状态再读取出来进行恢复。

场景：

1. 需要把内存中对象状态保存到一个文件中或者数据库中时
2. 需要用套接字在网络上传送对象时
3. 需要通过RMI传输对象时
特点：
1. 序列化时 只对对象的状态进行保存 而不管对象的方法
2. 当一个父类实现序列化，子类自动实现序列化 无须显式实现Serializable借口
3. 当一个对象的实例变量引用其他对象，序列化该对象时也会把引用对象进行序列化
4. 并非所有对象都可以序列化：
socket/thread

Serialization 是一种将对象以一连串的字节描述的过程，deserialization 将这些字节重构成一个对象的过程。Java序列化API提供一种处理对象序列化的标准机制。

##进阶问题：##

**序列化ID的问题**

serialVersionID即 序列化的版本号，是为了保证不同版本之间的兼容性。向前兼容以及向后兼容。
两种生成方式：

1. 使用默认的1L
2. 根据类名 接口名 成员方法以及属性来生成

如果一个类实现了序列化 但没有显示的申明序列化版本号  在程序编译的时候会自动生成一个版本号。存储在文件中，如果类有改动版本号也会随之改变。 在反序列化的时候JVM会将文件中的ID与本地相应实体类的ID比较  两者一致才能够序列化成功。否则会抛InvalidClassException异常。


 静态化变量的序列化


                        //初始时staticVar为5
                        ObjectOutputStream out = new ObjectOutputStream(
                                        new FileOutputStream("result.obj"));
                        out.writeObject(new Test());
                        out.close();

                        //序列化后修改为10
                        Test.staticVar = 10;

                        ObjectInputStream oin = new ObjectInputStream(new FileInputStream(
                                        "result.obj"));
                        Test t = (Test) oin.readObject();
                        oin.close();
                        //再读取，通过t.staticVar打印新的值
                        System.out.println(t.staticVar);
以上代码的输出结果为10.。。因为序列化并不保存静态化变量。

**父类的序列化**

一个子类实现了Serializable接口  它的父类没有实现Serializable接口   序列化该子类对象然后反序列化输出父类定义的某变量的数值  该数值与序列化时的数值不同

 在父类没有实现Serializable接口时  JVM将不会序列化父类对象  而一个Java对象的构造必须有父类对象 所以在反序列化时 只能调用父类的无参构造器作为默认的父对象  所以这是可以在父类的无参构造器中对变量进行初始化 否则父类变量都是默认的值

**Transient关键字**

Transient关键字的作用是控制变量的序列化，在变量申明前加上次关键字可以阻止该变量被序列化

**序列化存储规则**

     ObjectOutputStream out = new ObjectOutputStream(
                                        new FileOutputStream("result.obj"));
        Test test = new Test();
        //试图将对象两次写入文件
        out.writeObject(test);
        out.flush();
        System.out.println(new File("result.obj").length());
        out.writeObject(test);
        out.close();
        System.out.println(new File("result.obj").length());

        ObjectInputStream oin = new ObjectInputStream(new FileInputStream(
                        "result.obj"));
        //从文件依次读出两个文件
        Test t1 = (Test) oin.readObject();
        Test t2 = (Test) oin.readObject();
        oin.close();
        //判断两个引用是否指向同一个对象
        System.out.println(t1 == t2);

同一对象两次写入文件，打印出写入一次对象后的存储大小和写入两次后的存储大小，然后从文件中反序列化出两个对象，比较这两个对象是否为同一对象。一般的思维是，两次写入对象，文件大小会变为两倍的大小，反序列化时，由于从文件读取，生成了两个对象，判断相等时应该是输入 false 才对，最后却为TRUE。

因为：Java 序列化机制为了节省磁盘空间，具有特定的存储规则，当写入文件的为同一对象时，并不会再将对象的内容进行存储，而只是再次存储一份引用，上面增加的 5 字节的存储空间就是新增引用和一些控制信息的空间。反序列化时，恢复引用关系，使得清单 3 中的 t1 和 t2 指向唯一的对象，二者相等，输出 true。该存储规则极大的节省了存储空间。


