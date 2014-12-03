#####View不能在非UI线程进行操作
官网有如下的解释
he Andoid UI toolkit is not thread-safe. So, you must not manipulate your UI from a worker thread—you must do all manipulation to 

your user interface from the UI thread. Thus, there are simply two rules to Android's single thread model:

Do not block the UI thread
Do not access the Android UI toolkit from outside the UI thread

一开始觉得，用同步控制就能解决线程安全的问题，但是同步会带来等待，或者乱序的问题。所以为了简化设计，还是规定UI只在UI线程操作好。由

于UI操作不会是耗时操作，所以放UI线程不会出问题。
也是由于这个原因，android设计了handler looper机制


![会抛出CalledFromWrongThreadException的11个函数](http://d.pcs.baidu.com/thumbnail/83f52d2c6d92056566896e9c69f50bf5?fid=2315310014-250528-700586452369221&time=1409540400&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-R33vBoqUQkN5%2BaTv4Ah8yOXfFGQ%3D&rt=sh&expires=2h&r=629582827&sharesign=unknown&size=c710_u500&quality=100)




#####Transformation

	    static class TransformationInfo {
        /**
         * The transform matrix for the View. This transform is calculated internally
         * based on the rotation, scaleX, and scaleY properties. The identity matrix
         * is used by default. Do *not* use this variable directly; instead call
         * getMatrix(), which will automatically recalculate the matrix if necessary
         * to get the correct matrix based on the latest rotation and scale properties.
         */
        private final Matrix mMatrix = new Matrix();

        /**
         * The transform matrix for the View. This transform is calculated internally
         * based on the rotation, scaleX, and scaleY properties. The identity matrix
         * is used by default. Do *not* use this variable directly; instead call
         * getInverseMatrix(), which will automatically recalculate the matrix if necessary
         * to get the correct matrix based on the latest rotation and scale properties.
         */
        private Matrix mInverseMatrix;

        /**
         * An internal variable that tracks whether we need to recalculate the
         * transform matrix, based on whether the rotation or scaleX/Y properties
         * have changed since the matrix was last calculated.
         */
        boolean mMatrixDirty = false;

        /**
         * An internal variable that tracks whether we need to recalculate the
         * transform matrix, based on whether the rotation or scaleX/Y properties
         * have changed since the matrix was last calculated.
         */
        private boolean mInverseMatrixDirty = true;

        /**
         * A variable that tracks whether we need to recalculate the
         * transform matrix, based on whether the rotation or scaleX/Y properties
         * have changed since the matrix was last calculated. This variable
         * is only valid after a call to updateMatrix() or to a function that
         * calls it such as getMatrix(), hasIdentityMatrix() and getInverseMatrix().
         */
        private boolean mMatrixIsIdentity = true;

        /**
         * The Camera object is used to compute a 3D matrix when rotationX or rotationY are set.
         */
        private Camera mCamera = null;

        /**
         * This matrix is used when computing the matrix for 3D rotations.
         */
        private Matrix matrix3D = null;

        /**
         * These prev values are used to recalculate a centered pivot point when necessary. The
         * pivot point is only used in matrix operations (when rotation, scale, or translation are
         * set), so thes values are only used then as well.
         */
        private int mPrevWidth = -1;
        private int mPrevHeight = -1;

        /**
         * The degrees rotation around the vertical axis through the pivot point.
         */
        @ViewDebug.ExportedProperty
        float mRotationY = 0f;

        /**
         * The degrees rotation around the horizontal axis through the pivot point.
         */
        @ViewDebug.ExportedProperty
        float mRotationX = 0f;

        /**
         * The degrees rotation around the pivot point.
         */
        @ViewDebug.ExportedProperty
        float mRotation = 0f;

        /**
         * The amount of translation of the object away from its left property (post-layout).
         */
        @ViewDebug.ExportedProperty
        float mTranslationX = 0f;

        /**
         * The amount of translation of the object away from its top property (post-layout).
         */
        @ViewDebug.ExportedProperty
        float mTranslationY = 0f;

        /**
         * The amount of scale in the x direction around the pivot point. A
         * value of 1 means no scaling is applied.
         */
        @ViewDebug.ExportedProperty
        float mScaleX = 1f;

        /**
         * The amount of scale in the y direction around the pivot point. A
         * value of 1 means no scaling is applied.
         */
        @ViewDebug.ExportedProperty
        float mScaleY = 1f;

        /**
         * The x location of the point around which the view is rotated and scaled.
         */
        @ViewDebug.ExportedProperty
        float mPivotX = 0f;

        /**
         * The y location of the point around which the view is rotated and scaled.
         */
        @ViewDebug.ExportedProperty
        float mPivotY = 0f;

        /**
         * The opacity of the View. This is a value from 0 to 1, where 0 means
         * completely transparent and 1 means completely opaque.
         */
        @ViewDebug.ExportedProperty
        float mAlpha = 1f;
    }

上面那段是View.TransformationInfo,在view的setScale,setRotation,setTranslate,setAlpha这些方法中，其实都是调用了view中的mTransformationInfo

	/**
     * Sets the horizontal location of this view relative to its {@link #getLeft() left} position.
     * This effectively positions the object post-layout, in addition to wherever the object's
     * layout placed it.
     *
     * @param translationX The horizontal position of this view relative to its left position,
     * in pixels.
     *
     * @attr ref android.R.styleable#View_translationX
     */
    public void setTranslationX(float translationX) {
        ensureTransformationInfo();
        final TransformationInfo info = mTransformationInfo;
        if (info.mTranslationX != translationX) {
            // Double-invalidation is necessary to capture view's old and new areas
            invalidateViewProperty(true, false);
            info.mTranslationX = translationX;
            info.mMatrixDirty = true;
            invalidateViewProperty(false, true);
            if (mDisplayList != null) {
                mDisplayList.setTranslationX(translationX);
            }
            if ((mPrivateFlags2 & PFLAG2_VIEW_QUICK_REJECTED) == PFLAG2_VIEW_QUICK_REJECTED) {
                // View was rejected last time it was drawn by its parent; this may have changed
                invalidateParentIfNeeded();
            }
        }
    }

	 /**
     * Sets the amount that the view is scaled in x around the pivot point, as a proportion of
     * the view's unscaled width. A value of 1 means that no scaling is applied.
     *
     * @param scaleX The scaling factor.
     * @see #getPivotX()
     * @see #getPivotY()
     *
     * @attr ref android.R.styleable#View_scaleX
     */
    public void setScaleX(float scaleX) {
        ensureTransformationInfo();
        final TransformationInfo info = mTransformationInfo;
        if (info.mScaleX != scaleX) {
            invalidateViewProperty(true, false);
            info.mScaleX = scaleX;
            info.mMatrixDirty = true;
            invalidateViewProperty(false, true);
            if (mDisplayList != null) {
                mDisplayList.setScaleX(scaleX);
            }
            if ((mPrivateFlags2 & PFLAG2_VIEW_QUICK_REJECTED) == PFLAG2_VIEW_QUICK_REJECTED) {
                // View was rejected last time it was drawn by its parent; this may have changed
                invalidateParentIfNeeded();
            }
        }
    }

Android 3.0 also introduce the DisplayList mechanism, DisplayList can be considered as a 2D drawing commands buffer, every View 

has its own DisplayList , and when its onDraw method called, all drawing commands issue through the input Canvas actually store 

into its own DisplayList. When every DisplayList are ready, Android will draw all the DisplayLists, it actually turn the 2D 

drawing commands into GLES commands to use GPU to render the window surface. So the rendering of View Hierarchy actually separated 

into two steps, generate View's DisplayList, and then draw the DisplayLists, and the second one use GPU mostly.


对象对应关系：

一个SharedClient对应一个Android应用程序，而一个Android应用程序可能包含有多个窗口，即Surface。
在SurfaceFlinger服务中，每一个SharedBufferStack都对应一个Surface，即一个窗口

 SharedBufferStack是在Android应用程序和SurfaceFlinger服务之间共享的

 为了方便SharedBufferStack在Android应用程序和SurfaceFlinger服务中的访问，Android系统分别使用SharedBufferClient和SharedBufferServer来描述SharedBufferStack，其中，SharedBufferClient用来在Android应用程序这一侧访问SharedBufferStack的空闲缓冲区列表，而SharedBufferServer用来在SurfaceFlinger服务这一侧访问SharedBufferStack的排队缓冲区列表。

SharedBufferStack中的缓冲区只是用来描述UI元数据的，这意味着它们不包含真正的UI数据。真正的UI数据保存在GraphicBuffer中

SharedBufferServer类是用来在SurfaceFlinger服务这一侧描述一个UI元数据缓冲区堆栈的，即在SurfaceFlinger服务中，每一个绘图表面，即一个Layer对象，都关联有一个UI元数据缓冲区堆栈。

Applacation通过Binder机制和SurfaceFlinger服务跨进程调用，所以在SurfaceFlinger是Layer，在Application的代理是SurfaceLayer

有了Surface之后，Android应用程序就可以在上面绘制自己的UI了，接着再请求SurfaceFlinger服务将这个已经绘制好了UI的Surface渲染到设备显示屏上去

Android应用程序请求SurfaceFlinger服务渲染自己的UI可以分为三步曲：首先是创建一个到SurfaceFlinger服务的连接，接着再通过这个连接来创建一个Surface，最后请求SurfaceFlinger服务渲染该Surface。


 Android应用程序渲染一个Surface的过程大致如下所示：

       1. 从UI元数据缓冲区堆栈中得到一个空闲的UI元数据缓冲区；

       2. 请求SurfaceFlinger服务为这个空闲的UI元数据缓冲区分配一个图形缓冲区（GraphicBuffer）；

       3. 在图形缓冲区上面绘制好UI之后，即填充好UI数据之后，就将前面得到的空闲UI元数据缓冲区添加到UI元数据缓冲区堆栈中的待渲染队列中去（SharedBufferStack）；

       4. 请求SurfaceFlinger服务渲染前面已经准备好了图形缓冲区的Surface；

       5. SurfaceFlinger服务从即将要渲染的Surface的UI元数据缓冲区堆栈的待渲染队列中找到待渲染的UI元数据缓冲区；

       6. SurfaceFlinger服务得到了待渲染的UI元数据缓冲区之后，接着再找到在前面第2步为它所分配的图形缓冲区，最后就可以将这个图形缓冲区渲染到设备显示屏上去。


一共涉及到了三种类型的线程，它们分别是Binder线程、UI渲染线程和控制台事件监控线程
SurfaceFlinger服务虽然是由System进程的主线程来负责启动的，但是最终它会运行在一个独立的线程中。我们将这个独立的线程称为UI渲染线程，因为它负责渲染系统的UI。

在这三种类型的线程中，UI渲染线程是主角，Binder线程和控制台事件监控线程是配角。Binder线程池是为了让其它进程，例如Android应用程序进程，可以与SurfaceFlinger服务进行Binder进程间通信的，有一部分通信所执行的操作便是让UI渲染线程更新系统的UI。控制台事件监控线程是为了监控硬件帧缓冲区的睡眠/唤醒状态切换事件的。一旦硬件帧缓冲区要进入睡眠或者唤醒状态，控制台事件监控线程都需要通知UI渲染线程，以便UI渲染线程可以执行关闭或者启动显示屏的操作。

SurfaceFlinger服务在启动的时候，还会创建另外一个线程来监控由内核发出的帧缓冲区硬件事件。为了方便描述，我们将这个线程称为控制台事件监控线程。每当帧缓冲区要进入睡眠状态时，内核就会发出一个睡眠事件，这时候SurfaceFlinger服务就会执行一个释放屏幕的操作；而当帧缓冲区从睡眠状态唤醒时，内核就会发出一个唤醒事件，这时候SurfaceFlinger服务就会执行一个获取屏幕的操作。

SurfaceFlinger服务的UI渲染线程是围绕它的消息队列来运行的

  我们知道，应用程序是运行在与SurfaceFlinger服务不同的进程中的。每当应用程序需要更新自己的UI时，它们就会通过Binder进程间通信机制来通知SurfaceFlinger服务。SurfaceFlinger服务接到这个通知之后，就会调用SurfaceFlinger类的成员函数signalEvent来唤醒UI渲染线程，以便它可以执行渲染UI的操作


 Activity组件的UI实现需要与WindowManagerService服务和SurfaceFlinger服务进行交互


事实上，用来关联Activity组件和Layer对象的SurfaceLayer对象并不是由Android应用程序请求SurfaceFlinger服务来创建的，而是由WindowManagerService服务请求SurfaceFlinger服务来创建的。WindowManagerService服务得到这个SurfaceLayer对象之后，再将它的一个代理对象返回给在Android应用程序这一侧的Activity组件。这样，Activity组件和WindowManagerService服务就可以通过同一个SurfaceLayer对象来操作在SurfaceFlinger服务这一侧的Layer对象，而操作Layer对象的目的就是为了修改Activity组件的UI。

![](http://d.pcs.baidu.com/thumbnail/30c4e74ca572e22d2627aacb2328bb1c?fid=2315310014-250528-946560732755779&time=1409562000&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-%2BesY4HZ4QkIy9r4McEvfnAnNP%2Fo%3D&rt=sh&expires=2h&r=636908540&sharesign=unknown&size=c850_u580&quality=100)


简单来说，ViewRoot相当于是MVC模型中的Controller，它有以下职责：

        1. 负责为应用程序窗口视图创建Surface。

        2. 配合WindowManagerService来管理系统的应用程序窗口。

        3. 负责管理、布局和渲染应用程序窗口视图的UI。


事实上，Activity类的成员变量mWindow指向的并不是一个Window对象，而是一个PhoneWindow对象。也就是说，一个Activity组件的UI是使用一个PhoneWindow对象来描述的。

![](http://d.pcs.baidu.com/thumbnail/f04dde05ea6e56c6f8d59f239a28ed6d?fid=2315310014-250528-1011255357436051&time=1409626800&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-0jnPqNF1hQQE3%2BpcC5FlBbC41CE%3D&rt=sh&expires=2h&r=822929939&sharesign=unknown&size=c850_u580&quality=100)
 PhoneWindow类有两个重要的成员变量mDecor和mContentParent，它们的类型分别DecorView和ViewGroup。其中，成员变量mDecor是用描述自己的窗口视图，而成员变量mContentParent用来描述视图内容的父窗口。

DecorView类继承了FrameLayout类，而FrameLayout类又继承了ViewGroup类，最后ViewGroup类又继承了View类。View类有一个成员函数draw，它是用来绘制应用程序窗口的UI的。DecorView类、FrameLayout类和ViewGroup类都重写了父类的成员函数draw，这样，它们就都可以定制自己的UI。

DecorView类所描述的应用程序窗口视图是否需要重新绘制是由另外一个类ViewRoot来控制的。系统在启动一个Activity组件的过程中，会为这个Activity组件创建一个ViewRoot对象，同时还会将前面为这个Activity组件所创建的一个PhoneWindow对象的成员变量mDecor所描述的一个视图（DecorView）保存在这个ViewRoot对象的成员变量mView中。这样，这个ViewRoot对象就可以通过调用它的成员变量mView的所描述的一个DecorView的成员函数draw来绘制一个Acitivity组件的UI了。ViewRoot类的作用是非常大的，它除了用来控制一个Acitivity组件的UI绘制之外，还负责接收Acitivity组件的IO输入事件，例如，键盘事件

当ViewRoot类需要重新绘制与它所关联的一个Activity组件的UI时，它就会将这个绘制UI的操作封装成一个消息，并且发送到应用程序进程的主线程的消息队列中去进一步处理，这样同样可以保证绘制UI的操作可以在应用程序进程的主线程中执行。（那为什么还要规定只能在主线程操作view，这个ViewRoot不是已经保障了么？）


   每一个ViewRoot对象都有一个类型为Surface的成员变量mSurface，它指向了一个Java层的Surface对象。这个Java层的Surface对象通过它的成员变量mNativeSurface与一个C++层的Surface对象

![](http://d.pcs.baidu.com/thumbnail/5ccb934e8cd134a8b2ab2a3126da9e05?fid=2315310014-250528-240966916370457&time=1409626800&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-dpgGFi6d5pc3DgXnci4axyyiN18%3D&rt=sh&expires=2h&r=993954401&sharesign=unknown&size=c850_u580&quality=100)

但是，与ViewRoot类的成员变量mSurface所对应的在C++层的Surface对象并没有一个对应的SurfaceControl对象，这是因为ViewRoot类并不需要设置应用程序窗口的属性，它需要做的只是往应用程序窗口的图形缓冲区填充UI数据，即它需要设置的只是应用程序窗口的纹理。应用程序窗口的纹理保存在Java层的Surface类的成员变量mCanvas所描述一个画布（Canvas）中，即通过这个画布可以访问到应用程序窗口的图形缓冲区。**当ViewRoot类需要重新绘制与它对应的Activity组件的UI时，它就会调用它的成员函数draw来执行这个绘制的操作**。ViewRoot类的成员函数draw首先通过获得保存它的成员变量mSurface内部的一块画布，然后再将这个画布传递给它的成员变量mView所描述的一个View对象的成员函数draw。View类的成员函数draw得到了这块画布之后，就可以随心所欲地上面绘制应用程序窗口的纹理了。这些纹理的绘制工作是通过Skia图形库API来进行的。


对于使用Java来开发的Android应用程序来说，它们一般是使用Skia图形库提供的API来绘制UI的。在Skia图库中，所有的UI都是绘制在画布（Canvas）上的，因此，Android应用程序窗口需要将它的图形缓冲区封装在一块画布里面，然后才可以使用Skia库提供的API来绘制UI。

![](http://d.pcs.baidu.com/thumbnail/d291a9885f918e73166f842f5aa93a48?fid=2315310014-250528-507363248147724&time=1409637600&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-V8eH%2BVlscGE4PSMw2rCojVGgyp4%3D&rt=sh&expires=2h&r=749748142&sharesign=unknown&size=c850_u580&quality=100)

这个顶层视图最终是由ViewRoot类的成员函数performTraversals来启动测量、布局和绘制操作的，这三个操作分别由DecorView类的成员函数measure和layout以及ViewRoot类的成员函数draw来实现的。

	public class View implements Drawable.Callback, KeyEvent.Callback, AccessibilityEventSource {  
    ......  
  
    protected ViewParent mParent;  
    ......  
   
    int mPrivateFlags;  
    ......      
  
    public void invalidate() {  
        ......  
  
        if ((mPrivateFlags & (DRAWN | HAS_BOUNDS)) == (DRAWN | HAS_BOUNDS)) {  
            mPrivateFlags &= ~DRAWN & ~DRAWING_CACHE_VALID;  
            final ViewParent p = mParent;  
            final AttachInfo ai = mAttachInfo;  
            if (p != null && ai != null) {  
                final Rect r = ai.mTmpInvalRect;  
                r.set(0, 0, mRight - mLeft, mBottom - mTop);  
                // Don't call invalidate -- we don't want to internally scroll  
                // our own bounds  
                p.invalidateChild(this, r);  
            }  
        }  
    }  
  
    ......  
	}  

 这个函数定义在文件frameworks/base/core/java/android/view/View.java中。
 View类的成员函数invalidate首先检查成员变量mPrivateFlags的DRAWN位和HAS_BOUNDS位是否都被设置为1。如果是的话，那么就说明当前视图上一次请求执行的UI绘制操作已经执行完成了，这时候View类的成员函数invalidate才可以请求执行新的UI绘制操作。

 View类的成员函数invalidate在请求新的UI绘制操作之前，会将成员变量mPrivateFlags的DRAWN位和DRAWING_CACHE_VALID位重置为0，其中，后者scheduleTraversals表示当前视图正在缓存的一些绘图对象已经失效了，这是因为接下来就要重新开始绘制当前视图的UI了。

请求绘制当前视图的UI是通过调用View类的成员变量mParent所描述的一个ViewParent接口的成员函数invalidateChild来实现的。前面我们假设当前视图是应用程序窗口的顶层视图，即它是一个类型为DecoreView的视图，它的成员变量mParent指向的是与其所关联的一个ViewRoot对象。因此，绘制当前视图的UI的操作实际上是通过调用ViewRoot类的成员函数invalidateChild来实现的。

 注意，在调用ViewRoot类的成员函数invalidateChild的成员函数invalidateChild来绘制当前视图的UI之前，会将当前视图即将要绘制的区域记录在View类的成员变量mAttachInfo所描述的一个AttachInfo对象的成员变量mTmpInvalRect中。

 ViewRoot类的成员变量mTraversalScheduled用来表示应用程序进程的UI线程是否已经调度了一个DO_TRAVERSAL消息。如果已经调度了的话，它的值就会等于true。在这种情况下， ViewRoot类的成员函数scheduleTraversals就什么也不做，否则的话，它就会首先将成员变量mTraversalScheduled的值设置为true，然后再调用从父类Handler继承下来的成员函数sendEmptyMessage来往应用程序进程的UI线程发送一个DO_TRAVERSAL消息。

这个类型为DO_TRAVERSAL的消息是由ViewRoot类的成员函数performTraversals来处理的，因此，接下来我们就继续分析ViewRoot类的成员函数performTraversals的实现。


	public final class ViewRoot extends Handler implements ViewParent,  
        View.AttachInfo.Callbacks {  
    ......  
  
    View mView;  
    ......  
  
    boolean mLayoutRequested;  
    boolean mFirst;  
    ......  
    boolean mFullRedrawNeeded;  
    ......  
  
    private final Surface mSurface = new Surface();  
    ......  
  
    private void performTraversals() {  
        ......  
  
        final View host = mView;  
        ......  
  
        mTraversalScheduled = false;  
        ......  
        boolean fullRedrawNeeded = mFullRedrawNeeded;  
        boolean newSurface = false;  
        ......  
  
        if (mLayoutRequested) {  
            ......  
  
            host.measure(childWidthMeasureSpec, childHeightMeasureSpec);  
              
            .......  
        }  
  
        ......  
  
        int relayoutResult = 0;  
        if (mFirst || windowShouldResize || insetsChanged  
                || viewVisibilityChanged || params != null) {  
            ......  
  
            boolean hadSurface = mSurface.isValid();  
            try {  
                ......  
  
                relayoutResult = relayoutWindow(params, viewVisibility, insetsPending);  
                ......  
  
                if (!hadSurface) {  
                    if (mSurface.isValid()) {  
                        ......  
                        newSurface = true;  
                        fullRedrawNeeded = true;  
                        ......  
                    }  
                }   
                ......  
            } catch (RemoteException e) {  
            }  
  
            ......  
        }  
  
        final boolean didLayout = mLayoutRequested;  
        ......  
  
        if (didLayout) {  
            mLayoutRequested = false;  
            ......  
  
            host.layout(0, 0, host.mMeasuredWidth, host.mMeasuredHeight);  
  
            ......  
        }  
  
        ......  
  
        mFirst = false;  
        ......  
  
        boolean cancelDraw = attachInfo.mTreeObserver.dispatchOnPreDraw();  
  
        if (!cancelDraw && !newSurface) {  
            mFullRedrawNeeded = false;  
            draw(fullRedrawNeeded);  
  
            ......  
        } else {  
            ......  
  
            // Try again  
            scheduleTraversals();  
        }  
    }  
  
    ......  
	}  

如果本地变量cancelDraw的值等于true，或者本地变量newSurface的值等于true，那么就说明注册到当前正在处理的应用程序窗口中的Tree Observer要求取消当前的这次重绘操作，或者当前正在处理的应用程序窗口获得了一个新的绘图表面。在这两种情况下，应用程序进程的UI线程就不能对当前正在处理的应用程序窗口的UI进行重绘了，而是要等到下一个DO_TRAVERSAL消息到来的时候，再进行重绘，以便使得当前正在处理的应用程序窗口的各项参数可以得到重新设置。下一个DO_TRAVERSAL消息需要马上被调度，因此，应用程序进程的UI线程就会重新执行ViewRoot类的成员函数scheduleTraversals

综上，android的invalidate是通过handler.sendMessage机制（scheduleTraversals方法中），把几个setRotation，setScale这些结合成一次变化的。数据是通过共享内存的方法（变量，猜的）。然后通过sendMessage来通知ViewRoot执行performTraversals。如果DO_TRAVERSAL是true，说明已经sendMessage，所以就不用send了，performTraversals会从共享变量中取出要处理的东西。如果cancelDraw为true，那就只能再调用一次scheduleTraversals，再sendMessage，来通知需要再执行这样一个操作。而draw，是在performTraversals执行的。

![](http://d.pcs.baidu.com/thumbnail/deb309efb4fe81b61d02d959e4f461e8?fid=2315310014-250528-754242941397217&time=1409648400&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-uq2pGg6Io9OshIhUIg0jgVpiMEA%3D&rt=sh&expires=2h&r=116714703&sharesign=unknown&size=c850_u580&quality=100)

	public final class ViewRoot extends Handler implements ViewParent,  
        View.AttachInfo.Callbacks {  
    ......  
  
    private void draw(boolean fullRedrawNeeded) {  
        Surface surface = mSurface;  
        ......  
  
        int yoff;  
        final boolean scrolling = mScroller != null && mScroller.computeScrollOffset();  
        if (scrolling) {  
            yoff = mScroller.getCurrY();  
        } else {  
            yoff = mScrollY;  
        }  
        if (mCurScrollY != yoff) {  
            mCurScrollY = yoff;  
            fullRedrawNeeded = true;  
        }  
        float appScale = mAttachInfo.mApplicationScale;  
        boolean scalingRequired = mAttachInfo.mScalingRequired;  
  
        Rect dirty = mDirty;  
        ......  
  
        if (mUseGL) {  
            if (!dirty.isEmpty()) {  
                Canvas canvas = mGlCanvas;  
                if (mGL != null && canvas != null) {  
                    ......  
  
                    int saveCount = canvas.save(Canvas.MATRIX_SAVE_FLAG);  
                    try {  
                        canvas.translate(0, -yoff);  
                        if (mTranslator != null) {  
                            mTranslator.translateCanvas(canvas);  
                        }  
                        canvas.setScreenDensity(scalingRequired  
                                ? DisplayMetrics.DENSITY_DEVICE : 0);  
                        mView.draw(canvas);  
                        ......  
                    } finally {  
                        canvas.restoreToCount(saveCount);  
                    }  
  
                    ......  
                }  
            }  
            if (scrolling) {  
                mFullRedrawNeeded = true;  
                scheduleTraversals();  
            }  
            return;  
        }  
  
        if (fullRedrawNeeded) {  
            ......  
            dirty.union(0, 0, (int) (mWidth * appScale + 0.5f), (int) (mHeight * appScale + 0.5f));  
        }  
  
        ......  
  
        if (!dirty.isEmpty() || mIsAnimating) {  
            Canvas canvas;  
            try {  
                ......  
                canvas = surface.lockCanvas(dirty);  
  
                ......  
            } catch (Surface.OutOfResourcesException e) {  
                ......  
                return;  
            } catch (IllegalArgumentException e) {  
                ......  
                return;  
            }  
  
            try {  
                if (!dirty.isEmpty() || mIsAnimating) {  
                    .....  
  
                    mView.mPrivateFlags |= View.DRAWN;  
  
                    ......  
                    int saveCount = canvas.save(Canvas.MATRIX_SAVE_FLAG);  
                    try {  
                        canvas.translate(0, -yoff);  
                        if (mTranslator != null) {  
                            mTranslator.translateCanvas(canvas);  
                        }  
                        canvas.setScreenDensity(scalingRequired  
                                ? DisplayMetrics.DENSITY_DEVICE : 0);  
                        mView.draw(canvas);  
                    } finally {  
                        ......  
                        canvas.restoreToCount(saveCount);  
                    }  
  
                    ......  
                }  
  
            } finally {  
                surface.unlockCanvasAndPost(canvas);  
            }  
        }  
  
        ......  
  
        if (scrolling) {  
            mFullRedrawNeeded = true;  
            scheduleTraversals();  
        }  
    }  
  
    ......  
	}  

   ViewRoot类的成员函数draw的执行流程如下所示：(13.14使用非openGL，重点，其他可以不看)

1. 将成员变量mSurface所描述的应用程序窗口的绘图表面保存在变量surface中，以便接下来可以通过变量surface来操作应用程序窗口的绘图表面。
        
2. 调用成员变量mScroller所描述的一个Scroller对象的成员函数computeScrollOffset来计算应用程序窗口是否处于正在滚动的状态中。如果是的话，那么得到的变量scrolling就会等于true，这时候调用成员变量mScroller所描述的一个Scroller对象的成员函数getCurrY就可以得到应用程序窗口在Y轴上的即时滚动位置yoff。
 
3. 成员变量mScrollY用来描述应用程序窗口下一次绘制时在Y轴上应该滚动到的位置，因此，如果应用程序窗口不是处于正在滚动的状态，那么它在下一次绘制时，就应该直接将它在Y轴上的即时滚动位置yoff设置为mScrollY。
4. 成员变量mCurScrollY用来描述应用程序窗口上一次绘制时在Y轴上的滚动位置，如果它的值不等变量yoff的值，那么就表示应用程序窗口在Y轴上的滚动位置发生变化了，这时候就需要将变量yoff的值保存在成员变量mCurScrollY中，并且将参数fullRedrawNeeded的设置为true，表示要重新绘制应用程序窗口的所有区域。
5. 成员变量mAttachInfo所描述的一个AttachInfo对象的成员变量mScalingRequired表示应用程序窗口是否正在请求进行大小缩放，如果是的话，那么所请求的大小缩放因子就保存在这个AttachInfo对象的另外一个成员变量mApplicationScale中。函数将这两个值保存在变量scalingRequired和appScale中，以便接下来可以使用。
6. 成员变量mDirty描述的是一个矩形区域，表示应用程序窗口的脏区域，即需要重新绘制的区域。函数将这个脏区域保存变量dirty中，以便接下来可以使用。
7. 成员变量mUseGL用来描述应用程序窗口是否直接使用OpenGL接口来绘制UI。当应用程序窗口的绘图表面的内存类型等于WindowManager.LayoutParams.MEMORY_TYPE_GPU时，那么就表示它需要使用OpenGL接口来绘制UI，以便可以利用GPU来绘制UI。当应用程序窗口需要直接使用OpenGL接口来绘制UI时，另外一个成员变量mGlCanvas就表示应用程序窗口的绘图表面所使用的画布，这块画布同样是通过OpenGL接口来创建的。
8. 当应用程序窗口需要直接使用OpenGL接口来绘制UI时，函数接下来就会将它的UI绘制在成员变量mGlCanvas所描述的一块画布上，这是通过调用成员变量mView所描述的一个类型为DecorView的顶层视图的成员函数draw来实现的。注意，在绘制之前，还需要对画布进行适当的转换：A. 设置画布在Y轴上的偏移值yoff，以便可以正确反映应用程序窗口的滚动状态；B. 如果成员变量mTranslator的值不等于null，即它指向了一个Translator对象，那么就说明应用程序窗口运行在兼容模式下，这时候就需要相应对画布进行变换，以便可以正确反映应用程序窗口的大小；C. 当变量scalingRequired的值等于true时，同样说明应用程序窗口是运行在兼容模式下，这时候就需要修改画布在兼容模式下的点密度，以便可以正确地反映应用程序窗口的分辨率，注意，这时候屏幕在兼容模式下的点密度保存在DisplayMetrics类的静态成员变量DENSITY_DEVICE中。由于上述画布的转换操作只针对当前的这一次绘制操作有效，因此，函数就需要在绘制之后，调用画布的成员函数save来保存它在转换前的矩阵变换堆栈状态，以便在绘制完成之后，可以调用画布的成员函数restoreToCount来恢复之前的矩阵变换堆栈状态。
9. 使用OpenGL接口来绘制完成UI后，如果变量scrolling的值等于true，即应用程序窗口是处于正在滚动的状态，那么就意味着应用程序窗口接下来还需要马上进行下一次重绘，而且是所有的区域都需要重绘，因此，函数接下来就会将成员变量mFullRedrawNeeded的值设置为true，并且调用另外一个成员函数scheduleTraversals来请求执行下一次的重绘操作。
10. 以下的步骤针适用于使用非OpenGL接口来绘制UI的情况，也是本文所要关注的重点。
11. 参数fullRedrawNeeded用来描述是否需要绘制应用程序窗口的所有区域。如果需要的话，那么就会将应用程序窗口的脏区域的大小设置为整个应用程序窗口的大小（0，0，mWidth，mHeight），其中，成员变量mWidth和mHeight表示应用程序窗口的宽度和高度。注意，如果应用程序窗口的大小被设置了一个缩放因子，即变量appScale的值不等于1，那么就需要将应用程序窗口的宽度mWidth和高度mHeight乘以这个缩放因子，然后才可以得到应用程序窗口的实际大小。
12. 经过前面的一系列计算之后，如果应用程序窗口的脏区域dirty不等于空，或者应用程序窗口在正处于动画状态，即成员变量mIsAnimating的值等于true，那么函数接下来就需要重新绘制应用程序窗口的UI了。在绘制之前，首先会调用用来描述应用程序窗口的绘图表面的一个Surface对象surface的成员函数lockCanvas来创建一块画布canvas。有了这块画布之后，接下来就可以调用成员变量mView所描述的一个类型为DecorView的顶层视图的成员函数draw来在上面绘制应用程序窗口的UI了。 与前面的第8步一样，在绘制之前，还需要对画布进行适当的A、B和C转换，以及需要在绘制之后恢复画布在绘制之前的矩阵变换堆栈状态。
13. 
**. 
 绘制完成之后，应用程序窗口的UI就都体现在前面所创建的画布canvas上了，因此，这时候就需要将它交给SurfaceFlinger服务来渲染，这是通过调用用来描述应用程序窗口的绘图表面的一个Surface对象surface的成员函数unlockCanvasAndPost来实现的。**
14. 在请求SurfaceFlinger服务渲染应用程序窗口的UI之后，函数同样是需要判断变量scrolling的值是否等于true。如果等于的话，那么就与前面的第9步一样，函数需要将成员变量mFullRedrawNeeded的值设置为true，并且调用另外一个成员函数scheduleTraversals来请求执行下一次的重绘操作。


在本文中，我们只关注使用非OpenGL接口来绘制应用程序窗口的UI的步骤，其中，第12步和第13步是关键所在。第12步调用了Java层的Surface类的成员函数lockCanvas来为应用程序窗口的绘图表面创建了一块画布，并且调用了DecorView类的成员函数draw来在这块画布上绘制了应用程序窗口的UI，而第13步调用了Java层的Surface类的成员函数unlockCanvasAndPost来将前面已经绘制了应用程序窗口UI的画布交给SurfaceFlinger服务来渲染。接下来，我们就分别分析Java层的Surface类的成员函数lockCanvas、DecorView类的成员函数draw和Java层的Surface类的成员函数unlockCanvasAndPost的实现。

 获得图形缓冲区之后，我们就可以在上面绘制应用程序窗口的UI了。由于Java层的应用程序窗口是通Skia图形库来绘制应用程序窗口的UI的，而Skia图形库在绘制UI时，是需要一块画布的，因此，函数（lockCanvas中调用的Native的Surface_lockCanvas）接下来就会将**前面所获得的图形缓冲（GraphicBuffer）区封装在一块画布中**。

 画布canvas的类型为Java层的CompatibleCanvas，它是从Canvas类继承下来的。Canvas类有一个成员变量mNativeCanvas，它指向的是一个C++层的SkCanvas对象，这个C++层的SkCanvas对象描述的就是Skia图形库绘制应用程序窗口UI时所需要的画布。no是一个类型为no_t的结构体，它的成员变量native_canvas描述的是Java层的Canvas类的成员变量mNativeCanvas在类中的偏移量，因此，通过这个偏移量就可以获得变量canvas所指向的一个Java层的CompatibleCanvas对象的内部的一块类型为SkCanvas的画布nativeCanvas。

获得了Skia图形库所需要的画布nativeCanvas之后，函数就可以将前面所获得的图形缓冲区的地址，即SurfaceInfo对象info的成员变量bits封装到它内部去了，这是**通过调用它的成员函数setPixels来实现的**

函数接下来还会设置画布nativeCanvas的裁剪区域。这个裁剪区域是通过Region对象dirtyRegion来描述的，不过Skia图形库需要使用另外一个类型为SkRegion的对象clipReg来描述它。Region对象dirtyRegion所描述的区域有可能是一个矩形区域，也可能是一个不规则的区域。如果Region对象dirtyRegion描述的是一个矩形区域，那么就可以直接将这个矩形区域设置到SkRegion的对象clipReg里面去。如果Region对象dirtyRegion描述的是一个不规则区域，那么这个不规则区域就是由一系列的矩形小区域来描述的，这时候就将这些矩形小区域合并起来，并且设置到kRegion的对象clipReg里面去。 

设置好SkRegion的对象clipReg所包含的区域之后，函数就可以调用前面得到的SkCanvas画布nativeCanvas的成员函数clipRegion来将它设置为自己的裁剪区域了，接下来函数还会将该裁剪区域所围成的一个矩形区域的位置和大小设置到参数dirtyRect所描述的一个Java层的Rect对象中去，以便调用者可以知道现在正在创建的画布的大小。

函数在将与C++层的SkCanvas画布nativeCanvas所关联的一个Java层的CompatibleCanvas画布**canvas返回给调用者之前，还会将画布的当前堆栈状态保存下来，以便在绘制完成应用程序窗口的UI之后，可以恢复回来**，这是通过调用C++层的SkCanvas画布nativeCanvas的成员函数save来实现的。画布的当前堆栈状态是通过一个整数来描述的，这个整数即为C++层的SkCanvas画布nativeCanvas的成员函数save的返回值saveCount，它会被保存在参数clazz所描述的一个Java层的Surface对象的成员变量mSaveCount中，等到应用程序窗口的UI绘制完成之后，就可以通过这个整数来恢复画布的堆栈状态了。

Surface类是使用一种称为**双缓冲**的技术来渲染应用程序窗口的UI的。这种双缓冲技术需要两个图形缓冲区，其中一个称为前端缓冲区，另外一个称为后端缓冲区。前端缓冲区是正在渲染的图形缓冲区，而后端缓冲区是接下来要渲染的图形缓冲区，它们分别通过Surface类的成员变量mPostedBuffer和mLockedBuffer所指向的两个GraphicBuffer对象来描述。

如果能将前端缓冲区的内容拷贝到后端缓冲区中去，那么就不用重新绘制应用程序窗口的所有区域，而只需要绘制那些脏的区域，即Region对象newDirtyRegion所描述的区域

 Step 8. View.draw

这个函数定义在文件frameworks/base/core/java/android/view/View.java中，它主要是完成以下六个操作：

	1. 绘制当前视图的背景。

	2. 保存当前画布的堆栈状态，并且在在当前画布上创建额外的图层，
	   以便接下来可以用来绘制当前视图在滑动时的边框渐变效果。

	3. 绘制当前视图的内容。

	4. 绘制当前视图的子视图的内容。

	5. 绘制当前视图在滑动时的边框渐变效果。

	6. 绘制当前视图的滚动条。

在上面六个操作中，有些是可以优化的。例如，如果当前视图的某一个子视图是不透明的，并且覆盖了当前视图的内容，那么当前视图的背景以及内容就不会绘制了，即不用执行第1和第3个操作。又如，如果当前视图不是处于滑动的状态，那么第2和第5个操作也是不用执行的。


本来通过View类的成员变量mPaddingLeft、mPaddingRight、mPaddingTop和mPaddingBottom就可以得到当视图的左、右、上以及下内边距的大小的，但是有时候我们在定制一个视图的时候，可能会需要在视图的内边距上绘制其它额外的东西，这时候就有扩展视图的内边距的需求。如果有扩展视图的内边距的需求，那么就需要重写View类的成员函数isPaddingOffsetRequired，即将它的返回值设置为true，并且重载另外四个成员函数getLeftPaddingOffset、getRightPaddingOffset、getTopPaddingOffset和getBottomPaddingOffset来提供额外的左、右、上以及下内边距。

奇怪的东西：

这段代码经过计算后，就得到四个值left、right、top和bottom，它们分别表示当前视图可以用来绘制的内容区域，这个区域已经将内置的和扩展的内边距排除之外。

计算好left、right、top和bottom这四个值之后，就相当于得到左、右、上以及下边框的起始位置了，但是我还需要知道边框的长度，才能确定左、右、上以及下边框所要绘制的区域。

边框的长度length设置在View类的成员变量mScrollCache所指向的一个ScrollabilityCache对象的成员变量fadingEdgeLength中。但是，这个预先设置的边框长度length不一定适合当前视图使用。这是因为视图的大小是可以随时改变的，一旦发生了改变之后，原先设置的边框长度length可能就会显得过长。具体来说，就是当上下两个边框或者左右两个边框发生重叠时，就说明原先设置的边框长度过长了。在这种情况下，就要将边框长度length修改为当前视图的内容区域的高度和宽度的较小者的一半，以便可以保证上下两个边框或者左右两个边框不会发生重叠。

[ Android应用程序窗口（Activity）的测量（Measure）、布局（Layout）和绘制（Draw）过程分析](http://blog.csdn.net/luoshengyang/article/details/8372924)

So, as developer, we need to utilize the improvements in higher version Android as much as possible :
Turn on the GPU acceleration switch above Android 3.0;
Use the new Animation Framework for your animation;
Use Layer and TextureView when necessary;
etc...
And avoid to block the main thread as much as possible, that means :
If your handle the touch events too long, do it in another thread;
If your need to load a file from sdcard or read data from database, do it in another thread;
If your need to decode a bitmap, do it in another thread;
If your View's content is too complicated, use Layer, if it continue to change, use TextureView and generate its content in another thread;
Even your can use another standalone window (such as SurfaceView) as a overlay and render it in another thread;
etc...
Golden Rules for Butter Graphics
Whatever you can do in another thread, then do it in another thread;
Whatever you must do in main thread, then do it fast;
Always profiling, it is your most dear friend;
All you need to do is keep the loop of main thread under 16ms interval, and every frame will be perfect!

[Why your Android Apps Suck](http://blog.csdn.net/rogeryi/article/details/8724233)


###[Hardware Acceleration](http://developer.android.com/intl/zh-cn/guide/topics/graphics/hardware-accel.html)

1. 由于硬件加速会用到DisplayList，所以会用更多RAM


###[Optimizing the View](http://developer.android.com/training/custom-views/optimizing-view.html)
(take a look at PieChart Example)
#### Do Less, Less Frequently
1. eliminate allocations in onDraw(), because allocations may lead to a garbage collection that would cause a stutter

2. eliminate unnecessary calls to invalidate().When possible, call the four-parameter variant of invalidate() rather than the version that takes no parameters .如invalidate(l,t,r,b)这个函数，调用这个函数，onDraw还是会执行，onDraw的结果是在canvas中画出testure(其实是Bitmap，如果是Hardware acceleration 只会update displayList)，并没有真正draw到屏幕上，就是通过invalidate(l,t,r,b)这个函数，得知dirty region，渲染程序只会更新这个区域

3. Make your view hierarchies as shallow as possible（Any time a view calls requestLayout(), the Android UI system needs to traverse the entire view hierarchy to find out how big each view needs to be.）

####Use Hardware Acceleration
Mobile GPUs are very good at certain tasks, such as scaling, rotating, and translating bitmapped images. They are not particularly good at other tasks, such as drawing lines or curves. To get the most out of GPU acceleration, you should maximize the number of operations that the GPU is good at, and minimize the number of operations that the GPU isn't good at.

In the PieChart example, for instance, drawing the pie is relatively expensive. Redrawing the pie each time it's rotated causes the UI to feel sluggish. The solution is to place the pie chart into a child View and set that View's layer type to LAYER_TYPE_HARDWARE, so that the GPU can cache it as a static image. The sample defines the child view as an inner class of PieChart, which minimizes the amount of code changes that are needed to implement this solution.

	 private class PieView extends View {

       public PieView(Context context) {
           super(context);
           if (!isInEditMode()) {
               setLayerType(View.LAYER_TYPE_HARDWARE, null);
           }
       }
       
       @Override
       protected void onDraw(Canvas canvas) {
           super.onDraw(canvas);

           for (Item it : mData) {
               mPiePaint.setShader(it.mShader);
               canvas.drawArc(mBounds,
                       360 - it.mEndAngle,
                       it.mEndAngle - it.mStartAngle,
                       true, mPiePaint);
           }
       }

       @Override
       protected void onSizeChanged(int w, int h, int oldw, int oldh) {
           mBounds = new RectF(0, 0, w, h);
       }

       RectF mBounds;
   }

After this code change, PieChart.PieView.onDraw() is called only when the view is first shown. During the rest of the application's lifetime, the pie chart is cached as an image, and redrawn at different rotation angles by the GPU. GPU hardware is particularly good at this sort of thing, and the performance difference is immediately noticeable.

There is a tradeoff, though. Caching images as hardware layers consumes video memory, which is a limited resource. For this reason, the final version of PieChart.PieView only sets its layer type to LAYER_TYPE_HARDWARE while the user is actively scrolling. At all other times, it sets its layer type to LAYER_TYPE_NONE, which allows the GPU to stop caching the image.

###例子推论：android的View有一族setScale setRotation这类图形变换。这类型图形变换没有改变到View的纹理(texture)，所以View的onDraw是不会，也不应该被执行。直接交给渲染服务就好了，而执行这些变换这是GPU长处，所以这里用Hardware acceleration是有好处

Windows会创建一个DecroView，DecroView再关联一个ViewRoot