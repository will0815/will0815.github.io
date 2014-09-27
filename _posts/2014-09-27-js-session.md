---
layout: post
title: js 基础session
category: F2E
date: 2014-09-27
---
1. html java javascript 都应遵循codingstyle
2. 事件不写在html中， 应按照mvc的模式 各个层之间应实现低耦合
m: 功能模块
v: html
c: 事件
3. Javascript 中字符串的拼接 应向Java中一样 使用buffer
4. ++ 和 +1 是有区别的 ++ 为原子操作， 推荐使用
5. Modle层的命名与表示层不应该有关联
应该只与功能有关。如delete/add之类
6. 常量字面量在比较时应该放在左边
7. var a = function(){};
funtion a(){};
之间的区别：
第一种是变量式声明，
第二种是定义式声明。
js在解析的时候，先解析定义式方法，在解析其他的
变量式和普通变量定义一样，
8. js的方法没有重载，只会覆盖
9. arguments 指向实参数组
方法对象的length指向型参长度
10. 方法在没有return时 返回undefind
11. 对象的创建方式：
a: new Object();
Object 的类型为Function
是由Function对象产生对象
b: jeson
var o = {};
o 对象类型为object
c: 构造函数
构造方法必须采用定义式， 首字母大写

12. this 执行当前的执行环境
13. 构造方法会有一个prototype对象。
而由构造方法产生的对象没有prototype对象

14. 继承的两种实现：
a:对象冒充
缺点： 冒充对象只能接受原对象里的东西，不能接收到其原型对象中的东西
b:原型链
15. 闭包

1. function
全局方法: this--->window对象
对象内方法： this--->当前对象
构造方法： 初始化构造方法需要new关键字， 否则就会当成全局方法定义
构造方法首字母大写

2. apply和call方法
apply（指定调用方法中的this, 方法的参数数组）;
call(指定调用方法中的this, 方法的参数列表);
foo.call(this, arg1,arg2,arg3) == foo.apply(this, arguments)==this.foo(arg1, arg2, arg3)

在JavaScript中,代码总是有一个上下文对象,代码处理该对象之内. 上下文对象是通过this变量来体现的, 这个this变量永远指向当前代码所处的对象中.

此类方法的应用场景：
A, B类都有一个message属性(面向对象中所说的成员),A有获取消息的getMessage方法,B有设置消息的setMessage方法
//创建一个B类实例对象
var b = new B();
//给对象a动态指派b的setMessage方法,注意,a本身是没有这方法的!
b.setMessage.call(a, "a的消息");
//下面将显示"a的消息"
alert(a.getMessage());

3. argments
方法的参数数组，一般在实现不定长参数的方法时会用到这个属性
可以通过它获得参数长度，参数值。
一般情况不通过它修改参数值。

 