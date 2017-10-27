###Observable
这货其实就是，可被观察的，可被订阅的，当他被订阅的时候，预先给定的onSubscribe就会被执行

###OnSubscribe

	public interface OnSubscribe<T> extends Action1<Subscriber<? super T>> {
	    // cover for generics insanity
	}
	public interface Action1<T> extends Action {
		void call(T t);
	}

结合上文的Observable，就是当订阅发生时，OnSubscribe.call会被执行，而且参数必须是`Subscriber`
​	
###阶段总结1：
Observable就是定义事件，和这个事件发生时（Subscribe），通知谁（Subscriber）。

奇怪，如果是传统的观察者模式，Observable（事件event），然后Observer执行

	Observable.trigger -> Observer.call

当Observable.subscribe的时候，就直接调用Observer.call

但是Rx明显多了一步啊！Observable，OnSubscribe，Observer

	Observable.subscirbe -> OnSubscribe.call -> Observer.onNext,[onError|onComplete]

​	
###Operator

	interface Operator<R, T> extends Func1<Subscriber<? super R>, Subscriber<? super T>>

就是把Subscriber\<T\>转换成Subscriber\<R\>的

 

	// 注意：这不是 subscribe() 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
	// 如果需要看源码，可以去 RxJava 的 GitHub 仓库下载。
	public Subscription subscribe(Subscriber subscriber) {
	    subscriber.onStart();
	    onSubscribe.call(subscriber);
	    return subscriber;
	}

另外注意

	class Obsrvable<T>{
		final OnSubscribe<T> onSubscribe;//其实就是一个callback，在subscribe时候触发，见上面的subscribe()
		public final Subscription subscribe(Subscriber<? super T> subscriber) {...}
		...
	}


 ###lift理解

 目的在于两点：

1.  把observable串联起来，内部的一个执行流程，或者说顺序是怎样的
2.  为什么能串起来，是通过什么方法去除掉`callback hell`的？其实不单单`Rx`，facebook的`Bolts`一样能做到去除`callback hell`

      // 注意：这不是 lift() 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
     // 如果需要看源码，可以去 RxJava 的 GitHub 仓库下载。
     public <R> Observable<R> lift(Operator<? extends R, ? super T> operator) {
         return Observable.create(new OnSubscribe<R>() {
             @Override
             public void call(Subscriber subscriber) {
                 Subscriber newSubscriber = operator.call(subscriber);
                 newSubscriber.onStart();
                 onSubscribe.call(newSubscriber);
             }
         });
     }

 假设如下转换：


 	Observable.OnSubscribe onSubscribe1;
 	Obversable observable1 = Obversavle.create(onSubscribe1);
 	Obversable observable2 = observable1.lift(operater);
 	Subsriber subsriber2;
 	obserable2.subscribe(subscriber2);
 	
我们再做一个设定，observable1与subsriber1对应，observable2与subsriber2对应
。现在展开上面的代码，先展开的是 	
 	
`obserable2.subscribe(subscriber2);`这一句，我们来展开代码

	// 注意，subscribe触发的是observable2.onSubscribe.call方法

	subscriber2.onStart();
	// 下面是对obserable2.onSubscribe.call(subscriber2);的展开，而这里subscriber2是在observable1.lift中定义的，
	// 所以注意，下面的onSubscribe是observable1的！！！！
	// Subscriber newSubscriber = operator.call(subscriber2);实际上 newSubscriber 是 subscriber1
	Subscriber subscriber1 = operator.call(subscriber2);// 前文说过了，operator就是把subscriber转为另外一个subscriber.
	subscriber1.onStart();
	
	onSubscribe1.call(subscriber1);


	return subscriber2;

解释一下为什么newSubscriber 是 subscriber1，
因为newSubscriber是operator转换出来的结果
而operator是用户希望做的转换。也就是把observable1转为observable2
​	
去掉注释

	subscriber2.onStart();
	Subscriber subscriber1 = operator.call(subscriber2);
	
	subscriber1.onStart();
	onSubscribe1.call(subscriber1);
	
	return subscriber2; 	

很奇怪，没有看到onSubscribe2.call，这是因为，onSubscribe2.call隐含在onSubscribe1.call(subscriber1)这一句中！
也就是operator要做的事情，我observable1转化为observable2，当observable1被触发，是怎样转换，而且负责调用observable2！

也就是说，链式调用的真正是在

	Subscriber subscriber1 = operator.call(subscriber2);
	onSubscribe1.call(subscriber1);

这两句，而不同的转换，通过继承operator，来实现，这个实现还有一项任务，就是把转换结果传递到下一个Subscriber，也就是对原Subscriber进行调用！

在进一步注意到，lift的具体使用实例，如map，传入参数是`Fun1<T,R>`对象，也就是说，其实要做的，只是把`Obserable<T>`的T换成`Obserable<R>`的R！。所以在上面那句

	onSubscribe1.call(subscriber1);

实质上也只需要

	R r = fun<T,R>.call(t);
	subscriber1.onNext(r);





## Observable弹出数据

Rx的魅力在于操作符，而把原有的Observables通过operator转换后再subscribe，怎么执行到之后的onNext？需要自己实现的！可以参考RxEmitor（KL 写的）和BehaviorSubject。




# ReactiveX is a combination of the best ideas fromthe Observer pattern, the Iterator pattern, and functional programming

所以， RX就两方面的东西，观察者模式（实际上是响应模式）和函数式编程



# 函数式编程

参考：

[函数式编程初探 作者： 阮一峰](http://www.ruanyifeng.com/blog/2012/04/functional_programming.html)

函数式编程强调没有"副作用"，意味着函数要保持独立，所有功能就是返回一个新的值，没有其他行为，尤其是不得修改外部变量的值。



rx的函数式主要表现在operator的申明式



函数式编程不擅长于处理可变状态和处理IO [知乎](https://www.zhihu.com/question/28292740)



# [响应式 Reactor pattern](https://en.wikipedia.org/wiki/Reactor_pattern)

- **Resources:** Any resource that can provide input to or consume output from the system.
- **Synchronous Event Demultiplexer:** Uses an [event loop](https://en.wikipedia.org/wiki/Event_loop) to block on all resources. When it is possible to start a synchronous operation on a resource without blocking, the demultiplexer sends the resource to the dispatcher.
- **Dispatcher:** Handles registering and unregistering of request handlers. Dispatches resources from the demultiplexer to the associated request handler.
- **Request Handler:** An application defined request handler and its associated resource.



# Composition via Observable Operators

The real power comes with the “reactive extensions” (hence “ReactiveX”) — operators that allow you to transform, combine, manipulate, and work with the sequences of items emitted by Observables.



# Rx和Bolts或者Promise模式的区别

[link](https://zhuanlan.zhihu.com/p/20531896)



这个问题的发现是因为我做摩登启动的时候，发现Rx无法像Bolts那样continue...



我们在哪些场景下用Rx比较方便？首先是需要源源不断的流出数据的场景，因为Promise是一次性的，不适合做这类工作。

比如说把事件/定时器抽象成Rx的Observable更合适，事件可以响应很多次，定时器也可以响应很多次，我们还可以利用Rx的debounce运算符来进行节流，在频繁触发事件的时候过滤那些重复的。

其次是可能需要重试的场景，由于Rx有retry或者repeat这种从源头开始的运算符，我们可以用它来执行比如“出错后重试三次”之类动作，而Promise就需要你递归处理了，破坏了then的链式。



由于Rx不是基于这种then的链式调用，所以适合有外部状态的，如IO。而promise更适合类似于函数式编程的，没有外部状态。



Rx响应式，如果不是多次响应，也就没有意义了。

所以我们简单区分：

需要多次响应  响应式

一次链式调用的， promise 或函数式



## Rx的坑

Subscriber.unsubscribe 后  subscribe其他Observable  是无效的。也就是一旦unsubscribe  就废了。不过可以切换subscribe其他，也就是说  一个Subscriber只能接收一个Observable



​	BehaviorSubject<Integer> subject = BehaviorSubject.create();

        BehaviorSubject<Integer> subject1 = BehaviorSubject.create();
        Subscriber<Integer> sb = new Subscriber<Integer>() {
            @Override
            public void onCompleted() {
    
            }
    
            @Override
            public void onError(Throwable e) {
    
            }
    
            @Override
            public void onNext(Integer integer) {
                Log.i(TAG, "sb onNext: " + integer);
            }
        };
        subject.subscribe(sb);
        subject.onNext(1);
        //sb.unsubscribe();
        subject.onNext(2);
        subject1.onNext(-1);
        subject1.subscribe(sb);
        subject.onNext(3);
        subject1.onNext(-2);
结果是