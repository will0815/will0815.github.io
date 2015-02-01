---
layout: post
title: Linux学习笔记9-AWK
category: Linux
date: 2015-02-01
share: true
duoshuo: true
---
##Linux学习笔记9-AWK 
AWK是所有shell过滤工具中最难掌握的，不知道为什么，也许是其复杂的语法或含义不明确的错误提示信息。在学习awk语言过程中，就会慢慢掌握诸如Bailing out 和awk:cmd.Line:等错误信息。可以说awk是一种自解释的编程语言，之所以要在shell中使用awk是因为awk本身是学习的好例子，但结合awk与其他工具诸如grep和sed，将会使shell编程更加
容易。  
调用：  
有三种方式调用awk，第一种是命令行方式，如：  
awk [-F field-separator] 'awk命令' input-file(s)  
[-F域分隔符]是可选的，因为awk使用空格作为缺省的域分隔符，因此如果
要浏览域间有空格的文本，不必指定这个选项，但如果要浏览诸如passwd文件，此文件各域以冒号作为分隔符，则必须指明- F选项  
awk -F:'awk命令' input-file  
第二种方法是将所有awk命令插入一个文件，并使awk程序可执行，然后用awk命令解释器作为脚本的首行，以便通过键入脚本名称来调用它  
awk -f awk-script-file input-files  
第三种方式是将所有的awk命令插入一个单独文件. -f选项指明在文件awk_scriptfile中的awk脚本，inputfile(s)是使用awk进行浏览的文件
名。  
  
awk脚本  
在命令中调用awk时，awk脚本由各种操作和模式组成。  
如果设置了-F选项，则awk每次读一条记录或一行，并使用指定的分隔符分隔指定域，但如果未设置-F选项，awk假定空格为域分隔符，并保持这个设置直到发现一新行。当新行出现时，awk命令获悉已读完整条记录，然后在下一个记录启动读命令，这个读进程将持续到文件尾或文件不再存在。

