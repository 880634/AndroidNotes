---
title: RxJava操作符
date: 2016-10-22 19:50:30
categories: [RxJava]
tags: RxJava
---

# RxJava操作符

介绍下常用的操作符，并附上例子。全部操作符见[官方文档](https://mcxiaoke.gitbooks.io/rxdocs/content/Operators.html)

<!--more-->

# 创建操作符

创建`Observable`的各种方法。
> 
just( ) — 将一个或多个对象转换成发射这个或这些对象的一个Observable
from( ) — 将一个Iterable, 一个Future, 或者一个数组转换成一个Observable
repeat( ) — 创建一个重复发射指定数据或数据序列的Observable
repeatWhen( ) — 创建一个重复发射指定数据或数据序列的Observable，它依赖于另一个Observable发射的数据
create( ) — 使用一个函数从头创建一个Observable
defer( ) — 只有当订阅者订阅才创建Observable；为每个订阅创建一个新的Observable
range( ) — 创建一个发射指定范围的整数序列的Observable
interval( ) — 创建一个按照给定的时间间隔发射整数序列的Observable
timer( ) — 创建一个在给定的延时之后发射单个数据的Observable
empty( ) — 创建一个什么都不做直接通知完成的Observable
error( ) — 创建一个什么都不做直接通知错误的Observable
never( ) — 创建一个不发射任何数据的Observable

## Create
使用一个函数从头开始创建一个`Observable`,建议在传递给`create`方法的函数中检查观察者的`isUnsubscribed`状态，以便在没有观察者的时候，让你的`Observable`停止发射数据或者做昂贵的运算。

示例代码：
```java
Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> observer) {
        try {
            if (!observer.isUnsubscribed()) {
                for (int i = 1; i < 5; i++) {
                    observer.onNext(i);
                }
                observer.onCompleted();
            }
        } catch (Exception e) {
            observer.onError(e);
        }
    }
 } ).subscribe(new Subscriber<Integer>() {
        @Override
        public void onNext(Integer item) {
            System.out.println("Next: " + item);
        }

        @Override
        public void onError(Throwable error) {
            System.err.println("Error: " + error.getMessage());
        }

        @Override
        public void onCompleted() {
            System.out.println("Sequence complete.");
        }
    });
```
输出：
```
Next: 1
Next: 2
Next: 3
Next: 4
Sequence complete.
```
`create`不太常用，一般都用`from`或者`just`

## Interval

创建一个按固定时间间隔发射整数序列的`Observable`

![](http://oeiu2t0ur.bkt.clouddn.com/interval.c.png)
```java
// 每隔 1 s 发送一个序列号，序列号从 0 开始，每次累加 1。
Observable<Long> observable = Observable.interval(1, TimeUnit.SECONDS);
```

## timer( )
创建一个在指定延迟时间后发射一条数据（ 固定值：0 ）的 Observable 对象，可用来做定时操作。
![](http://oeiu2t0ur.bkt.clouddn.com/timer.png)
```java
// 定时 3 s
Observable<Long> observable = Observable.timer(3, TimeUnit.SECONDS);
```

## range( )
创建一个发射特定整数序列的`Observable`,接受两个参数，一个是范围的起始值，一个是范围的数据的数目。如果你将第二个参数设为0，将导致`Observable`不发射任何数据（如果设置为负数，会抛异常）
![](http://oeiu2t0ur.bkt.clouddn.com/range.png)
```java
// 依次发射 5、6、7
Observable<Integer> observable = Observable.range(5, 3);
```

## empty()
创建一个不发射任何数据就发出 onCompleted() 通知的 `Observable` 对象。
```java
// 发出一个 onCompleted() 通知
Observable<Object> observable = Observable.empty();
```

## error( )
创建不发射任何数据就发出 `onError` 通知的` Observable `对象。
```java
// 发出一个 onError() 通知
Observable<Object> observable = Observable.error(new Throwable("message"));
```
##  never()

创建一个不发射任何数据和通知的 `Observable `对象。
```java
Observable<Object> observable = Observable.never();
```

## defer( )
在订阅的时候才会创建 `Observable` 对象；每一次订阅都创建一个新的 `Observable` 对象。`Defer`操作符会一直等待直到有观察者订阅它，然后它使用`Observable`工厂方法生成一个`Observable`。它对每个观察者都这样做，因此尽管每个订阅者都以为自己订阅的是同一个`Observable`，事实上每个订阅者获取的是它们自己的单独的数据序列。
在某些情况下，等待直到最后一分钟（就是知道订阅发生时）才生成`Observable`可以确保`Observable`包含最新的数据。
![](http://oeiu2t0ur.bkt.clouddn.com/defer.png)
```java
Observable<String> observable = Observable.defer(new Func0<Observable<String>>() {
    @Override
    public Observable<String> call() {
        return Observable.just("string");
    }
});
```
## repeat( )

使`Observable `对象在发出` onNext()` 通知之后重复发射数据。重做结束才会发出 `onComplete()` 通知，若重做过程中出现异常则会中断并发出 `onError()` 通知。
![](http://oeiu2t0ur.bkt.clouddn.com/repeat.o.png)

```java
Observable<String> observable = Observable.just("string");
// 无限重复执行
observable.repeat();
// 重复执行 5 次
observable.repeat(5);
```

## repeatWhen( )
使`Observable` 对象在发出` onNext()` 通知之后有条件的重复发射数据。重做结束才会发出 `onCompleted()` 通知，若重做过程中出现异常则会中断并发出` onError() `通知。
将原始`Observable`的终止通知（完成或错误）当做一个`void`数据传递给一个通知处理器，它以此来决定是否要重新订阅和发射原来的`Observable`。这个通知处理器就像一个`Observable`操作符，接受一个发射`void`通知的`Observable`为输入，返回一个发射`void`数据（意思是，重新订阅和发射原始`Observable`）或者直接终止（意思是，使用`repeatWhen`终止发射数据）的`Observable`。
![](http://oeiu2t0ur.bkt.clouddn.com/repeatWhen.f.png)
```java
observable.repeatWhen(new Func1<Observable<? extends Void>, Observable<?>>() {
    @Override
    public Observable<?> call(Observable<? extends Void> observable) {
        // 重复 3 次, 每次间隔 1 s
        return observable.zipWith(Observable.range(1, 3), new Func2<Void, Integer, Integer>() {
            @Override
            public Integer call(Void aVoid, Integer integer) {
                return integer;
             }
        }).flatMap(integer -> Observable.timer(1, TimeUnit.SECONDS));
    }
});
```


# 错误处理操作符

## retry( )

在执行 `Observable`对象的序列出现异常时，不直接发出`onError()` 通知，而是重新订阅该 `Observable`对象，直到重做过程中未出现异常，则会发出 `onNext() `和 `onCompleted()` 通知；若重做过程中也出现异常，则会继续重试，直到达到重试次数上限，超出次数后发出最新的 `onError()` 通知。
![](http://oeiu2t0ur.bkt.clouddn.com/retry.C.png)
```java
Observable<Integer> observable = Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        System.out.println(".......");
        int a = 1 / 0;
        subscriber.onNext(a);
        subscriber.onCompleted();
    }
});
// 无限次的重试
observable.retry();
// 重试 3 次
observable.retry(3);
// 使用谓语函数决定是否重试
observable.retry(new Func2<Integer, Throwable, Boolean>() {
    @Override
    public Boolean call(Integer integer, Throwable throwable) {
        // 参数 integer 是订阅的次数; 参数 throwable 是抛出的异常
        // 返回值为 true 表示重试, 返回值为 false 表示不重试
        return false;
    }
});
```

## retryWhen( )
有条件的执行重试。`retryWhen`和`retry`类似，区别是，`retryWhen`将`onError`中的`Throwable`传递给一个函数，这个函数产生另一个`Observable`，`retryWhen`观察它的结果再决定是不是要重新订阅原始的`Observable`。如果这个`Observable`发射了一项数据，它就重新订阅，如果这个`Observable`发射的是`onError`通知，它就将这个通知传递给观察者然后终止。
![](http://oeiu2t0ur.bkt.clouddn.com/retryWhen.f.png)
```java
// 重试 3 次，每次间隔 1 s
observable.retryWhen(new Func1<Observable<? extends Throwable>, Observable<?>>() {
    @Override
    public Observable<?> call(Observable<? extends Throwable> observable) {
        return observable.zipWith(Observable.range(1, 3), new Func2<Throwable, Integer, Object>() {
            @Override
            public Object call(Throwable throwable, Integer integer) {
                return integer;
            }
        }).flatMap(new Func1<Object, Observable<?>>() {
            @Override
            public Observable<?> call(Object o) {
                return Observable.timer(1, TimeUnit.SECONDS);
            }
        });
    }
});
```













# 转换操作符

所有这些转换类操作符都作用于一个可观测序列，然后变换它发射的值，最后用一种新的形式返回它们。


## map
`map()`函数接受一个`Func1`类型的参数(就像这样`map(Func1<? super T, ? extends R> func`)),然后吧这个`Func1`应用到每一个由`Observable`发射的值上，将发射的只转换为我们期望的值。`Func1`构造函数中的两个参数分别是`Observable`发射值当前的类型和`map`转换后的类型，
原理图：
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-b9835e78f8b3c30d.png)
例如：
![](http://oeiu2t0ur.bkt.clouddn.com/2187866-d2962c3f1dfb56ab.jpg)
举个例子：接收一系列的数字，经过map转成string类型，然后打印出来。
```java
Observable.just(1, 2, 3, 4, 5)
    .map(new Func1<Integer, String>() {
        @Override
        public String call(Integer integer) {
            return "the position is" + integer;
        }
    }).subscribe(new Observer<String>() {
    @Override
    public void onCompleted() {

    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onNext(String s) {
        Log.i(tag, s);
    }
        });
```
## flatMap
原理图：
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-7728bea372bd612c.png)

假设有一个数据结构『学生』,每个学生有许多Course，现要求打印出每个学生的所有Course。
你可以这么写：
```java
Student[] students = ...;
Subscriber<Student> subscriber = new Subscriber<Student>() {
    @Override
    public void onNext(Student student) {
        List<Course> courses = student.getCourses();
        for (int i = 0; i < courses.size(); i++) {
            Course course = courses.get(i);
            Log.d(tag, course.getName());
        }
    }
    ...
};
Observable.from(students)
    .subscribe(subscriber);
```
如果我不想在 `Subscriber` 中使用 for 循环，而是希望` Subscriber` 中直接传入单个的` Course `对象呢（这对于代码复用很重要）。用 `map()` 显然是不行的，因为 `map()` 是一对一的转化，而我现在的要求是一对多的转化。这就要用到`flatMap`了。
```java
Student[] students = ...;
Subscriber<Course> subscriber = new Subscriber<Course>() {
    @Override
    public void onNext(Course course) {
        Log.d(tag, course.getName());
    }
    ...
};
Observable.from(students)
    .flatMap(new Func1<Student, Observable<Course>>() {
        @Override
        public Observable<Course> call(Student student) {
            return Observable.from(student.getCourses());
        }
    })
    .subscribe(subscriber);
```
`flatMap()`和`map()`都是把传入的参数转化之后返回另一个对象。但和`map()`不同的是，`flatMap()`中返回的是`Observable`对象，并且这个`Observable`对象并不是被直接发送到` Subscriber`的回调方法中。
`flatMap()` 的原理是这样的：
> 1. 使用传入的事件对象创建一个 Observable 对象；
2. 并不发送这个 Observable, 而是将它激活，于是它开始发送事件；
3. 每一个创建出来的 Observable 发送的事件，都被汇入同一个 Observable ，而这个 Observable 负责将这些事件统一交给 Subscriber 的回调方法。

这三个步骤，把事件拆成了两级，通过一组新创建的 `Observable` 将初始的对象『铺平』之后通过统一路径分发了下去。而这个『铺平』就是 `flatMap()` 所谓的` flat`。

## concatMap
`concatMap()`解决了`flatMap()`的交叉问题，它能够把发射的值连续在一起，就像这样：
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-164e546750bb2d25.png)
上面的例子中，比如Student1有A，B两门课程，Student2有C，D两门课程。
如果用`flatMap`，最后打印出来的可能是：`A C B D`。用`concatMap`可以保证先打印玩Student1的课程，再打印Student2的课程。从这两张图也可以看出区别。

## flatMapIterable
`flatMapIterable()`和`flatMap()`几乎是一样的，不同的是flatMapIterable()它转化的多个Observable是使用Iterable作为源数据的。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-795b0c77be948cb5.png)

```java
Observable.from(communities)
    .flatMapIterable(new Func1<Community, Iterable<House>>() {
        @Override
        public Iterable<House> call(Community community) {
            return community.houses;
        }
    })
    .subscribe(new Action1<House>() {

        @Override
        public void call(House house) {

        }
    });
```

## switchMap
`switchMap()`和`flatMap()`很像，除了一点：每当源Observable发射一个新的数据项（Observable）时，它将取消订阅并停止监视之前那个数据项产生的Observable，并开始监视当前发射的这一个。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-da947588728ad2c0.png)

例子：<http://blog.csdn.net/axuanqq/article/details/50738464>

## scan()
`scan()`对一个序列的数据应用一个函数，并将这个函数的结果发射出去作为下个数据应用合格函数时的第一个参数使用,有点类似于递归操作
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-0bcbf02e847bd42c.png)

```java
Observable.just(1, 2, 3, 4, 5)
        .scan(new Func2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer integer, Integer integer2) {
                return integer + integer2;
            }
        }).subscribe(new Action1<Integer>() {
    @Override
    public void call(Integer integer) {
        System.out.print(integer+“ ”);
    }
});
```
输出结果为：1 3 6 10 15

## groupBy
groupBy()将原始Observable发射的数据按照key来拆分成一些小的Observable，然后这些小Observable分别发射其所包含的的数据，和SQL中的groupBy类似。实际使用中，我们需要提供一个生成key的规则（也就是Func1中的call方法），所有key相同的数据会包含在同一个小的Observable中。另外我们还可以提供一个函数来对这些数据进行转化，有点类似于集成了flatMap。
例子：
```java
Observable.interval(1, TimeUnit.SECONDS).take(10).groupBy(new Func1<Long, Long>() {
            @Override
            public Long call(Long value) {
                //按照key为0,1,2分为3组
                return value % 3;
            }
        }).subscribe(new Action1<GroupedObservable<Long, Long>>() {
            @Override
            public void call(GroupedObservable<Long, Long> result) {
                result.subscribe(new Action1<Long>() {
                    @Override
                    public void call(Long value) {
                        System.out.println("key:" + result.getKey() +", value:" + value);
                    }
                });
            }
        });
```
运行结果如下： 
key:0, value:0 
key:1, value:1 
key:2, value:2 
key:0, value:3 
key:1, value:4 
key:2, value:5 
key:0, value:6 
key:1, value:7 
key:2, value:8 
key:0, value:9

![](http://oeiu2t0ur.bkt.clouddn.com/1167421-f583fa689f2d1f6f.png)


#过滤操作符。
顾名思义，这类`operators`主要用于对事件数据的筛选过滤，只返回满足我们条件的数据。过滤类操作符主要包含： `Filter Take TakeLast TakeUntil Skip SkipLast ElementAt Debounce Distinct DistinctUntilChanged First Last`等等。


## Filter
`filter(Func1)`用来过滤观测序列中我们不想要的值，只返回满足条件的值
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-b065454753c9f0a2.png)
```java
Observable.just("H1", "h2","h3","h4","h5").filter(new Func1<String, Boolean>() {
            @Override
            public Boolean call(String s) {
                return s.startsWith("H");
            }
        }).subscribe(new Subscriber<String>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                Log.i(tag, s);
            }
        });
```
通过设置filter，然后在call里面添加s.startsWith("H")，如果是H开头就返回true，否则返回false。从而能够过滤掉不是h开头的消息，打印出以H开头的消息。


## Take
`take(int)`用一个整数n作为一个参数，从原始的序列中发射前n个元素.

![](http://oeiu2t0ur.bkt.clouddn.com/1167421-22bfbabf028711d3.png)
```java
Observable.just("H1", "h2","h3","h4","h5").take(3)
.subscribe(new Subscriber<String>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                Log.i(tag, s);
            }
        });
```
通过设置`take(3)`,就可以取出前3个消息，打印出H1,h2,h3


如果想从后面取数据，可以调用`takeLast(3)`取最后3条消息

## TakeLast
`takeLast(int)`同样用一个整数n作为参数，只不过它发射的是观测序列中后n个元素。

![](http://oeiu2t0ur.bkt.clouddn.com/1167421-71ada93d756d3834.png)


## TakeUntil
### `takeUntil(Observable)`
订阅并开始发射原始`Observable`，同时监视我们提供的第二个`Observable`。如果第二个`Observable`发射了一项数据或者发射了一个终止通知，`takeUntil()`返回的`Observable`会停止发射原始`Observable`并终止。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-c068f5eaaf920cbd.png)

```java
Observable<Long> observableA = Observable.interval(300, TimeUnit.MILLISECONDS);
Observable<Long> observableB = Observable.interval(800, TimeUnit.MILLISECONDS);

observableA.takeUntil(observableB)
        .subscribe(new Subscriber<Long>() {
            @Override
            public void onCompleted() {
                System.exit(0);
            }
            @Override
            public void onError(Throwable e) {

            }
            @Override
            public void onNext(Long aLong) {
                System.out.println(aLong);
            }
        });

try {
    Thread.sleep(Integer.MAX_VALUE);
} catch (InterruptedException e) {
    e.printStackTrace();
}
```
程序输出：
0
1

### takeUntil(Func1)
通过`Func1`中的`call`方法来判断是否需要终止发射数据。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-ec085c374d9bc3d9.png)

```java
Observable.just(1, 2, 3, 4, 5, 6, 7)
    .takeUntil(new Func1<Integer, Boolean>() {
        @Override
        public Boolean call(Integer integer) {
            return integer >= 5;
        }
    }).subscribe(new Action1<Integer>() {
    @Override
    public void call(Integer integer) {
        System.out.println(integer);
    }
});
```
程序输出：
1
2
3
4
5

## Skip/skipLast
`skip(int)`让我们可以忽略`Observable`发射的前n项数据。
`skipLast(int`)忽略Observable发射的后n项数据

![](http://oeiu2t0ur.bkt.clouddn.com/1167421-8b343c90b2f71e95.png)
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-79e07ca5e1e35412.png)

```java
Observable.just("hello", "my", "world").skip(1).skipLast(1)
    .subscribe(new Subscriber<String>() {
        @Override
        public void onCompleted() {

        }

        @Override
        public void onError(Throwable e) {

        }

        @Override
        public void onNext(String s) {
            Log.i(tag, s);
        }
    });
```
//跳过第一个hello和最后一个world，打印my


## ElementAt/elementAtOrDefault
`elementAt(int)`用来获取元素`Observable`发射的事件序列中的第n项数据，并当做唯一的数据发射出去。如果我们想查找第五个元素但是可观测序列只有三个元素可供发射时该怎么办？我们可以使用`elementAtOrDefault()`。

![](http://oeiu2t0ur.bkt.clouddn.com/1167421-cf9886da48897135.png)
```java
Observable.just("hello", "my", "world").elementAt(1)
    .subscribe();
    
Observable.just("hello", "my", "world").elementAtOrDefault(10, "null")
    .subscribe();
```
## Debounce
###`debounce(long, TimeUnit)`
过滤掉了由`Observable`发射的速率过快的数据；如果在一个指定的时间间隔过去了仍旧没有发射一个，那么它将发射最后的那个。通常我们用来结合`RxBing`(Jake Wharton大神使用`RxJava`封装的Android UI组件)使用，防止button重复点击。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-79ce441d54e1c6d8.png)

### `debounce(Func1)`
可以根据`Func1`的`call`方法中的函数来过滤，`Func1`中的中的`call`方法返回了一个临时的`Observable`，如果原始的`Observable`在发射一个新的数据时，上一个数据根据`Func1`的`call`方法生成的临时`Observable`还没结束，那么上一个数据就会被过滤掉。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-6677d06993015a57.png)

## Distinct
### `distinct()`
`distinct()`的过滤规则是只允许还没有发射过的数据通过，所有重复的数据项都只会发射一次。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-2576e801f19fa185.png)

```java
Observable.just(2, 1, 2, 2, 3, 4, 3, 4, 5, 5)
        .distinct()
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer i) {
                System.out.print(i + " ");
            }
        });
```
程序输出：
2 1 3 4 5

### distinct(Func1)
`distinct(Func1)`参数中的`Func1`中的`call`方法会根据`Observable`发射的值生成一个`Key`，然后比较这个`key`来判断两个数据是不是相同；如果判定为重复则会和`distinct()`一样过滤掉重复的数据项。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-5944f17f2ef2cb51.png)

## DistinctUntilChanged
### distinctUntilChanged()
`distinctUntilChanged()`和`distinct()`类似，只不过它判定的是`Observable`发射的当前数据项和前一个数据项是否相同。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-132563847d82aa7f.png)

```java
Observable.just(2, 1, 2, 2, 3, 4, 3, 4, 5, 5)
        .distinctUntilChanged()
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer i) {
                System.out.print(i + " ");
            }
        });
```
程序输出：
2 1 2 3 4 3 4 5

### distinctUntilChanged(Func1)
`distinctUntilChanged(Func1)`和`distinct(Func1)`一样，根据Func1中call方法产生一个Key来判断两个相邻的数据项是否相同。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-fd439bd240905d28.png)

## First
### first()
`first()`顾名思义，它是的`Observable`只发送观测序列中的第一个数据项。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-d89b84c5bfdc689f.png)

### first(Func1)
`first(Func1)`只发送符合条件的第一个数据项。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-4a70f8ca7898536e.png)


## Last

### last()
`last()`只发射观测序列中的最后一个数据项。

![](http://oeiu2t0ur.bkt.clouddn.com/1167421-a52c8505a57939a1.png)

### last(Func1)
`last(Func1)`只发射观测序列中符合条件的最后一个数据项。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-cd505313d33d3760.png)



# 组合操作符
这类`operators`可以同时处理多个`Observable`来创建我们所需要的`Observable`。组合操作符主要包含：` Merge StartWith Concat Zip CombineLatest  SwitchOnNext Join`等等

## Merge
### merge(Observable, Observable)
`merge(Observable, Observable)`将两个`Observable`发射的事件序列组合并成一个事件序列，就像是一个`Observable`发射的一样。你可以简单的将它理解为两个`Obsrvable`合并成了一个`Observable`，合并后的数据是无序的。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-2f269a8811793bf9.png)
```java
String[] letters = new String[]{"A", "B", "C", "D", "E", "F", "G", "H"};
Observable<String> letterSequence = Observable.interval(300, TimeUnit.MILLISECONDS)
        .map(new Func1<Long, String>() {
            @Override
            public String call(Long position) {
                return letters[position.intValue()];
            }
        }).take(letters.length);

Observable<Long> numberSequence = Observable.interval(500, TimeUnit.MILLISECONDS).take(5);

Observable.merge(letterSequence, numberSequence)
        .subscribe(new Observer<Serializable>() {
            @Override
            public void onCompleted() {
                System.exit(0);
            }

            @Override
            public void onError(Throwable e) {
                System.out.println("Error:" + e.getMessage());
            }

            @Override
            public void onNext(Serializable serializable) {
                System.out.print(serializable.toString()+" ");
            }
        });

```
程序输出：
A 0 B C 1 D E 2 F 3 G H 4

### merge(Observable[])
`merge(Observable[])`将多个Observable发射的事件序列组合并成一个事件序列，就像是一个Observable发射的一样。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-b1790479187c2251.png)


### mergeDelayError()
每个Observable抛出的错误都将会打断合并。如果你需要避免这种情况，`RxJava`提供了`mergeDelayError()`，它能从一个`Observable`中继续发射数据即便是其中有一个抛出了错误。当所有的`Observables`都完成时，`mergeDelayError()`将会发射`onError()`，如下图所示：
![](http://oeiu2t0ur.bkt.clouddn.com/2187866-3f9c705089834ae1.jpg)

## StartWith
### startWith(T)
`startWith(T)`用于在源`Observable`发射的数据前插入数据。使用`startWith(Iterable<T>)`我们还可以在源`Observable`发射的数据前插入`Iterable`。官方示意图：
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-46bf223042f59214.png)

### startWith(Observable<T>)
`startWith(Observable<T>)`用于在源`Observable`发射的数据前插入另一个`Observable`发射的数据（这些数据会被插入到
源Observable发射数据的前面）。官方示意图：
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-3be11fbea609675a.png)

## Concat
`concat(Observable<? extends T>, Observable<? extends T>)` `concat(Observable<？ extends Observable<T>>)`用于将多个`obserbavle`发射的的数据进行合并发射，`concat`严格按照顺序发射数据，前一个`Observable`没发射玩是不会发射后一个`Observable`的数据的。它`和merge、startWitch`和相似，不同之处在于：
> 
merge:合并后发射的数据是无序的；
startWitch:只能在源Observable发射的数据前插入数据。

![](http://oeiu2t0ur.bkt.clouddn.com/1167421-cb53c8189015cb58.png)
```java
String[] letters = new String[]{"A", "B", "C", "D", "E", "F", "G", "H"};
Observable<String> letterSequence = Observable.interval(300, TimeUnit.MILLISECONDS)
        .map(new Func1<Long, String>() {
            @Override
            public String call(Long position) {
                return letters[position.intValue()];
            }
        }).take(letters.length);

Observable<Long> numberSequence = Observable.interval(500, TimeUnit.MILLISECONDS).take(5);

Observable.concat(letterSequence, numberSequence)
        .subscribe(new Observer<Serializable>() {
            @Override
            public void onCompleted() {
                System.exit(0);
            }

            @Override
            public void onError(Throwable e) {
                System.out.println("Error:" + e.getMessage());
            }

            @Override
            public void onNext(Serializable serializable) {
                System.out.print(serializable.toString() + " ");
            }
        });
```
程序输出：
A B C D E F G H 0 1 2 3 4

## Zip
`zip(Observable, Observable, Func2)`用来合并两个`Observable`发射的数据项，根据`Func2`函数生成一个新的值并发射出去。当其中一个`Observable`发送数据结束或者出现异常后，另一个`Observable`也将停在发射数据。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-dc5abf808a199553.png)

```java
String[] letters = new String[]{"A", "B", "C", "D", "E", "F", "G", "H"};
Observable<String> letterSequence = Observable.interval(120, TimeUnit.MILLISECONDS)
        .map(new Func1<Long, String>() {
            @Override
            public String call(Long position) {
                return letters[position.intValue()];
            }
        }).take(letters.length);

Observable<Long> numberSequence = Observable.interval(200, TimeUnit.MILLISECONDS).take(5);

Observable.zip(letterSequence, numberSequence, new Func2<String, Long, String>() {
    @Override
    public String call(String letter, Long number) {
        return letter + number;
    }
}).subscribe(new Observer<String>() {
    @Override
    public void onCompleted() {
        System.exit(0);
    }

    @Override
    public void onError(Throwable e) {
        System.out.println("Error:" + e.getMessage());
    }

    @Override
    public void onNext(String result) {
        System.out.print(result + " ");
    }
});
```
程序输出：
A0 B1 C2 D3 E4

## CombineLatest
`comnineLatest(Observable, Observable, Func2)`用于将两个`Observale`最近发射的数据已经`Func2`函数的规则进展组合。下面是官方提供的原理图：
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-f07b5e718eb48142.png)

![](http://oeiu2t0ur.bkt.clouddn.com/1167421-8a324e1c18a36c06.png)


## SwitchOnNext
`switchOnNext(Observable<? extends Observable<? extends T>>`用来将一个发射多个小Observable的源Observable转化为一个Observable，然后发射这多个小Observable所发射的数据。如果一个小的Observable正在发射数据的时候，源Observable又发射出一个新的小Observable，则前一个Observable发射的数据会被抛弃，直接发射新
的小Observable所发射的数据。

结合下面的原理图大家应该很容易理解，我们可以看到下图中的黄色圆圈就被丢弃了。
![](http://oeiu2t0ur.bkt.clouddn.com/1167421-c428b5f7352fb097.png)



## Join
`join(Observable, Func1, Func1, Func2)`我们先介绍下join操作符的4个参数：
> 
Observable：源Observable需要组合的Observable,这里我们姑且称之为目标Observable；
Func1：接收从源Observable发射来的数据，并返回一个Observable，这个Observable的声明周期决定了源Obsrvable发射出来的数据的有效期；
Func1：接收目标Observable发射来的数据，并返回一个Observable，这个Observable的声明周期决定了目标Obsrvable发射出来的数据的有效期；
Func2：接收从源Observable和目标Observable发射出来的数据，并将这两个数据组合后返回。
所以Join操作符的语法结构大致是这样的：onservableA.join(observableB, 控制observableA发射数据有效期的函数， 控制observableB发射数据有效期的函数，两个observable发射数据的合并规则)

![](http://oeiu2t0ur.bkt.clouddn.com/1167421-2f068c6a7cd3f4a6.png)


join操作符的效果类似于排列组合，把第一个数据源A作为基座窗口，他根据自己的节奏不断发射数据元素，第二个数据源B，每发射一个数据，我们都把它和第一个数据源A中已经发射的数据进行一对一匹配；举例来说，如果某一时刻B发射了一个数据“B”,此时A已经发射了0，1，2，3共四个数据，那么我们的合并操作就会把“B”依次与0,1,2,3配对，得到四组数据： [0, B] [1, B] [2, B] [3, B]




# 资料
[RxJava JavaDoc](http://reactivex.io/RxJava/javadoc/)
[ReactiveX文档中文翻译](https://www.gitbook.com/book/mcxiaoke/rxdocs/details)
[RxJava Essentials 中文翻译版](http://wiki.jikexueyuan.com/project/rxjava/)