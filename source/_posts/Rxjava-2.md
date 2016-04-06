---
title: Rxjava(2)
date: 2016-04-05 14:04:38
tags: Rxjava
---

###Rxjava响应式编程的好处  

####错误处理  

`Rxjava`中的错误处理模式有以下的特点:  

> 1、只要有异常发生onError()一定会被调用 ，这极大的简化了错误处理。只需要在一个地方处理错误即可以。
> 2、操作符不需要处理异常 ，将异常处理交给订阅者来做，Observerable的操作符调用链中一旦有一个抛出了异常，就会直接执行onError()方法。
> 3、你能够知道什么时候订阅者已经接收了全部的数据。知道什么时候任务结束能够帮助简化代码的流程。（虽然有可能Observable对象永远不会结束）


这种错误处理方式比传统的错误处理要简单，传统的错误处理过程中，通常需要回调处理每一个错误，这不仅导致了需要处理重复的代码，并且意味着每个回调都需要知道如何处理错误，回调代码和调用者紧密的耦合在一起  

在Rxjava中，`Observable`对象不需要知道如何处理错误，操作符也不需要处理错误状态，一旦发生了错误，就会跳过当前和后续的操作符，所有的错误处理都交给订阅者来做。  

####调度器  

编写多线程是Android中较为麻烦的事情，因为必须确保代码运行在正确的线程中，否则的话就能导致App崩溃，最常见的错误就是在Ui线程中访问了网络。  

在Rxjava中，可以使用`subscribeOn`指定被观察者代码运行的线程，`observerOn()`可以指定订阅者运行的线程。  

```java
myObservableServices.retrieveImage(url)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(bitmap -> myImageView.setImageBitmap(bitmap));
```  

任何在订阅者处理线程之前的嗲吗都是运行在I/O线程中，操作View的代码在主线程中运行。


####订阅  

当调用了`Observable.subscribe()`将会返回一个`subscribe`对象，这个对象将代表了被观察者和订阅者之间的联系  

```java
ubscription subscription = Observable.just("Hello, World!")
    .subscribe(s -> System.out.println(s));

//中断执行订阅

```  