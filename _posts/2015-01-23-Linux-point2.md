---
layout: post
title: Linux学习笔记2-find和xargs
category: Linux
date: 2015-01-23
share: true
duoshuo: true
---
Linux学习笔记2-find和xargs  
Find命令的一般形式为：  
find pathname -options [-print -exec -ok]  
pathname find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。  
-print find命令将匹配的文件输出到标准输出。  
-exec find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'comm -and'{} \;，注意{ }和\；之间的空格。  
-ok 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令,在执行每一个命令之前，都会给出提示，让用户来确定是否执行。   

find命令选项  
find命令有很多选项或表达式，每一个选项前面跟随一个横杠-。让我们先来看一下该命令的主要选项，然后再给出一些例子。  
-name 按照文件名查找文件。  
-perm 按照文件权限来查找文件。   
-prune 使用这一选项可以使find命令不在当前指定的目录中查找，如果同时使用了
- depth选项，那么- prune选项将被find命令忽略。  
-user 按照文件属主来查找文件。  
-group 按照文件所属的组来查找文件。  
-mtime -n +n 按照文件的更改时间来查找文件， 
- n表示文件更改时间距现在n天以内，+ n
表示文件更改时间距现在n天以前。
Find命令还有-atime和-ctime选项，但它们都和-mtime选项
相似，所以我们在这里只介绍-mtime选项。
-nogroup 查找无有效所属组的文件，即该文件所属的组在/etc/groups中不存在。  
-nouser 查找无有效属主的文件，即该文件的属主在/etc/passwd中不存在。  
-newer file1 !file2 查找更改时间比文件file1新但比文件file2旧的文件。  
-type 查找某一类型的文件，诸如：  
b - 块设备文件。  
d - 目录。  
c - 字符设备文件。  
p - 管道文件。  
l - 符号链接文件。  
f - 普通文件。  
-size n[c] 查找文件长度为n块的文件，带有c时表示文件长度以字节计。
-depth 在查找文件时，首先查找当前目录中的文件，然后再在其子目录中查找。  
-fstype 查找位于某一类型文件系统中的文件，这些文件系统类型通常可以在配置文件, /etc/fstab中找到，该配置文件中包含了本系统中有关文件系统的信息。
-mount 在查找文件时不跨越文件系统mount点。  
-follow 如果find命令遇到符号链接文件，就跟踪至链接所指向的文件。  
-cpio 对匹配的文件使用cpio命令，将这些文件备份到磁带设备中。 

