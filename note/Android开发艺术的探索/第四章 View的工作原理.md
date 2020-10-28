# **View的工作原理**

## 初识ViewRoot和DecorView
&ensp;&ensp;ViewRoot对应于的是ViewRootImpl类，它是连接WindowManager和DecorView的纽带。在ActivityThread中当Activity对象被创建完毕后，会将DecorView添加到Window中,同时创建ViewRootImpl对象，并且将ViewRootImpl对象和DecorView建立关联。
``` java
/**
 * ViewRootImpl
 * The top of a view hierarchy, implementing the needed protocol between View
 * and the WindowManager.  This is for the most part an internal implementation
 * detail of {@link WindowManagerGlobal}.
 *
 */
```

&ensp;&ensp;View的绘制流程是从ViewRootImpl的performTraversals方法开始的，其中measure用来测量View的宽和高，layout用来确定View的位置，draw用来负责将View绘制在屏幕上。

&ensp;&ensp;DecorView作为顶级的View,一般情况下它内部会包含一个竖直方向的LinearLayout,在这个LinearLayout中有上下两部分，上面是标题栏下面是内容栏。在Activity中的onCreate()方法中通过setContentView()方法设置的layout的id就是设置内容栏的布局。
![DecorView](image/DecorView.jpg)

## 理解MeasureSpec

&ensp;&ensp;MeaureSpec代表一个32位的int值，高2位代表的是SpecMode,低30位代表的是SpecSize,SpecMode代表的是测量模式，SpecSize代表的是某种测量模式下的规格大小。SpecMode有三类在下面代码注释中有说明。
```java
        private static final int MODE_SHIFT = 30;
        private static final int MODE_MASK  = 0x3 << MODE_SHIFT;

        /**
         * Measure specification mode: The parent has not imposed any constraint
         * on the child. It can be whatever size it wants.
         */
        public static final int UNSPECIFIED = 0 << MODE_SHIFT;

        /**
         * Measure specification mode: The parent has determined an exact size
         * for the child. The child is going to be given those bounds regardless
         * of how big it wants to be.对应于LayoutParams中的match_parent和具体的数值
         */
        public static final int EXACTLY     = 1 << MODE_SHIFT;

        /**
         * Measure specification mode: The child can be as large as it wants up
         * to the specified size. 对应于LayoutParams中的wrap_content
         */
        public static final int AT_MOST     = 2 << MODE_SHIFT;
```

&ensp;&ensp;在View进行测量的时候，系统会将LayoutParams在父容器的约束下转换成对应的MeasureSpec，然后再根据这个MeasureSpec来确定View测量后的宽/高。MeasureSpec不是唯一由LayoutParams决定的，LayoutParams需要和父容器一起才决定View的MeasureSpec，从而决定View的宽/高。  
- 对于顶级View(DecorView)，其MeasureSpec由窗口的尺寸和其自身的LayoutParams来共同确定
- 对于普通View，其MeasureSpec由父容器的MeasureSpec和其自身的LayoutParams来确定

对于顶级View的情况: ViewRootImpl中的measureHierarchy()方法中显示了DecorView的MeasureSpec的创建过程。
```java
    childWidthMeasureSpec = getRootMeasureSpec(baseSize, lp.width);
    childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
```
getRootMeasureSpec()方法实现如下
```java
private static int getRootMeasureSpec(int windowSize, int rootDimension) {
        int measureSpec;
        switch (rootDimension) {

        case ViewGroup.LayoutParams.MATCH_PARENT:
            // Window can't resize. Force root view to be windowSize.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
            break;
        case ViewGroup.LayoutParams.WRAP_CONTENT:
            // Window can resize. Set max size for root view.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
            break;
        default:
            // Window wants to be an exact size. Force root view to be that size.
            measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
            break;
        }
        return measureSpec;
    }
```

对于普通的View的来说，View的measure过程由ViewGroup传递过来
``` java
    protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }

```

从代码可以看出来子View的MeasureSpec的创建与父容器的MeasureSpec和其自身的LayoutParams有关。此外还与View的margin及padding有关。

```java 
 public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }

```

## View的工作流程

View的工作流程主要是指measure、layout、draw这三大流程，即测量、布局和绘制，其中measure主要确定View的宽和高,layout确定View的最终宽/高和四个顶点的位置，而draw会将View绘制到屏幕上。

### measure过程
- View的measure过程
  
![View的Measure过程](image/View的Measure过程.jpg)

从getDefaultSize()方法来看,View的宽/高由specSize决定，由此可以得出结论:直接继承自View的自定义控件需要重写onMeasure()方法并且设置wrap_content时的自身大小，否则在布局中使用wrap_content和使用match_parent的效果一样。解决此问题的代码如下
```java
@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec,heightMeasureSpec);
        
        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);
        //分析模式，根据不同的模式来设置
        if(widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST){
            setMeasuredDimension(mWidth, mHeight);
        }else if(widthSpecMode == MeasureSpec.AT_MOST){
            setMeasuredDimension(mWidth, heightSpecSize);
        }else if(heightSpecMode == MeasureSpec.AT_MOST){
            setMeasuredDimension(widthSpecSize, mHeight);
        }
    }
```
- ViewGroup的measure过程  
  ViewGroup除了完成自己的measure过程以外，还会去遍历调用所有子元素的measure方法，各个子元素再去递归的执行这个过程，与View不同的是ViewGroup是一个抽象类，因此它没有重写View的onMeasure方法，但是提供了一个measureChildren()方法

  ![ViewGroup的Measure过程](image/ViewGroup的Measure过程.jpg)  
  ViewGroup没有定义其测量的具体过程，那是因为ViewGroup是一个抽象类，其测量过程的onMeasure方法需要各自子类去实现。  
  当measure完成后，就可以通过view.getMeasuredWidth/Height方法就可以正确的获取到View的宽和高。  

  **如何在Activity中获取view的宽度和高度 ?**  
  Activity的生命周期和view的measure过程并没有同步，因为无法再Activity的生命周期回调中获取view的宽度和高度。  
  1. Activity/View#onWindowFocusChanged
    onWindowFocusChanged()回调发生时View已经初始化完毕了，可以再此方法中获取View的宽度和高度，需要注意的是此方法可能会被调用多次。
  2. view.post(runnable)
    通过post方法可以将一个runnable投递到消息队列的尾部，然后等到Looper调用此runnable的时候，View已经初始化完毕了。
    ```java
    protected void onStart(){
        super.onStart();
        view.post(new Runnable(){

            @Override
            public void run(){
                int width = view.getMeasuredWidth();
                int height = view.getMeasuredHeight();
            }
        });
    }
    ```
    3. ViewTreeObserver  
        使用ViewTreeObserver的众多接口回调可以完成这个功能，比如使用OnGlobalLayoutListener这个接口。
    4. view.measure(int widthMeasureSpec, int heightMeasureSpec)  
    手动对View进行measure来得到View的宽/高     

### layout过程
View的绘制流程是从ViewRootImpl的performTraversals方法开始的，在此方法中会依次调用performMeasure()、performLayout()、performDraw()。
``` java
private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,
        int desiredWindowHeight) {
    mLayoutRequested = false;
    mScrollMayChange = true;
    mInLayout = true;

    final View host = mView;
    if (DEBUG_ORIENTATION || DEBUG_LAYOUT) {
        Log.v(TAG, "Laying out " + host + " to (" +
                host.getMeasuredWidth() + ", " + host.getMeasuredHeight() + ")");
    }

    Trace.traceBegin(Trace.TRACE_TAG_VIEW, "layout");
    try {
        //调用View的layout方法
        host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight()); // 1

        //省略...
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_VIEW);
    }
    mInLayout = false;
}

```
从上面代码可以看出在调用DecorView的layout方法时传递的参数分别是上下左右四个位置的坐标。接下来是View#layout方法

```java 
public void layout(int l, int t, int r, int b) {
        if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
            onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        }

        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;
        //调用setFrame方法
        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            //调用onLayout方法
            onLayout(changed, l, t, r, b);
            mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnLayoutChangeListeners != null) {
                ArrayList<OnLayoutChangeListener> listenersCopy =
                        (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
                int numListeners = listenersCopy.size();
                for (int i = 0; i < numListeners; ++i) {
                    listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
                }
            }
        }

        mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
        mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
    }
```
调用setFrame()方法来设定View的四个顶点的位置，View的四个顶点的位置一旦确定，那么View在父容器中的位置也确定了，接着调用onLayout方法用来在父容器中确定子元素的位置。View的onLayout方法是空实现。需要交由子类去实现。FrameLayout中的实现如下
```java
@Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        layoutChildren(left, top, right, bottom, false /* no force left gravity */);
    }

    void layoutChildren(int left, int top, int right, int bottom, boolean forceLeftGravity) {
        final int count = getChildCount();
        //影响子View的布局参数
        final int parentLeft = getPaddingLeftWithForeground();
        final int parentRight = right - left - getPaddingRightWithForeground();

        final int parentTop = getPaddingTopWithForeground();
        final int parentBottom = bottom - top - getPaddingBottomWithForeground();
        //循环遍历子View
        for (int i = 0; i < count; i++) {
            final View child = getChildAt(i);
            //不为GONE类型的
            if (child.getVisibility() != GONE) {
                //子View的布局参数
                final LayoutParams lp = (LayoutParams) child.getLayoutParams();

                final int width = child.getMeasuredWidth();
                final int height = child.getMeasuredHeight();

                int childLeft;
                int childTop;

                int gravity = lp.gravity;
                if (gravity == -1) {
                    gravity = DEFAULT_CHILD_GRAVITY;
                }

                final int layoutDirection = getLayoutDirection();
                final int absoluteGravity = Gravity.getAbsoluteGravity(gravity, layoutDirection);
                final int verticalGravity = gravity & Gravity.VERTICAL_GRAVITY_MASK;
                //水平方向的layout_gravity参数
                switch (absoluteGravity & Gravity.HORIZONTAL_GRAVITY_MASK) {
                    /* 水平居中，由于子View要在水平中间的位置显示，因此，要先计算出以下：
                    * (parentRight - parentLeft -width)/2 此时得出的是父容器减去子View宽度后的
                    * 剩余空间的一半，那么再加上parentLeft后，就是子View初始左上角横坐标(此时正好位于中间位置)，
                    * 假如子View还受到margin约束，由于leftMargin使子View右偏而rightMargin使子View左偏，所以最后
                    * 是 +leftMargin - rightMargin .
                    */
                    case Gravity.CENTER_HORIZONTAL:
                        childLeft = parentLeft + (parentRight - parentLeft - width) / 2 +
                        lp.leftMargin - lp.rightMargin;
                        break;
                    //水平居右 父容器Right-子View宽度 - 子View右Margin    
                    case Gravity.RIGHT:
                        if (!forceLeftGravity) {
                            childLeft = parentRight - width - lp.rightMargin;
                            break;
                        }
                    //不设置layout_gravity时默认就是靠左 子View的左Margin+父容器的左Margin    
                    case Gravity.LEFT:
                    default:
                        childLeft = parentLeft + lp.leftMargin;
                }

                switch (verticalGravity) {
                    case Gravity.TOP:
                        childTop = parentTop + lp.topMargin;
                        break;
                    case Gravity.CENTER_VERTICAL:
                        childTop = parentTop + (parentBottom - parentTop - height) / 2 +
                        lp.topMargin - lp.bottomMargin;
                        break;
                    case Gravity.BOTTOM:
                        childTop = parentBottom - height - lp.bottomMargin;
                        break;
                    default:
                        childTop = parentTop + lp.topMargin;
                }
                //调用子View自己的layout方法
                child.layout(childLeft, childTop, childLeft + width, childTop + height);
            }
        }
    }
```

### draw过程

  ![View的draw过程](image/View的draw过程.jpg)






     







