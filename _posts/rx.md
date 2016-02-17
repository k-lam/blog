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
	
###阶段总结1：
Observable就是定义事件，和这个事件发生时（Subscribe），通知谁（Subscriber）。

奇怪，如果是传统的观察者模式，Observable（事件event），然后Observer执行

	Observable.trigger -> Observer.call

当Observable.subscribe的时候，就直接调用Observer.call

但是Rx明显多了一步啊！Observable，OnSubscribe，Observer

	Observable.subscirbe -> OnSubscribe.call -> Observer.onNext,[onError|onComplete]

	
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
 
 1. 把observable串联起来，内部的一个执行流程，或者说顺序是怎样的
 2. 为什么能串起来，是通过什么方法去除掉`callback hell`的？其实不单单`Rx`，facebook的`Bolts`一样能做到去除`callback hell`
 
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