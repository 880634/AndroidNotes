---
title: RxJava基本概念
date: 2016-07-20 19:50:30
categories: RxJava
tags: RxJava
---



# RxJava基本概念

# 是什么
RxJava 在 GitHub 主页上的自我介绍是`a library for composing asynchronous and event-based programs using observable sequences for the Java VM"`（一个在 `Java VM` 上使用可观测的序列来组成异步的、基于事件的程序的库）。` RxJava` 的本质可以压缩为异步这一个词。说到根上，它就是一个实现异步操作的库，而别的定语都是基于这之上的。
`RxJava` 好在哪？就好在简洁，好在那把什么复杂逻辑都能穿成一条线的简洁。

<!--more-->
# RxJava 的观察者模式
`RxJava `有四个基本概念：
> `Observable` (可观察者，即被观察者)
`Observer `(观察者)
`subscribe `(订阅)、
事件

`Observable` 和` Observer` 通过 `subscribe()` 方法实现订阅关系，从而` Observable` 可以在需要的时候发出事件来通知 `Observer`。

与传统观察者模式不同，` RxJava` 的事件回调方法除了普通事件` onNext() `之外，还定义了两个特殊的事件：`onCompleted() `和 `onError()`。

> 
- `onCompleted()`: 事件队列完结。`RxJava` 不仅把每个事件单独处理，还会把它们看做一个队列。`RxJava` 规定，当不会再有新的` onNext()`发出时，需要触发` onCompleted()` 方法作为标志。
- `onError()`: 事件队列异常。在事件处理过程中出异常时，`onError()` 会被触发，同时队列自动终止，不允许再有事件发出。

在一个正确运行的事件序列中, `onCompleted()` 和 `onError()` 有且只有一个，并且是事件序列中的最后一个。需要注意的是，`onCompleted() `和 `onError()` 二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。

# 基本使用
可分为三点：

## 1. 创建观察者Observer
观察者决定事件触发的时候将有怎样的行为。` RxJava `中的` Observer` 接口的实现方式：
```java
Observer<String> observer = new Observer<String>() {
    @Override
    public void onNext(String s) {
        Log.d(tag, "Item: " + s);
    }

    @Override
    public void onCompleted() {
        Log.d(tag, "Completed!");
    }

    @Override
    public void onError(Throwable e) {
        Log.d(tag, "Error!");
    }
};
```
`RxJava` 还内置了一个实现了` Observer` 的抽象类：`Subscriber`。 `Subscriber` 对`Observer` 接口进行了一些扩展，但他们的基本使用方式是完全一样的：
```java
Subscriber<String> subscriber = new Subscriber<String>() {
    @Override
    public void onNext(String s) {
        Log.d(tag, "Item: " + s);
    }

    @Override
    public void onCompleted() {
        Log.d(tag, "Completed!");
    }

    @Override
    public void onError(Throwable e) {
        Log.d(tag, "Error!");
    }
};
```

实质上，在` RxJava `的` subscribe` 过程中，`Observer` 也总是会先被转换成一个 `Subscriber` 再使用。所以如果你只想使用基本功能，选择` Observer` 和 `Subscriber` 是完全一样的。它们的区别对于使用者来说主要有两点：

> 
- onStart(): 这是 `Subscriber` 增加的方法。它会在` subscribe` 刚开始，而事件还未**发送**之前被调用，可以用于做一些准备工作，例如数据的清零或重置。这是一个可选方法，默认情况下它的实现为空。需要注意的是，如果对准备工作的线程有要求（例如弹出一个显示进度的对话框，这必须在主线程执行），` onStart()` 就不适用了，因为它总是在 `subscribe `所发生的线程被调用，而**不能指定线程**。要在指定的线程来做准备工作，可以使用 `doOnSubscribe()` 方法，具体可以在后面的文中看到。
- unsubscribe(): 这是 `Subscriber` 所实现的另一个接口 `Subscription` 的方法，用于取消订阅。在这个方法被调用后`，Subscriber` 将不再接收事件。一般在这个方法调用前，可以使用` isUnsubscribed()` 先判断一下状态。 `unsubscribe()` 这个方法很重要，因为在` subscribe() `之后， `Observable `会持有 `Subscriber` 的引用，这个引用如果不能及时被释放，将有内存泄露的风险。所以最好保持一个原则：要在不再使用的时候尽快在合适的地方（例如 `onPause() onStop() `等方法中）调用 `unsubscribe()` 来解除引用关系，以避免内存泄露的发生。

## 2.创建 Observable

`Observable` 即被观察者，它决定什么时候触发事件以及触发怎样的事件。 `RxJava` 使用 `create() `方法来创建一个 `Observable` ，并为它定义事件触发规则：
```java
Observable observable = Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        subscriber.onNext("Hello");
        subscriber.onNext("Hi");
        subscriber.onNext("Aloha");
        subscriber.onCompleted();
    }
});
```
可以看到，这里传入了一个` OnSubscribe `对象作为参数。`OnSubscribe` 会被存储在返回的` Observable` 对象中，它的作用相当于一个计划表，当 `Observable` 被订阅的时候，`OnSubscribe` 的 `call()` 方法会自动被调用，事件序列就会依照设定依次触发（对于上面的代码，就是观察者`Subscriber` 将会被调用三次` onNext()` 和一次 `onCompleted()`）。这样，由被观察者调用了观察者的回调方法，就实现了由被观察者向观察者的事件传递，即观察者模式。

`create()` 方法是` RxJava` 最基本的创造事件序列的方法。基于这个方法，` RxJava` 还提供了一些方法用来快捷创建事件队列，例如：

- `just(T...)`: 将传入的参数依次发送出来。
```java
Observable observable = Observable.just("Hello", "Hi", "Aloha");
// 将会依次调用：
// onNext("Hello");
// onNext("Hi");
// onNext("Aloha");
// onCompleted();
```
- `from(T[]) / from(Iterable<? extends T>) `: 将传入的数组或 `Iterable` 拆分成具体对象后，依次发送出来。
```java
String[] words = {"Hello", "Hi", "Aloha"};
Observable observable = Observable.from(words);
// 将会依次调用：
// onNext("Hello");
// onNext("Hi");
// onNext("Aloha");
// onCompleted();
```
上面 `just(T...) `的例子和` from(T[])` 的例子，都和之前的 `create(OnSubscribe) `的例子是等价的。

## 3.Subscribe (订阅)

创建了` Observable` 和 `Observer `之后，再用 `subscribe()` 方法将它们联结起来，整条链子就可以工作了。代码形式很简单：
```java
observable.subscribe(observer);
// 或者：
observable.subscribe(subscriber);
```
`Observable.subscribe(Subscriber)` 的内部实现是这样的（仅核心代码）：
```java
// 注意：这不是 subscribe() 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
// 如果需要看源码，可以去 RxJava 的 GitHub 仓库下载。
public Subscription subscribe(Subscriber subscriber) {
    subscriber.onStart();
    onSubscribe.call(subscriber);
    return subscriber;
}
```
可以看到，`subscriber()` 做了3件事：

- 调用` Subscriber.onStart()` 。这个方法在前面已经介绍过，是一个可选的准备方法。
- 调用 `Observable `中的 `OnSubscribe.call(Subscriber)` 。在这里，事件发送的逻辑开始运行。从这也可以看出，在 `RxJava` 中， `Observable` 并不是在创建的时候就立即开始发送事件，而是在它被订阅的时候，即当` subscribe()`方法执行的时候。
- 将传入的` Subscriber `作为 `Subscription` 返回。这是为了方便 `unsubscribe()`.


除了` subscribe(Observer)` 和 `subscribe(Subscriber) `，`subscribe()` 还支持不完整定义的回调`，RxJava` 会自动根据定义创建出` Subscriber `。形式如下：
```java
Action1<String> onNextAction = new Action1<String>() {
    // onNext()
    @Override
    public void call(String s) {
        Log.d(tag, s);
    }
};
Action1<Throwable> onErrorAction = new Action1<Throwable>() {
    // onError()
    @Override
    public void call(Throwable throwable) {
        // Error handling
    }
};
Action0 onCompletedAction = new Action0() {
    // onCompleted()
    @Override
    public void call() {
        Log.d(tag, "completed");
    }
};

// 自动创建 Subscriber ，并使用 onNextAction 来定义 onNext()
observable.subscribe(onNextAction);
// 自动创建 Subscriber ，并使用 onNextAction 和 onErrorAction 来定义 onNext() 和 onError()
observable.subscribe(onNextAction, onErrorAction);
// 自动创建 Subscriber ，并使用 onNextAction、 onErrorAction 和 onCompletedAction 来定义 onNext()、 onError() 和 onCompleted()
observable.subscribe(onNextAction, onErrorAction, onCompletedAction);
```
解释：
`Action0` 是 `RxJava` 的一个接口，它只有一个方法 `call()`，这个方法是无参无返回值的；由于` onCompleted()` 方法也是无参无返回值的，因此 `Action0 `可以被当成一个包装对象，将 `onCompleted()` 的内容打包起来将自己作为一个参数传入` subscribe()` 以实现不完整定义的回调。这样其实也可以看做将 `onCompleted()` 方法作为参数传进了 `subscribe()`，相当于其他某些语言中的『闭包』。 `Action1 `也是一个接口，它同样只有一个方法 `call(T param)`，这个方法也无返回值，但有一个参数；与` Action0` 同理，由于 `onNext(T obj) `和 `onError(Throwable error)` 也是单参数无返回值的，因此 `Action1` 可以将 `onNext(obj)` 和 `onError(error)` 打包起来传入 `subscribe()` 以实现不完整定义的回调。事实上，虽然` Action0` 和 `Action1 `在 API 中使用最广泛，但` RxJava `是提供了多个 `ActionX `形式的接口 (例如 `Action2`, `Action3`) 的，它们可以被用以包装不同的无返回值的方法。

# 线程控制 
在不指定线程的情况下，` RxJava` 遵循的是线程不变的原则，即：在哪个线程调用 `subscribe()`，就在哪个线程生产事件；在哪个线程生产事件，就在哪个线程消费事件。如果需要切换线程，就需要用到` Scheduler`（调度器）。

`RxJava` 通过`Scheduler`来指定每一段代码应该运行在什么样的线程。`RxJava `已经内置了几个` Scheduler` ，它们已经适合大多数的使用场景：
> 
- Schedulers.immediate(): 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。
Schedulers.newThread(): 总是启用新线程，并在新线程执行操作。
- Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。
- Schedulers.computation(): 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。
- 另外， Android 还有一个专用的  AndroidSchedulers.mainThread()，它指定的操作将在 Android 主线程运行。

有了这几个 `Scheduler` ，就可以使用` subscribeOn() `和 `observeOn()` 两个方法来对线程进行控制了。 
> 
` subscribeOn()`: 指定` subscribe() `所发生的线程，即 `Observable.OnSubscribe` 被激活时所处的线程。或者叫做事件产生的线程。 
`observeOn()`: 指定 `Subscriber` 所运行在的线程。或者叫做事件消费的线程。

# 资料
[RxJava JavaDoc](http://reactivex.io/RxJava/javadoc/)  
[ReactiveX文档中文翻译](https://www.gitbook.com/book/mcxiaoke/rxdocs/details)  
[RxJava Essentials 中文翻译版](http://wiki.jikexueyuan.com/project/rxjava/)  
[给 Android 开发者的 RxJava 详解](https://gank.io/post/560e15be2dca930e00da1083)  
[RxJava系列专题（Android方向）](http://www.jianshu.com/collection/d79a6385bded)  
[RxJava系列](http://www.jianshu.com/p/ec9849f2e510)  
[Rxjava 从入门到开发](http://www.jianshu.com/p/9160e800dffd)  
[使用RxJava 提升用户体验](http://www.jianshu.com/p/33c548bce571)  
[Rxjava实战](http://www.jianshu.com/p/64aa976a46be)  