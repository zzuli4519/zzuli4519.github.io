---
title: RxJava
date: 2016-04-05 10:46:39

tags: Rxjava  

---


### Rxjava使用简介（一）  

Rxjava为响应式编程，与一般的命令式编程不同。`Rxjava`最核心的东西就是`Observables`(被观察者)以及`Subscribers`(观察者)。`Observables`发出的一系列的事件，观察者`Observables`负责处理这些事件，这里的事件可以是任何一种需要处理的东西（如触摸事件以及请求网络返回数据）  

一个`Observables`可以发出零个或者多个事件，知道事件结束或者是出错，每发出一个事件，就将会调用它的`Subscriber`的`onNext()`方法，最后调用`Subscriber.onNext()`或者是`Subscriber.onError()`结束  

Rxjava的看起来很想设计模式中的观察者模式，但是有一点明显不同，那就是如果一个Observerble没有任何的的Subscriber，那么这个Observable是不会发出任何事件的。  


创建一个基本的`Observables`对象很简单，直接调用`Observable.create()`即可。    

```java
Observable<String> myObservable = Observable.create(  
    new Observable.OnSubscribe<String>() {  
        @Override  
        public void call(Subscriber<? super String> sub) {  
            sub.onNext("Hello, world!");  
            sub.onCompleted();  
        }  
    }  
); 
```  
这里定义的被观察者对象仅仅只是发出了一个`Hello World`字符串，然后就结束了，接着我们需要创建一个观察者对象来接受被观察者发出的字符串。  

```java
Subscriber<String> mySubscriber = new Subscriber<String>() {  
    @Override  
    public void onNext(String s) { System.out.println(s); }  
  
    @Override  
    public void onCompleted() { }  
  
    @Override  
    public void onError(Throwable e) { }  
};  
```  

然后需要将观察者以及被观察者对象都绑定起来    

```java
	myObservable.subscribe(mySubscriber);  
```  

在使用的过程中也可以使用更简洁的代码来时间被观察者对象的创建过程。RxJava内置了很多的简化`Observable`对象的函数，比如`Observable.just()`就是用来创建只发出一个事件就结束的`Observable`对象，上面创建`Observable`对象的代码就可以简化为一行。  

```java
Observable<String> myObservable = Observable.just("Hello, world!"); 
```   

如果在上面的例子中，我们不关心`onComplete()`以及`onError()`我们只需要在`onNext()`中做出一些处理，这时候就可以使用`Action1`类  

```java
Action1<String> onNextAction = new Action1<String>() {  
    @Override  
    public void call(String s) {  
        System.out.println(s);  
    }  
};  
```  

`subscribe`方法有三个重载版本，接受了三个`Action1`类型的参数，分别对应的是`onNext()`、`onComplete()`、`onError()`函数  

```java
myObservable.subscribe(onNextAction, onErrorAction, onCompleteAction); 
```  

如果不关心其他的参数，只需要第一个参数就可以了  

```java
myObservable.subscribe(onNextAction);  
```  
上面的代码则最终可以写成如下的样子：  

```java
Observable.just("Hello, world!")  
    .subscribe(new Action1<String>() {  
        @Override  
        public void call(String s) {  
              System.out.println(s);  
        }  
    });  

使用Java的 Iambda 则可以简化代码如
Observable.just("Hello, world!").subscribe(s->System.out.println(s));
```    

在Android开发的过程中可以使用`retrolambda`这个`gradle`插件，这样就可以在代码中使用`java8`的`Iambda`表达式了  

在`Rxjava`中支持了操作符来对参数进行过滤。可以解决对`Observable`对象进行变换的问题，操作符用在`Observable`和最终的`Subscriber`之间修改`Observable`发出的事件。`Rxjava`提供了很多的操作符，如`map`操作符就是把一个事件转化为另一个事件的。







