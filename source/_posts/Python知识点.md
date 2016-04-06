---
title: Python知识点
date: 2016-04-06 16:49:22
tags: Python
---

###python学习

获取指定的日期或者是时间代码：  

		from datetime import datetime
		dt=datetime(2016,2,22,19,49)
		print(dt)
		//创建了一个当前时间，并且打印出来
		
在计算机中时间一般都是以数字来进行表示的，相对时间就是以1970年的时间为一个界限，相当于timestamp。也可以认为是当前的时间减去1970年的时间  
转换的代码如下所示：  

		from datetime import datetime
		t=1429417200.0
		print(datetime.fromtimestamp(t))
		
		//timestamp实际上是一个浮点数，没有时区的概念，但是  
		//datetime是存在时区的概念的，上述转换为了本地的时间
		//但是如果在其他的时区将会产生错误的时间，所以有必要要  
		//进行时区转换的操作  
		
时区转换  

		//获取到utc 时间，并且将时间强制设定到utc+8:00
		utc_dt=datetime.utcnow().replace(tzinfo=timezone.utc)
		//转换为北京时区
		utc_dt.astimezone(timezone(timedelta(hours=8)))  
		转换时区的关键在于拿到一个`datetime`要获取其正确的时区，然后要强制设置时区，作为基准时间  
		利用带时区的时间，任何带时区的时间都是可以正确的转换

使用集合collections  

collection是Python内建的一个集合模块，提供了很多有用的集合类  

> namedtuple ,tuple表示的是一个内容不变的集合，一个二维的坐标点就可以表示为：  
> p=(1,2)  
> 但是很难看出来这是表示的是一个坐标，但是定义个class同时也显得太小题大做了，所以这时候最简单的方式是使用namedtuple  
> 

		from collectionn import namedtuple
		Point=namedtuple('Point',['x','y'])
		p=Point(1,2)
		//这种简单的写作方式可以明确的声明一个类似C++ Struct结构体  
		
deque 
 
使用list 存储数据的时候，按照索引访问元素的速度较快，但是插入以及删除元素的速度就变得比较慢，因为list是线性存储，数据量大的时候，插入以及删除的效率就变得很低  

deque是为了实现高效的实现插入以及删除操作的双向列表，适合用于队列以及堆栈
