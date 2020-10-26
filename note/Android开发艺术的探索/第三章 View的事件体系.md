# **View的基础知识**

## MotionEvent和TouchSlop
1. MotionEvent
 - Action_DOWN 手指刚接触屏幕
 - ACTION_MOVE 手机在屏幕上滑动
 - ACTION_UP 手指从屏幕上松开的一瞬间

通过MotionEvent对象可以得到点击事件的x和y坐标  
getX()/getY() 相对于当前View左上角的坐标  
getRawX()/getRawY() 相对于屏幕左上角的坐标

2. TouchSlop
 TouchSlop 是系统所能识别出的被认为是滑动的最小距离,处理滑动时可以利用这个常量来做一些过滤.
 /frameworks/base/core/res/res/values/config.xml  
 `
<dimen name="config_viewConfigurationTouchSlop">8dp</dimen>
`

## VelocityTracker、GestureDetector和Scroller

VelocityTracker: 用于跟踪手指在滑动过程中的速度，包括水平和垂直方向上的速度  
GestureDetector: 手势检测，用于辅助检测用户的单击、滑动、长按、双击等行为
Scroller:弹性滑动对象，实现View的弹性滑动。

# View的滑动

## 1.使用scrollTo/scrollBy

``` java

    /**
     * Set the scrolled position of your view. This will cause a call to
     * {@link #onScrollChanged(int, int, int, int)} and the view will be
     * invalidated.
     * @param x the x position to scroll to
     * @param y the y position to scroll to
     */
    public void scrollTo(int x, int y) {
        if (mScrollX != x || mScrollY != y) {
            int oldX = mScrollX;
            int oldY = mScrollY;
            mScrollX = x;
            mScrollY = y;
            invalidateParentCaches();
            onScrollChanged(mScrollX, mScrollY, oldX, oldY);
            if (!awakenScrollBars()) {
                postInvalidateOnAnimation();
            }
        }
    }

    public void scrollBy(int x, int y) {
        scrollTo(mScrollX + x, mScrollY + y);
    }
```

使用scrollTo()和scrollBy()实现View的滑动只能将View的内容进行移动，并不能将View本身移动.

``` java
 /**
     * The offset, in pixels, by which the content of this view is scrolled
     * horizontally.
     * {@hide}
     */
    protected int mScrollX;
    /**
     * The offset, in pixels, by which the content of this view is scrolled
     * vertically.
     * {@hide}
     */
    protected int mScrollY;
```    

## 2.使用动画
可以使用属性动画完成View的移动
``` java
ObjectAnimator.ofFloat(targetView, "translationX", 0 300).setDuration(1000).start();
```
## 3.改变布局参数
通过改变View的LayoutParams即可改变View的位置

# View的事件分发机制

事件分发其实就是对于MotionEvent对象的传递过程。
``` java
public boolean dispatchTouchEvent(MotionEvent ev){
    boolean consume = false;
    if(onInterceptTouchEvent(ev)){
        consume = onTouchEvent(ev);
    }else{
        consume = child.dispatchTouchEvent() //交给子元素去处理
    }
    return consume;
}

```

![View事件传递](View事件传递.jpg)

# View的滑动冲突