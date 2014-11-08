#android中的多线程



*本文例子github下载地址：*[点我！点我！](https://github.com/k-lam/mutiTheadTest)

***本文从理解出发，不一定正确，仅作为总结。不对的地方求欢迎务必指正。***
##进程/线程同步简介：

1.  我们先看看一开始的程序设计，是顺序执行的，洗澡，煮饭，吃饭。但是你会发现，煮饭是电饭煲的事啊，于是我们就想到煮饭和洗澡同时进行，等到煮好饭，洗完澡就能吃饭。这样是不是更“省时”？

2. 另外一个例子：两个程序员抢妹子(妹子默认是互斥资源，不共享)，那一个程序员抢到手后，另外一个只能等他两分手。（race Condition）

上面第一个例子,我们就想到，能不能把两项任务同时进行？于是就出现了并发。并发虽然“省时”，但究竟是先煮好饭，还是先洗完澡呢？这是不确定的，所以这两项任务的执行，必须协作。

第二个例子，我们看到。两个程序员可能没什么关系的，只是抢同一个资源，于是出现了互斥。

概括一下:
*由于系统资源是有限的，所以n个进程同时请求只有m个的资源，当n > m时，就会出现竞争。而当两个进程或线程合作去完成一项工作，进度不一样时，就要出现等待条件满足的时候，就会出现同步的需要。*

解决同步互斥，通常包括以下方法：

* 信号量与PV操作
* 管程（Monitor）
* 临界区
* 互斥量

**经典问题：**

* **生产者消费者**
* **5位哲学家吃通心粉**
* **读写者**

其实上面这几个不止是“问题”，更加是解决方案、模式，如生产者消费者模式，就是产生数据与处理数据的处理不一致时，经常会用到。如渲染，渲染用GPU，用专门的线程去做（如android中的GLSurfaceView），ui线程接收用户操作，向GL线程发送操作指令，GL线程再去处理，这里产生数据和处理数据就很好地解耦了。java中经常用`BlockingQueue`类来实现生产者消费者模式。

而对于读写者，java中通常用读写锁来实现。

下面我们只从java来分析。因为android程序大多都是用java的，ndk中用pthread也有，但不多。

首先，不是所有线程都需要同步控制，那到底两个线程是否需要同步控制呢？
Bernstein提出

>假设：
>R(Pi) = {程序Pi在执行期间所引用的变量集}，
>W(Pi) = {程序Pi在执行期间所引用的变量集}
>
>如果R(P1)∩W(P2) ∪ R(P2)∩W(P1) = ∅,则并发执行的P1和P2可以保持封闭性和可再现性。

也就是说，不需要同步控制。

##Java中

###[JMM](http://en.wikipedia.org/wiki/Java_Memory_Model)
首先明确，

* *在多线程的环境下讨论JMM才有意义，因为单线程下，肯定按照代码顺序执行。*
* 线程是处理器调到的执行单位，（进程是资源分配单位）。也可以认为，线程运行时，对应一个处理器。

让我们看看底层的环境：

1. 在编译器中生成的指令顺序，可以与源代码中的顺序不同
2. 处理器可以采用乱序或并行等方式来执行指令（指令重排序，处理器架构的内容）
3. 执行时变量保存在寄存器中，而不是内存中，而处理器还有cache来缓存
4. 缓存的存在会改变写入变量提高到主内存的次序
5. 多处理器环境中，处理器A的缓存对处理器B是不可见的

上面几点，难懂，但是这里只是说java，所以就没必要搞那么深入，所以总结为以下两点

1. **代码编译成指令，为了达到最佳的执行性能，编译器，处理器等会把这些指令进行重排序（reorder），也就是指令执行和代码顺序可能会不同**
2. **处理器中有寄存器，有寄存器缓存。当一条线程被运行时，会把在主存上下文load到处理器，执行过程中，处理器中的数据，和主存的数据肯定不一样。**

对于第一点，还是例子说说吧

		//重排序例子：
		static int a,b;
	    new Thread(new Runnable() {
			
			@Override
			public void run() {
				a = 1;
				b = 2;
			}
		}).start();
        
       int c = a;
	   int d = b;

c 和 d的值是不确定的，可能是(0,0) (0,2) (1,0) (1,2)。因为重排序的存在

哇！f*ck,想想都可怕。不过细想一下，并行的两个线程，其实要协作竞争的，可能就那么一两个变量，所以只需要搞掂这一两个协作竞争的变量就好了。

**一种想法：**我们希望两个线程是按照规则的我们预想的顺序执行的，如果存在这种方法，我们去寻找这种方法就好了。

这时候，我们问问，java用来干嘛的？有什么特性？跨平台。yes，java就是为了屏蔽这些平台，系统差异的。让java程序员专注于业务性代码的开发。所以！

java提供了这种方法，来实现我们这种想法，用JMM来描述，具体就是定义了

1.  lock:将主内存中的变量锁定，为一个线程所独占
2.  unclock:将lock加的锁定解除，此时其它的线程可以有机会访问此变量
3.  read:将主内存中的变量值读到工作内存当中
4.  load:将read读取的值保存到工作内存中的变量副本中。
5.  use:将值传递给线程的代码执行引擎
6.  assign:将执行引擎处理返回的值重新赋值给变量副本
7.  store:将变量副本的值存储到主内存中。
8.  write:将store存储的值写入到主内存的共享变量当中。

8种原子操作，且必须成对出现。和Happens-Before偏序关系。又由于我们不是研究JMM或Java这门语言的
所以我们知道下面这个就可以了：

线程操作某个对象时，执行顺序如下：

1. 从主存复制变量到当前工作内存 (read and load)
2. 执行代码，改变共享变量值 (use and assign)
3. 用工作内存数据刷新主存相关内容 (store and write)


###Volatile
很多地方说，volatile变量是提供一种轻量的同步机制。这个轻量同步机制是什么意思呢？

首先，我们解析**可见性**这个概念
当单线程时，a = 1;线程自己肯定是见到a的值是1.
但是多线程就未必了，当线程A 执行a = 1，线程B未必见到a = 1；因为上面说过处理器本地内存问题。

所以为了修改马上可见。我们有一种轻量的同步机制，Volatile。记住以下两点

* 读取volatile的值，总会是最新的
* volatile不会阻塞线程

**注意：volatile并没有保证原子操作，也就是说，没有保证线程A修改volatile变量时，其他线程不可修改。**以下代码可以验证：

	static volatile int vi = 0;
	static AtomicInteger finishCount = new AtomicInteger(0);
	static final int N = 10;
	static final int preAddCount = 5000;
	static Object vLock = new Object();
	static boolean useSync = false;

	public static void main(String[] args) {
		testVi(30);
	}
	
	static void testVi(int count){
			...
			for(int i = 0; i != N;i++){
				new Thread(new Sum(count,startLatch)).start();
			}
			...
	}
	
	static class Sum implements Runnable{
		int id;
		CountDownLatch startLatch;
		Sum(int id,CountDownLatch startLatch){
			this.startLatch = startLatch;
			this.id = id;
		}
		@Override
		public void run() {
			...
			for(int i = 0;i != preAddCount;i++){
				if(useSync){
					synchronized (vLock) {
						vi++;
					}
				}else {
					vi++;
				}
			}
			...
		}
	};

多次测试可以看到vi的结果是不对的。这个例子在代码的`TestVolatile`类中

*所以volatile经常是用在：只有一个线程修改volatile变量，其他线程只是读volatile变量*

###synchronized关键字

可以见内置锁，或叫监视器模式，其实就是管程（monitor）。

这个关键字就是用来实现可见性和Happens-Before关系的。退出锁的时候，会发布状态（store and write）和 可以让其他线程获取锁。

同一时刻，只有一个线程能持有锁对象，直到这个线程释放这个锁对象

四种方式   synchronized关键字

   1. sychronized method(){}

   2. sychronized (objectReference) {/*block*/}

   3. static synchronized method(){}

   4. synchronized(classname.class)

可以看看`TestReadWriteLockWithSysnc` 和`TestReadWriteLockWithSysnc2`这两个类

####Object.wait Object.notify Object.notifyAll

wait()方法是，当获得了锁，发现现在条件还不满足下面的执行，就先**阻塞**当前线程，直到这个对象的notify/notifyAll被调用。

注意，notify是唤醒一个之前wait的。notifyAll是唤醒所有。

可以看看`TestReadWriteLockWithSysnc` 和`TestReadWriteLockWithSysnc2`这两个类
   
###显式锁
实现了java.util.concurrent.locks.Lock这个接口

synchronized关键字的好处是简单，但是synchronized是不能查询锁的相关信息，如锁的状态。导致synchronized对象如果被其他线程占有，当前线程又去请求锁的话，只能阻塞。这样很容易出现阻塞。

####[ReentrantLock](http://developer.android.com/reference/java/util/concurrent/locks/ReentrantLock.html)

记得一定要在finally里释放锁，通常是这样的

		try {
			if(lock.tryLock()){
				//或取锁成功
				do sth
			}else {
				//没能获取锁
				do sth
			} catch (InterruptedException e) {
				e.printStackTrace();
			}finally{
				if(lock.isHeldByCurrentThread()){
					lock.unlock();
				}
			}

在`TestReadWriteLockWithLock`中

####ReentrantReadWriteLock
读写锁。
很多情况下是，有专门的读线程和写线程。读读是不需要同步的，他们互不影响。但是写读和写写之间就会有冲突。读写锁就是为了这种情况的。

在代码中可以看到TestReadWriteLockWithLock2这个类就是用了读写锁的。


###同步容器类
如HashTable和Vector，但是注意一个问题，就是复合操作。如下面这个需求：
给Vector每个元素的值加1.的确 Vector.get是线程安全的。但get完后，对每个元素进行操作时，就不是了。可以这样做

	Vector<Integer> vector;
	synchronized(vector){
		for(...){
			add 1;
		}
	}

但是这样做也是有问题的，就是vector如果元素多的时候，锁的粒度就太大了。当多个线程竞争时，并发性就会大大降低，很多线程都处于等待获得锁。尽管不是复合操作，锁对象的粒度同样是整个容器类。问题同样存在。于是有了并发容器。

但是在android编程中需要考虑到，android中一般很少会出现复杂的并发情况。所以同步容器类基本够用了。


###并发容器类
####BlockingQueue
这个类有一组阻塞的put和take操作，当达到容量上限时，put会阻塞。单队列空时，take方法阻塞
而且`take`和`put`方法通过锁模式保证了原子性的操作。

最佳的例子就是生产者消费者模式，官网代码：

	 class Producer implements Runnable {
	   private final BlockingQueue queue;
	   Producer(BlockingQueue q) { queue = q; 
	   public void run() {
	     try {
	       while (true) { queue.put(produce()); }
	     } catch (InterruptedException ex) { ... handle ...}
	   }
	   Object produce() { ... }
	 }
	
	 class Consumer implements Runnable {
	   private final BlockingQueue queue;
	   Consumer(BlockingQueue q) { queue = q; }
	   public void run() {
	     try {
	       while (true) { consume(queue.take()); }
	     } catch (InterruptedException ex) { ... handle ...}
	   }
	   void consume(Object x) { ... }
	 }
	
	 class Setup {
	   void main() {
	     BlockingQueue q = new SomeQueueImplementation();
	     Producer p = new Producer(q);
	     Consumer c1 = new Consumer(q);
	     Consumer c2 = new Consumer(q);
	     new Thread(p).start();
	     new Thread(c1).start();
	     new Thread(c2).start();
	   }
	 }}

在下文，你会看到基于BlockingQueue实现的基于消息的线程执行机制。

* ConcurrentHashMap 使用分段锁技术，大大提高了并发性
* CopyOnWriteArrayList

###一些工具类
####CountDownLatch
这个类就像一个闸门一样，await方法阻塞线程，用countDown()进行减操作，直到count = 0。阻塞的地方就可以继续执行了。

这里有一个有趣的跑马例子

####FutureTask

####Semaphore 
信号量 可以基于数量来进行阻塞，如果阻塞是和数量有关的，用信号量最好。

可以参考`TestReadWriteLockWithSemaphore`这个类
####Barrier 栅栏 
to be continued

####中断线程
to be continued...

##Android的多线程

多线程在Android编程中为什么那么重要？

1. android的UI渲染。

	1. android没有专门的渲染线程
	2. android接收事件的线程和主线程中
	3. 要保证UI的流畅，必须保证UI线程不能执行太多的计算
	
2. 多核处理器。如果一直用单线程的话，是无法提升app的速度。也就是说，手机怎么多核变快，也与你的app无关。你的app依然运行很慢

所以在android中多线程尤为重要，而android中的多线程主要是把UI线程的任务交给其他线程，把计算分给各个线程。而不是像服务器那种处理多个请求的阻塞式多线程通讯的多线程。

###Handler,Looper,MessageQueue,Message
很多人都分析过这几个东西，都挺详细的。但是把多的，大的东西说得简单，才是能真正明白。

首先一般的程序，入口时main(),但是这些程序，执行完就完了，不能好像GUI程序那样，点击一下会响应。那怎么办？

1. **放在一个`while(true){}`里**

	但是你总不能让程序不断的`while(true)`轮训有没有事件吧，所以

2. **一个阻塞机制**

	那怎样唤醒这个阻塞呢？触摸屏幕怎样通知我们有触摸事件发生了呢？

3. **消息机制**
	
以下就是简单的代码实现片段：

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
					Runnable runnable = mRunnableQueue.take();//mRunnableQueue空会阻塞，直至有新的Runnable放进队列
					runnable.run();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}
	
*完整代码在`MessageBaseThread`类中*

在android中，UI线程是一个基于消息机制的线程。就是有消息来的时候，就执行，没消息来的时候，就阻塞。android屏幕的Touch，其他事件的发生，都是通过HAL通知window，window的ViewRoot通过消息机制通知UI线程的，其实就是发给MessageQueue，当Looper处理完上一个操作，就去看看MessageQueue有没有消息，有就处理。Handler就是用来发送Message甚至发送处理这个Message的程序（Runnable）的类。

下面是这些类的代码详解：

####Handler

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

关键就是`queue.next()`和`msg.target.dispatchMessage(msg);`
`queue.next()`通过监视器(synchronized关键字)提供原子性的操作和线程的阻塞。

`msg.target.dispatchMessage(msg);`这句，msg.target是一个Handler

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




###AsyncTask
这个类时怎样做到在doInBackground这个非UI线程，向onProgressUpdate和onPostExecute传入参数的呢？其实是用handler.sendMessage这些方法实现的（具体是Message.sendToTarget）


##Bolts
这是第三方开源库，Parse and Facebook是用这个的。

用起来的确比AsyncTask好用，AsyncTask的代码用起来很容易变成一坨的东西，维护起来很不好用。
而且new的时候就要指定所有泛型，而Bolts则在需要用到时，才指明泛型类型。

不过Bolts上手有一点难度，但一旦上手，还是挺好用的。之前的Bolts的cancel是没有真正cancel到的，最新版要靠各位看看了。

github地址：[go->](https://github.com/BoltsFramework/Bolts-Android)