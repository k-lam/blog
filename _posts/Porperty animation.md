###ValueAnimator

>This class provides a simple timing engine for running animations which calculate animated values and set them on target objects.

There is a single timing pulse that all animations use. It runs in a custom handler to ensure that property changes happen on the UI thread.

By default, ValueAnimator uses non-linear time interpolation, via the AccelerateDecelerateInterpolator class, which accelerates into and decelerates out of an animation. This behavior can be changed by calling setInterpolator(TimeInterpolator).

上面有两个重要概念

1. timing engine。 It runs in a custom handler
2. interpolation

同时注意到
###TimeAnimator

>This class provides a simple callback mechanism to listeners that is synchronized with all other animators in the system. There is no duration, interpolation, or object value-setting with this Animator. Instead, it is simply started, after which it proceeds to send out events on every animation frame to its TimeListener (if set), with information about this animator, the total elapsed time, and the elapsed time since the previous animation frame.

The ValueAnimator encapsulates a TimeInterpolator, which defines animation interpolation, and a TypeEvaluator, which defines how to calculate values for the property being animated.


A time interpolator defines the rate of change of an animation. This allows animations to have non-linear motion, such as acceleration and deceleration.


Interface for use with the setEvaluator(TypeEvaluator) function. Evaluators allow developers to create animations on arbitrary property types, by allowing them to supply custom evaluators for types that are not automatically understood and used by the animation system.

##[ValueAnimator](http://developer.android.com/reference/android/animation/ValueAnimator.html)

####getAnimatedFraction

这个方法得出的结果是经过Interpretation运算的

	public float getAnimatedFraction() {
        return mCurrentFraction;
    }

	void animateValue(float fraction) {
        fraction = mInterpolator.getInterpolation(fraction);
        mCurrentFraction = fraction;
	    ...
    }

####getAnimatedValue()

####ofInt(int... i)

源码能看到，这些i是用来设定keyFrameSet的



	ValueAnimator animation = ValueAnimator.ofFloat(0f, 1f);
	animation.setDuration(1000);
	animation.start();

The previous code snippets, however, has no real effect on an object, because the ValueAnimator does not operate on objects or properties directly. The most likely thing that you want to do is modify the objects that you want to animate with these calculated values. You do this by defining listeners in the ValueAnimator to appropriately handle important events during the animation's lifespan, such as frame updates. When implementing the listeners, you can obtain the calculated value for that specific frame refresh by calling getAnimatedValue(). For more information on listeners, see the section about Animation Listeners.



问题：callback的时间？如果决定的？基于这个callback，应该如何使用基于时间变化的函数(映射)


ValueAnimator中

addUpdateListener

	 mUpdateListeners.add(listener);

调用mUpdateListeners的两个可能的地方：

	public ValueAnimator clone();
	void animateValue(float fraction) {
        fraction = mInterpolator.getInterpolation(fraction);
        mCurrentFraction = fraction;
        int numValues = mValues.length;
        for (int i = 0; i < numValues; ++i) {
            mValues[i].calculateValue(fraction);
        }
        if (mUpdateListeners != null) {
            int numListeners = mUpdateListeners.size();
            for (int i = 0; i < numListeners; ++i) {
                mUpdateListeners.get(i).onAnimationUpdate(this);
            }
        }
    }

animateValue就是执行这些update的callback的
*子类的TimeAnimator方法是空实现*

call animateValue的方法有：
	
	public void end();

	 /**
     * This internal function processes a single animation frame for a given animation. The
     * currentTime parameter is the timing pulse sent by the handler, used to calculate the
     * elapsed duration, and therefore
     * the elapsed fraction, of the animation. The return value indicates whether the animation
     * should be ended (which happens when the elapsed time of the animation exceeds the
     * animation's duration, including the repeatCount).
     *
     * @param currentTime The current time, as tracked by the static timing handler
     * @return true if the animation's duration, including any repetitions due to
     * <code>repeatCount</code> has been exceeded and the animation should be ended.
     */
	boolean animationFrame(long currentTime)

注意`animationFrame`的参数The current time, as tracked by the static timing handler. currentTime parameter is the timing pulse sent by the handler, used to calculate the   elapsed duration, and therefore  the elapsed fraction, of the animation.

在这个方法中，currentTime只是用来计算fraction的

	float fraction = mDuration > 0 ? (float)(currentTime - mStartTime) / mDuration : 1f;

在哪里调用这个方法呢？提示是一个static timing handler

	// The static sAnimationHandler processes the internal timing loop on which all animations
    // are based
    /**
     * @hide
     */
	protected static ThreadLocal<AnimationHandler> sAnimationHandler =
            new ThreadLocal<AnimationHandler>();

	
调用animationFrame的是doAnimationFrame。调用doAnimationFrame的：

	/**
     * Sets the position of the animation to the specified point in time. This time should
     * be between 0 and the total duration of the animation, including any repetition. If
     * the animation has not yet been started, then it will not advance forward after it is
     * set to this time; it will simply set the time to this value and perform any appropriate
     * actions based on that time. If the animation is already running, then setCurrentPlayTime()
     * will set the current playing time to this value and continue playing from that point.
     *
     * @param playTime The time, in milliseconds, to which the animation is advanced or rewound.
     */
    public void setCurrentPlayTime(long playTime);

	AnimationHandler.doAnimationFrame(long frameTime)

AnimationHandlerimplements Runnable，就在run方法中调用doAnimationFrame

	// Called by the Choreographer.
        @Override
        public void run() {
            mAnimationScheduled = false;
            doAnimationFrame(mChoreographer.getFrameTime());
        }

先看看AnimationHandler是一个静态类，不是内部类，所以先梳理下，AnimationHandler和ValueAnimator之间的数据沟通。

ValueAnimator中通过在startAnimation方法中`handler.mAnimations.add(this);`把自己传入AnimationHandler。

现在研究AnimationHandler的run是怎样被调用的注意到start方法

	   /**
         * Start animating on the next frame.
         */
        public void start() {
            scheduleAnimation();
        }

		private void scheduleAnimation() {
            if (!mAnimationScheduled) {
                mChoreographer.postCallback(Choreographer.CALLBACK_ANIMATION, this, null);
                mAnimationScheduled = true;
            }
        }

`AnimationHandler`就是通过
`mChoreographer.postCallback(Choreographer.CALLBACK_ANIMATION, this, null);`这句把AnimationHandler放入`Choreographer`的。嗯，一切都指向`mChoreographer`这个对象，而且关注`(mChoreographer.getFrameTime()`这个方法。

AnimationHandler中获得`Choreographer`对象：

		private final Choreographer mChoreographer;
        private AnimationHandler() {
            mChoreographer = Choreographer.getInstance();
        }

	    /**
	     * Gets the choreographer for the calling thread.  Must be called from
	     * a thread that already has a {@link android.os.Looper} associated with it.
	     *
	     * @return The choreographer for this thread.
	     * @throws IllegalStateException if the thread does not have a looper.
	     */
	    public static Choreographer getInstance() {
	        return sThreadInstance.get();//sThreadInstance是ThreadLocal类型
	    }

###Choreographer
(中文译意：编舞者！)
>Coordinates the timing of animations, input and drawing.

>The choreographer receives timing pulses (such as vertical synchronization) from the display subsystem then schedules work to occur as part of rendering the next display frame.
>
基于显示子系统去安排工作？

通过上面这句话看出，timing pulses应该是靠底层产生的。应该是无法保证间隔多少时间会产生一个timing pulse

>Applications typically interact with the choreographer indirectly using higher level abstractions in the animation framework or the view hierarchy. Here are some examples of things you can do using the higher-level APIs.

>To post an animation to be processed on a regular time basis synchronized with display frame rendering, use start().
To post a Runnable to be invoked once at the beginning of the next display frame, use postOnAnimation(Runnable).
To post a Runnable to be invoked once at the beginning of the next display frame after a delay, use postOnAnimationDelayed(Runnable, long).
To post a call to invalidate() to occur once at the beginning of the next display frame, use postInvalidateOnAnimation() or postInvalidateOnAnimation(int, int, int, int).
To ensure that the contents of a View scroll smoothly and are drawn in sync with display frame rendering, do nothing. This already happens automatically. onDraw(Canvas) will be called at the appropriate time.
However, there are a few cases where you might want to use the functions of the choreographer directly in your application. Here are some examples.

>If your application does its rendering in a different thread, possibly using GL, or does not use the animation framework or view hierarchy at all and you want to ensure that it is appropriately synchronized with the display, then use postFrameCallback(Choreographer.FrameCallback).
... and that's about it.

上面这段的关键是 synchronized with the display

Each Looper thread has its own choreographer. Other threads can post callbacks to run on the choreographer but they will run on the Looper to which the choreographer belongs.


这个类800+行啊（4.0的源码），怎么研究？从两个地方入手，1. 文档。能了解这个类大概是做什么的，主要实现功能是什么？按这个主要功能这条线去研究。2. 入口点。就是看有哪些开放接口。从入口点研究。

下面，我们先从入口研究。从

	private void scheduleAnimation() {
            if (!mAnimationScheduled) {
                mChoreographer.postCallback(Choreographer.CALLBACK_ANIMATION, this, null);
                mAnimationScheduled = true;
            }
        }
的postCallback。->postCallbackDelayed->postCallbackDelayedInternal

	    private void postCallbackDelayedInternal(int callbackType,
            Object action, Object token, long delayMillis) {
       	...
        synchronized (mLock) {
            final long now = SystemClock.uptimeMillis();
            final long dueTime = now + delayMillis;
            mCallbackQueues[callbackType].addCallbackLocked(dueTime, action, token);

            if (dueTime <= now) {
                scheduleFrameLocked(now);
            } else {
                Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);
                msg.arg1 = callbackType;
                msg.setAsynchronous(true);
                mHandler.sendMessageAtTime(msg, dueTime);
            }
        }
    }

关键是这句`mCallbackQueues[callbackType].addCallbackLocked(dueTime, action, token);`这个callbackType是`Choreographer.CALLBACK_ANIMATION`。是常量1

所以以下我们看看这个mCallbackQueues[1]在哪里被调用就好了。

我们在doCallbacks找到调用

	    void doCallbacks(int callbackType, long frameTimeNanos) {
        CallbackRecord callbacks;
	       ...
			callbacks = mCallbackQueues[callbackType].extractDueCallbacksLocked(now);
            for (CallbackRecord c = callbacks; c != null; c = c.next) {
                if (DEBUG) {
                    Log.d(TAG, "RunCallback: type=" + callbackType
                            + ", action=" + c.action + ", token=" + c.token
                            + ", latencyMillis=" + (SystemClock.uptimeMillis() - c.dueTime));
                }
                c.run(frameTimeNanos);
            }
       ...
    }

只有在doFrame中调用doCallbacks。doFrame又有三个地方调用到

1. FrameHandler.handleMessage
2. FrameDisplayEventReceiver.run
3. CallbackRecord.run



回到Choreographer的postCallbackDelayedInternal这个方法
因为dueTime = now，所以进入scheduleFrameLocked

	    private void scheduleFrameLocked(long now) {
        if (!mFrameScheduled) {
            mFrameScheduled = true;
            if (USE_VSYNC) {
                if (DEBUG) {
                    Log.d(TAG, "Scheduling next frame on vsync.");
                }

                // If running on the Looper thread, then schedule the vsync immediately,
                // otherwise post a message to schedule the vsync from the UI thread
                // as soon as possible.
                if (isRunningOnLooperThreadLocked()) {
                    scheduleVsyncLocked();
                } else {
                    Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_VSYNC);
                    msg.setAsynchronous(true);
                    mHandler.sendMessageAtFrontOfQueue(msg);
                }
            } else {
                final long nextFrameTime = Math.max(
                        mLastFrameTimeNanos / NANOS_PER_MS + sFrameDelay, now);
                if (DEBUG) {
                    Log.d(TAG, "Scheduling next frame in " + (nextFrameTime - now) + " ms.");
                }
                Message msg = mHandler.obtainMessage(MSG_DO_FRAME);
                msg.setAsynchronous(true);
                mHandler.sendMessageAtTime(msg, nextFrameTime);
            }
        }
    }

一开始mFrameScheduled是false。假设isRunningOnLooperThreadLocked() retur true。执行scheduleVsyncLocked

	private void scheduleVsyncLocked() {
        mDisplayEventReceiver.scheduleVsync();
    }
	/**
     * Schedules a single vertical sync pulse to be delivered when the next
     * display frame begins.
     */
    public void scheduleVsync() {
        if (mReceiverPtr == 0) {
            Log.w(TAG, "Attempted to schedule a vertical sync pulse but the display event "
                    + "receiver has already been disposed.");
        } else {
            nativeScheduleVsync(mReceiverPtr);
        }
    }

nativeScheduleVsync在NativeDisplayEventReceiver.cpp中

	static void nativeScheduleVsync(JNIEnv* env, jclass clazz, jint receiverPtr) {
	    sp<NativeDisplayEventReceiver> receiver =
	            reinterpret_cast<NativeDisplayEventReceiver*>(receiverPtr);
	    status_t status = receiver->scheduleVsync();
	    if (status) {
	        String8 message;
	        message.appendFormat("Failed to schedule next vertical sync pulse.  status=%d", status);
	        jniThrowRuntimeException(env, message.string());
	    }
	}

reinterpret_cast是C++里的强制类型转换符，但是怎么可以把一个int转成sp(智能指针)？

看mReceiverPtr是怎样来的。

在DisplayEventReceivers的构造函数中有`mReceiverPtr = nativeInit(this, mMessageQueue);`这一句


	static jint nativeInit(JNIEnv* env, jclass clazz, jobject receiverObj,
        jobject messageQueueObj) {
    sp<MessageQueue> messageQueue = android_os_MessageQueue_getMessageQueue(env, messageQueueObj);
    if (messageQueue == NULL) {
        jniThrowRuntimeException(env, "MessageQueue is not initialized.");
        return 0;
    }

    sp<NativeDisplayEventReceiver> receiver = new NativeDisplayEventReceiver(env,
            receiverObj, messageQueue);
    status_t status = receiver->initialize();
    if (status) {
        String8 message;
        message.appendFormat("Failed to initialize display event receiver.  status=%d", status);
        jniThrowRuntimeException(env, message.string());
        return 0;
    }

    receiver->incStrong(gDisplayEventReceiverClassInfo.clazz); // retain a reference for the object
    return reinterpret_cast<jint>(receiver.get());
	}

	NativeDisplayEventReceiver::NativeDisplayEventReceiver(JNIEnv* env,
        jobject receiverObj, const sp<MessageQueue>& messageQueue) :
        mReceiverObjGlobal(env->NewGlobalRef(receiverObj)),
        mMessageQueue(messageQueue), mWaitingForVsync(false) {
    ALOGV("receiver %p ~ Initializing input event receiver.", this);
	}

然后回到nativeScheduleVsync这方法的`status_t status = receiver->scheduleVsync();`这句

	status_t NativeDisplayEventReceiver::scheduleVsync() {
	    if (!mWaitingForVsync) {
	        ALOGV("receiver %p ~ Scheduling vsync.", this);
	
	        // Drain all pending events.
	        nsecs_t vsyncTimestamp;
	        int32_t vsyncDisplayId;
	        uint32_t vsyncCount;
	        processPendingEvents(&vsyncTimestamp, &vsyncDisplayId, &vsyncCount);
	
	        status_t status = mReceiver.requestNextVsync();
	        if (status) {
	            ALOGW("Failed to request next vsync, status=%d", status);
	            return status;
	        }
	
	        mWaitingForVsync = true;
	    }
	    return OK;
		}





###垂直同步

垂直同步又称场同步（Vertical Hold），从CRT显示器的显示原理来看，单个象素组成了水平扫描线，水平扫描线在垂直方向的堆积形成了完整的画面。显示器的刷新率受显卡DAC控制，显卡DAC完成一帧的扫描后就会产生一个垂直同步信号。我们平时所说的打开垂直同步指的是将该信号送入显卡3D图形处理部分，从而让显卡在生成3D图形时受垂直同步信号的制约。


