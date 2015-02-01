---
layout: post
title: Linux学习笔记6-命令执行顺序
category: Linux
date: 2015-02-01
share: true
duoshuo: true
---
##Linux学习笔记6-命令执行顺序
场景：执行某个命令的时候，有时需要依赖于前一个命令是否执行成功。例如，假设你希望  
将一个目录中的文件全部拷贝到另外一个目录中后，然后删除源目录中的全部文件。在删除之前，你希望能够确信拷贝成功，否则就有可能丢失所有的文件。    
1.&&  
&&的一般形式：  
命令1 && 命令2  
以上表示：命令1返回真(返回0，成功执行)后，命令2才能被执行  
如：mv /apps/bin /apps/dev/bin && rm -r /apps/bin  
2.||    
||一般形式：  
命令1||命令2  
以上表示：命令1未执行成功，那么就执行命令2。  
3.()和{}  
如果希望把几个命令合在一起执行，shell提供了两种方法。既可以在当前shell也可以在子shell中执行一组命令。  
为了在当前shell中执行一组命令，可以用命令分隔符隔开每一个命令，并把所有的命令  
用圆括号()括起来。  
它的一般形式为：  
(命令1;命令2;. . .)  
如果使用{}来代替()，那么相应的命令将在子shell而不是当前shell中作为一个整体被执行，只有在{}中所有命令的输出作为一个整体被重定向时，其中的命令才被放到子shell中执行，否则在当前shell执行。它的一般形式为：    
{命令1;命令2;. . . }  