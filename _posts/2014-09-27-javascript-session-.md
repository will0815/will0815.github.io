---
layout: post
title: javascript session 记录
category: F2E
date: 2014-09-27
---
1. function
全局方法:    this--->window对象
对象内方法： this--->当前对象
构造方法：   初始化构造方法需要new关键字， 否则就会当成全局方法定义
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

4. 闭包问题


5. module

6. 继承，
javascript的继承发生在对象与对象之间，
可以通过以下方法实现：
objectCreate(o) {
var F = function() {};
F.prototype = o;
return new F();
}