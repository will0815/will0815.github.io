---
layout: post
title: Linux 学习笔记5 shell输入输出
category: Linux
date: 2015-01-23
share: true
duoshuo: true
---
Linux 学习笔记5 shell输入输出  
shell脚本中几种读入数据方式：标准输入(缺省为键盘)，指定一个文件作为输入  
输出：标准输出(终端屏幕)，指定输出到一个文件  

echo  
echo命令可以显示文本行或是变量 或者把字符串输入到文件 一般形式为：
echo string  
常见功能：  
\c 不换行   
\t 跳格  
\n 换行  
初涉shell的用户常常会遇到的一个问题就是如何把双引号包含到echo命令的字符串中。  
引号是一个特殊字符，所以必须要使用反斜杠\来使shell忽略它的特殊含义。假设你希望使用echo命令输出这样的字符串："/dev/rmt0"，那么我们只要在引号前面加上反斜杠\即可   

read   
可以使用read语句从键盘或文件的某一行文本中读入信息，并将其赋给一个变量。如果只指定了一个变量，那么read将会把所有的输入赋给该变量，直至遇到第一个文件结束符或回车。  
它的一般形式为：  
read varible1 varible2 ...  


