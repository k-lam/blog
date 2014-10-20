##进程/线程同步简介：
由于系统资源是有限的，所以n个进程同时请求只有m个的资源，当n > m时，就会出现竞争。而当两个进程或线程合作去完成一项工作，进度不一样时，就要出现等待条件满足的时候，就会出现同步的需要。这就是进程线程同步的概念。

解决同步互斥，通常包括以下方法：

* 信号量与PV操作
* 管程（Monitor）

##Handler

>A Handler allows you to send and process Message and Runnable objects associated with a thread's MessageQueue. Each Handler instance is associated with a single thread and that thread's message queue. When you create a new Handler, it is bound to the thread / message queue of the thread that is creating it -- from that point on, it will deliver messages and runnables to that message queue and execute them as they come out of the message queue.

看官网的这段文字，得到几点信息：

1. Handler是用来发送Message或Runnable的
2. 发给MessageQueue，这个MessageQueue是属于Thread的
3. 当你new Handler时，就已经把这个Handler和Thread的MessageQueue绑定了 
4. 发送给message queue的Message或Runnable，当他们出队时，会被执行。

奇怪的是，MessageQueue是属于Thread的，这个怎么理解？因为第三点
我们来看看Handler的构造函数

	程序1：
	 public Handler() {
        ...
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        ...
    }


Looper.myLooper()：

	程序2：
	static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
	public static Looper myLooper() {
        return sThreadLocal.get();
    }
	
ThreadLocal这个类是关键，看看

>Implements a thread-local storage, that is, a variable for which each thread has its own value. All threads share the same ThreadLocal object, but each sees a different value when accessing it, and changes made by one thread do not affect the other threads. The implementation supports null values.

简单说就是，一个对象了，每一个线程有自己的副本，每一个线程可以有自己的值，通过get() set()来获取值。

这里还没有完，因为第三点说的是Handler和MessageQueue绑定啊，这里只出现了Looper，同时注意到程序1.的以下语句

	mLooper = Looper.myLooper();
	if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }

也就是说，在调用`mLooper = Looper.myLooper();`之前，必须先调用`Looper.prepare()`

	 public static void prepare() {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper());
    }

	private Looper() {
        mQueue = new MessageQueue();
        mRun = true;
        mThread = Thread.currentThread();
    }

原来MessageQueue在Looper里面，而且注意到Looper.prepare()是静态的，直接把Looper的实例存到线程相关的变量里面，这样我们就不用自己保存一个引用了，怪不得`mLooper = Looper.myLooper();`能直接获取一个线程相关的实例。好了，2和3解决了，Handler里面有一个ThreadLocal的Looper，Looper中有MessageQueue.


对于第1和4点，我们先看看Android官网Handler的

>Scheduling messages is accomplished with the post(Runnable), postAtTime(Runnable, long), postDelayed(Runnable, long), sendEmptyMessage(int), sendMessage(Message), sendMessageAtTime(Message, long), and sendMessageDelayed(Message, long) methods. The post versions allow you to enqueue Runnable objects to be called by the message queue when they are received; the sendMessage versions allow you to enqueue a Message object containing a bundle of data that will be processed by the Handler's handleMessage(Message) method (requiring that you implement a subclass of Handler).

我们看看sendMessageAtTime这个方法，因为最后sendMessage族的方法，最后都是调用这个

	 public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        boolean sent = false;
        MessageQueue queue = mQueue;
        if (queue != null) {
            msg.target = this;
            sent = queue.enqueueMessage(msg, uptimeMillis);
        }
        else {
            RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
        }
        return sent;
    }

就是把msg放到Looper的MessageQueue中，第1点解决，现在看第4点，怎样执行呢？

看看官网给出的一段关于Handler的程序

	 class LooperThread extends Thread {
      public Handler mHandler;

      public void run() {
          Looper.prepare();

          mHandler = new Handler() {
              public void handleMessage(Message msg) {
                  // process incoming messages here
              }
          };

          Looper.loop();
      }
  	}

注意到最后一句`Looper.loop();`

	/**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
    public static void loop() {
        Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        MessageQueue queue = me.mQueue;   
        ...
        while (true) {
            Message msg = queue.next(); // might block
            if (msg != null) {
                 ...
                msg.target.dispatchMessage(msg);
 			    ...
                msg.recycle();
            }
        }
    }

关键就是 `msg.target.dispatchMessage(msg);`这句了，msg.target是一个Handler

	/**
     * Handle system messages here.
     */
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }

	 private final void handleCallback(Message message) {
        message.callback.run();
    }

	//mCallback就是构造函数Handler(Callback callback)传入来的，没有传就没有了
	public interface Callback {
        public boolean handleMessage(Message msg);
    }


最后才调用handleMessage(msg);就是一般我们会覆盖的方法，如：
	
	Handler handler = new Handler(){
			@Override
			public void handleMessage(Message msg) {
				int type = msg.what;
				switch (type) {
				case 0:
					//do sth
					break;
				case 1:
					//do sth
					break;
				default:
					break;
				}
			}
		};		

至此，Handler的sendMsg和怎么执行已经分析完毕

##HandlerThread
根据官方文档，说这个是一个
>Handy class for starting a new thread that has a looper. The looper can then be used to create handler classes.

怎样方便呢？

	private static class myHandlerThread extends HandlerThread{
		private Handler myHandler;
		public myHandlerThread(String name) {
			super(name);
		}
		
		public Handler getHandler(){
			return myHandler;
		}

		@Override
		public synchronized void start() {
			super.start();
			Looper looper = getLooper();//这一句会阻塞到Looper对象初始化结束，就可以确保strat后调用getHandler不会返回null
			myHandler = new Handler(looper){
				@Override
				public void handleMessage(Message msg) {
					//处理消息
				}
			};
		}
	}

就是注释的那一句


上面讲了好多，其实总结原理是很简单的。首先，我们要明白，Handler，Looper，Message，MessageQueue这个机制是为主线程（UI线程建立的），为什么？因为UI线程是一个基于消息机制的线程。就是有消息来的时候，就执行，没消息来的时候，就阻塞。android屏幕的Touch，其他事件的发生，都是通过HAL通知window，window的ViewRoot通过消息机制通知UI线程的。那么基于消息的线程怎样设计？我们不结合android源码，就最一般的java代码简单实现一下

	public class MessageBaseThread extends Thread {
		
		BlockingQueue<Runnable> mRunnableQueue = new ArrayBlockingQueue<Runnable>(10);
		
		public void sendRunnable(Runnable runnable){
			try {
				mRunnableQueue.put(runnable);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	
		@Override
		public void run() {
			while(true){
				try {
					Runnable runnable = mRunnableQueue.take();//mRunnableQueue空会//阻塞，知道有新的Runnable放进队列
					runnable.run();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}
	
够简单吧，就是一个while(true)，和一个会阻塞的BlockingQueue。
那就能理解Looper就是这么一个while(true)机制，MessageQueue就是上面的能阻塞的BlockingQueue。而Message，就是封装了message和Runnable的一个类，Handler就是用来发送Message和处理Message到UI线程的处理类


###AsyncTask
这个类时怎样做到在doInBackground这个非UI线程，向onProgressUpdate和onPostExecute传入参数的呢？其实是用handler.sendMessage这些方法实现的（具体是Message.sendToTarget）