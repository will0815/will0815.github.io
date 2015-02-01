---
layout: post
title: Linux学习笔记7-正则表达式介绍
category: Linux
date: 2015-02-01
share: true
duoshuo: true
---
##Linux学习笔记7-正则表达式介绍
当从一个文件或命令输出中抽取或过滤文本时，可以使用正则表达式(RE)，正则表达式是一些特殊或不很特殊的字符串模式的集合。  
为了抽取或获得信息，我们给出抽取操作应遵守的一些规则。这些规则由一些特殊字符或进行模式匹配操作时使用的元字符组成。也可以使用规则字符作为模式中的一部分进行搜寻。  
例如，A将查询A，x将查找字母x    
基本元字符  
^ 只只匹配行首  
$ 只只匹配行尾  
* 只一个单字符后紧跟*，匹配0个或多个此单字符  
[ ] 只匹配[ ]内字符。可以是一个单字符，也可以是字符序列。可以使用-表示[ ]内字符序列范围，如用[1-5]代替[12345]  
\ 只用来屏蔽一个元字符的特殊含义。因为有时在shell中一些元字符有
特殊含义。\可以使其失去应有意义  
. 只匹配任意单字符  
pattern\{n\} 只用来匹配前面pattern出现次数。n为次数  
pattern\{n,\} m 只含义同上，但次数最少为n  
pattern\{n,m\} 只含义同上，但pattern出现次数在n与m之间  
  
1.使用句点匹配单字符  
句点"."可以匹配任意单字符。例如，如果要匹配一个字符串，以beg开头，中间夹一个任意字符，那么可以表示为beg.n，"."可以匹配字符串头，也可以是中间任意字符。  
在ls -l命令中，可以匹配一定权限：  
...x..x ..x  : 匹配用户本身，用户组及其他组成员的执行权限。  
"."允许匹配ASCII集中任意字符，或为字母，或为数字。  
2.在行首以^匹配字符串或字符序列  
^只允许在一行的开始匹配字符或单词。例如，使用ls -l命令，并匹配目录。之所以可以这样做是因为ls -l命令结果每行第一个字符是d，即代表一个目录。    
^...4XC....    
以上模式表示，在每行开始，匹配任意3个字符，后跟4XC，最后为任意4个字符。^在正则表达式中使用频繁，因为大量的抽取操作通常在行首.  
3.在行尾以$匹配字符串或字符    
$与^正相反，它在行尾匹配字符串或字符， $符号放在匹配单词后。假定要匹配以单词test结尾的所有行，操作为：  
test$  
^$  
具体分析为匹配行首，又匹配行尾，中间没有任何模式，因此为空行。  
如果只返回包含一个字符的行，操作如下：  
^.$  
4.使用*匹配字符串中的单字符或其重复序列    
使用此特殊字符匹配任意字符或字符串的重复多次表达式。例如：  
compu*t  
将匹配字符u一次或多次：  
compuut 
compuuut  
compuuuuut  
5.使用\屏蔽一个特殊字符的含义    
有时需要查找一些字符或字符串，而它们包含了系统指定为特殊字符的一个字符。什么是特殊字符？一般意义上讲，下列字符可以认为是特殊字符：  
$ . ' " * || ^ [] 0 \ + ?  
假定要匹配包含字符“ .”的各行而“，”代表匹配任意单字符的特殊字符，因此需要屏蔽其含义。操作如下：  
\.
6.使用[]匹配一个范围或集合    
使用[ ]匹配特定字符串或字符串集，可以用逗号将括弧内要匹配的不同字符串分开，但并不强制要求这样做（一些系统提倡在复杂的表达式中使用逗号），这样做可以增加模式的可读性。  
使用"-"表示一个字符串范围，表明字符串范围从"-"左边字符开始，到"-"右边字符结束。  
如果熟知一个字符串匹配操作，应经常使用[]模式。  
假定要匹配任意一个数字，可以使用：  
[0123456789]  
然而，通过使用“-”符号可以简化操作：  
[0-9]  
7.使用\{\}匹配模式结果出现的次数  
使用*可匹配所有匹配结果任意次，但如果只要指定次数，就应使用\ { \ }，此模式有三种形式，即：  
pattern\{n\} 匹配模式出现n次。  
pattern\{n,\} 匹配模式出现最少n次。  
pattern\{n,m} 匹配模式出现n到m次之间，n , m为0 -255中任意整数。
请看第一个例子，匹配字母A出现两次，并以B结尾，操作如下：  
A\{2\}B  
匹配值为AAB  
匹配A至少4次，使用：  
A\{4,\} B  
可以得结果AAAAB或AAAAAAAB，但不能为AAAB。  