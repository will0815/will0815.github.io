---
layout: post
title: sql frist day
category: DB
date: 2014-09-27
---
计算周月的第一天， 以星期天为一周的开始

		SELECT LAST_DAY('2013-2-1');
		SELECT concat(date_format(LAST_DAY('2013-02-01'),'%Y-%m-'),'01');
		select DATE_SUB('2014-03-08',INTERVAL (WEEKDAY('2014-03-08')+1)%7 DAY);
		select DATE_SUB('2014-03-08',INTERVAL ((WEEKDAY('2014-03-08')+1)%7)-6 DAY);