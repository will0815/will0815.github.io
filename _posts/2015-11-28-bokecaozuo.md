---
layout: post
title: 博客操作记录
category: F2e
date: 2015-10-28
duoshuo: true
share: true
---

在_config.yml配置站点信息，详细配置如下：

blog:
	name:                  # 博客名称
	description:           # 博客描述
	title:                 # 网页标题
	url:                   # 博客地址
	duoshuo:               # 多说ID
	tongji:                # 百度统计ID
	qiniu:                 # 七牛云地址
author:
	name:                  # 作者名称
	email:                 # 邮箱地址
	weibo:                 # 微博地址
	github:                # GitHub地址
	douban:
		name:              # 豆瓣地址名称
		key:               # 豆瓣API Key

多说评论框
_posts文章默认开启评论框，而简版页面默认关闭。
_posts文章可以在开头设置duoshuo: false来关闭。
简版页面可以在开头设置duoshuo: true来开启。

MathJax数学公式
需要在页面开头添加math: true来开启

在需要用到公式的地方用\[ \]或$$ $$括起来

例如：

行内公式：
\\E=mc^2\\

行间公式：
$$E=mc^2$$
效果：

\E=mc^2\

E=mc2
创建文章/页面  
定位到博客目录，可以运行以下命令  
  
创建文章：rake post title="Post Name"  
创建简版页面：rake life title="Page Name"  
创建页面： rake page title="Page name"  
页面的使用   
修改的都是markdown文件   
   
普通页面   
layout项改为blog   
简版页面   
layout项改为life   
文章   
_posts文件夹下的markdown文件的layout项改为post，使用简版页面就改成life  
生成静态博客  
把你的博客推送到GitHub或者其它支持Jekyll的代码托管网站就可以了。  
具体可以到Jekyll官网或GitHub Pages查看详细教程。  
  


