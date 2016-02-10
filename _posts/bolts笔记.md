[GitHub](https://github.com/BoltsFramework/Bolts-Android)

[API文档](http://boltsframework.github.io/docs/android/)

###Task

A task is not tied to a particular threading model: it represents the work being done, not where it is executing.

	/**
	 * Represents the result of an asynchronous operation.
	 * 
	 * @param <TResult>
	 *          The type of the result of the task.
	 */
	public class Task<TResult>{
		...
	}

* call

`static
<TResult> Task<TResult>
call(Callable<TResult> callable)` 

 Invokes the callable on the current thread, producing a Task.

`static
<TResult> Task<TResult>
callInBackground(Callable<TResult> callable) `

Invokes the callable on a background thread using the default thread pool, returning a Task to represent the operation.

Task没有构造函数，靠这两个函数执行一个Task。

Tresult是Callable的call函数返回结果，然后这个返回结果可以传给continueWith

* continueWith

`<TContinuationResult> 
Task<TContinuationResult>
continueWith(Continuation<TResult,TContinuationResult> continuation) `

Adds a synchronous continuation to this task, returning a new task that completes after the continuation has finished running.

先看Continuation

	/**
	 * A function to be called after a task completes.
	 * @see Task
	 */
	public interface Continuation<TTaskResult, TContinuationResult> {
	  //then中的task，是上一个task
	  TContinuationResult then(Task<TTaskResult> task) throws Exception;
	}

TTaskResult：就是上一个Task（就是taskA.continueWith(...)的taskA）执行完的返回结果，在then中会以参数的形式传入

TContinuationResult，就是这个Contiuation.then的返回类型

* continueWithTask

`<TContinuationResult> 
Task<TContinuationResult>
continueWithTask(Continuation<TResult,Task<TContinuationResult>> continuation)`

Adds an asynchronous continuation to this task, returning a new task that completes after the task returned by the continuation has completed.

TResult,同`continueWith` 的`TTaskResult`一样，看上文

TContinuationResult，因为这个方法是异步的，所以执行会用Task封装起来，而TContinuationResult就是这个封装起来的Task的，也就是这个Task的返回，你这个异步操作想要的返回。

taskA.continueWith还是taskA.continueWithTask,返回的Task都不是taskA.

forResult这些方法都是这样


### task.continueWithTask(ca).continueWith(cb).continueWith(cc,Task.BACKGROUND_EXECUTOR).continueWith(cd,UI_THREAD_EXECUTOR)。其中ca,cb,cc,cd都是Continuation,这些都是串行的，只是执行的线程不一样，这就是所谓的Chaining Tasks。如果要并行的话：

task.continueWith(cb,Task.BACKGROUND_EXECUTOR);

do sth;

这样do sth和cb就是并行的。


对于continuteWith和continuteWithTask的例子，见自己写的代码，testBolts