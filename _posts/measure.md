##Measure layout draw

http://blog.csdn.net/luoshengyang/article/details/8372924

##Measure

>>The measure pass uses two classes to communicate dimensions. The ViewGroup.LayoutParams class is used by View objects to tell their parents how they want to be measured and positioned. The base ViewGroup.LayoutParams class just describes how big the View wants to be for both width and height. For each dimension, it can specify one of:

>>an exact number
MATCH_PARENT, which means the View wants to be as big as its parent (minus padding)
WRAP_CONTENT, which means that the View wants to be just big enough to enclose its content (plus padding).
There are subclasses of ViewGroup.LayoutParams for different subclasses of ViewGroup. For example, RelativeLayout has its own subclass of ViewGroup.LayoutParams, which includes the ability to center child View objects horizontally and vertically.

>>MeasureSpec objects are used to push requirements down the tree from parent to child. A MeasureSpec can be in one of three modes:

>>UNSPECIFIED: This is used by a parent to determine the desired dimension of a child View. For example, a LinearLayout may call measure() on its child with the height set to UNSPECIFIED and a width of EXACTLY 240 to find out how tall the child View wants to be given a width of 240 pixels.
EXACTLY: This is used by the parent to impose an exact size on the child. The child must use this size, and guarantee that all of its descendants will fit within this size.
AT MOST: This is used by the parent to impose a maximum size on the child. The child must guarantee that it and all of its descendants will fit within this size.

Measure是一个top-down traversal的过程，子通过`ViewGroup.LayoutParams`告诉父，自己希望多大，而父通过`MeasureSpec`来限制子的size。也就是说，size是一个协商的过程，子告诉父希望多大，父通过MeasureSpec来告诉子一些限制（如子希望wrapContent，父就可能对子说，AT_MOST 父size(如100px)）。

ViewGroup中有一个`measureChildWithMargins`的方法，这个方法会通过调用`child.measure(childWidthMeasureSpec, childHeightMeasureSpec);`这个语句，让子View进行measure。

其中`measureChildWithMargins`调用`ViewGroup.getChildMeasureSpec`正是这个方法，child把自己希望的大小`ViewGroup.LayoutParams`告诉ViewGroup的

而View中得measure调用onMeasure。一般measure是不被Override的（ViewGroup也补Override measure），只重载onMeasure。所以一般来说整个过程就是ViewGroup.measure->ViewGroup(也就是View中的).OnMeasure->child.measure->child.onMeasure。
*ps父要负责调用child.measure来让child测量*

也就是在ViewGroup的子类的onMeasure中，父可以获取child.layoutParam 这样child就告诉了父自己希望的大小。而父经过一系列处理通过getChildMeasureSpec方法，计算出对子的size的限制，然后通过child.measure()也就是`View.measure(int widthMeasureSpec, int heightMeasureSpec) `

Framework只是提供了方法，和ViewGroup，view的子类，具体怎么用，还要看具体的ViewGroup 和 View的子类。

onMeasure一个很重要的是，必须把结果通过`setMeasuredDimension`方法设置回去，就是设置了

		private void setMeasuredDimensionRaw(int measuredWidth, int measuredHeight) {
        mMeasuredWidth = measuredWidth;
        mMeasuredHeight = measuredHeight;

        mPrivateFlags |= PFLAG_MEASURED_DIMENSION_SET;
    }
如果没有设置会包异常(通过检测状态位PFLAG_MEASURED_DIMENSION_SET)。

好了，这里就可以看到了

	public final int getMeasuredHeight() {
        return mMeasuredHeight & MEASURED_SIZE_MASK;
    }
	
	public final int getHeight() {
        return mBottom - mTop;
    }	
    
getWidth getHeight实质上还要经过layout的，而getMeasuredHeight是Measure的结果，当然可以在layout阶段设置width ！= mMeasuredWidth。

在进一步分析前，先看看MeasureSpec是怎样做到父向子传递约束要求的
###MeasureSpec

	    public static class MeasureSpec {
        private static final int MODE_SHIFT = 30;
        private static final int MODE_MASK  = 0x3 << MODE_SHIFT;

        
        public static final int UNSPECIFIED = 0 << MODE_SHIFT;
        
        public static final int EXACTLY     = 1 << MODE_SHIFT;
        
        public static final int AT_MOST     = 2 << MODE_SHIFT;


        public static int makeMeasureSpec(int size, int mode) {
            return size + mode;
        }

        
        public static int getMode(int measureSpec) {
            return (measureSpec & MODE_MASK);
        }

        
        public static int getSize(int measureSpec) {
            return (measureSpec & ~MODE_MASK);
        }

        ...
    }
    
MeasureSpec由Mode和Size组成，由一个32位int描述，其中高两位用来表示mode，剩下的30位用来表示size。注意，由于`UNSPECIFIED = 0 << MODE_SHIFT`所以在UNSPECIFIED模式下，MeasureSpec就是size，其他两个mode不可以这样

另外measure还有measureState：

    /**
     * Bits of {@link #getMeasuredWidthAndState()} and
     * {@link #getMeasuredWidthAndState()} that provide the actual measured size.
     */
    public static final int MEASURED_SIZE_MASK = 0x00ffffff;

    /**
     * Bits of {@link #getMeasuredWidthAndState()} and
     * {@link #getMeasuredWidthAndState()} that provide the additional state bits.
     */
    public static final int MEASURED_STATE_MASK = 0xff000000;

    /**
     * Bit shift of {@link #MEASURED_STATE_MASK} to get to the height bits
     * for functions that combine both width and height into a single int,
     * such as {@link #getMeasuredState()} and the childState argument of
     * {@link #resolveSizeAndState(int, int, int)}.
     */
    public static final int MEASURED_HEIGHT_STATE_SHIFT = 16;

    /**
     * Bit of {@link #getMeasuredWidthAndState()} and
     * {@link #getMeasuredWidthAndState()} that indicates the measured size
     * is smaller that the space the view would like to have.
     */
    public static final int MEASURED_STATE_TOO_SMALL = 0x01000000;
    
那这个measureState又是在哪里使用的呢？
View中有的

	public static int resolveSizeAndState(int size, int measureSpec, int childMeasuredState) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize =  MeasureSpec.getSize(measureSpec);
        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
            if (specSize < size) {
                result = specSize | MEASURED_STATE_TOO_SMALL;
            } else {
                result = size;
            }
            break;
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result | (childMeasuredState&MEASURED_STATE_MASK);
    }
    
    
    
 只有这个地方有设置MeasureState，而且只有普通（不设置）和 MEASURED_STATE_TOO_SMALL两种state.但是`MEASURED_STATE_MASK = 0xff000000`也就是说其他地方可以扩展到2^8种State


To be continue measure 中还有一个state，那个state，也就是resolveSizeAndState的state又是什么，和MeasureSpce的mode有没有联系（倾向没有）