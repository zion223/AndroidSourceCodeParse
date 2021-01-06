# View体系与自定义View

## 3.1 View与ViewGroup
View是Android所有控件的基类，常用的TextView、ImageView等等都是继承自View。  
常用的布局控件比如LinearLayout、RelativeLayout是继承自ViewGroup。  
ViewGroup同样也是继承自View
## 3.2 坐标系
Android系统中有两种坐标系，Android坐标系和View坐标系。
### 3.2.1 Android坐标系
在Android中，将屏幕左上角的顶点作为Android坐标系的原点，这个原点向右是X轴正方向，向下是Y轴正方向。MotionEvent中getRawX()、getRawY()获取的就是Android坐标系的坐标。
### 3.2.2 View坐标系
|  View的静态方法   | 解释  |
|  ----  | ----  |
| getLeft()  | 返回View自身左边到父布局左边的距离 |
| getTop()  | 返回View自身顶边到父布局顶边的距离 |
| getRight()  | 返回View自身右边到父布局右边的距离 |
| getBottom()  | 返回View自身右边到父布局底边的距离 |
| getX()、getY()  | View左上角相对于父容器的坐标 |  

当View没有发生平移操作时，getX()==getLeft()、getY==getTop()  
|  MotionEvent的静态方法   | 解释  |
|  ----  | ----  |
| getX()  | 获取点击事件距离控件左边的距离，即视图坐标 |
| getY()  | 获取点击事件距离控件顶边的距离，即视图坐标 |
| getRawX()  | 获取点击事件距离整个屏幕左边的距离，即绝对坐标 |
| getRawY()  | 获取点击事件距离整个屏幕顶边的距离，即绝对坐标 |
下图中的蓝色箭头表示对应于MotionEvent的事件的坐标  

![坐标系](img/坐标系.png)

## 3.3 View的滑动
### 3.3.1 layout()方法
View在进行绘制时会调用onLayout()方法来设置显示的位置，因此可以通过修改View的left、top、right、bottom这四种属性来控制View的坐标。在自定义View中，通过重写onTouchEvent()方法来实现。  
```java
public boolean onTouchEvent(MotionEvent event){
    int x = (int)event.getX();
    int y = (int)event.getY();

    switch(event.getAction){
        case MotionEvent.ACTION_DOWN:
            lastX = x;
            lasyY = y;
            break;
        case MotionEvent.ACTION_MOVE:
            int offsetX = x - lastX;
            int offsetY = y - lastY;
            layout(getLeft() + offsetX, getTop() + offsetY, getRight() + offsetX, getBottom() + offsetY);
            break;           
    }
    return true;
}

```
### 3.3.2 offsetLeftAndRight()与offsetTopAndBottom()方法
```java
public boolean onTouchEvent(MotionEvent event){
    int x = (int)event.getX();
    int y = (int)event.getY();

    switch(event.getAction){
        case MotionEvent.ACTION_DOWN:
            lastX = x;
            lasyY = y;
            break;
        case MotionEvent.ACTION_MOVE:
            int offsetX = x - lastX;
            int offsetY = y - lastY;
            // 移动View
            offsetLeftAndRight(offsetX);
            offsetTopAndBottom(offsetY);
            break;           
    }
    return true;
}
```

### 3.3.3 改变LayoutParams
可以通过改变View的参数来达到改变View位置的效果。

```java
public boolean onTouchEvent(MotionEvent event){
    int x = (int)event.getX();
    int y = (int)event.getY();

    switch(event.getAction){
        case MotionEvent.ACTION_DOWN:
            lastX = x;
            lasyY = y;
            break;
        case MotionEvent.ACTION_MOVE:
            int offsetX = x - lastX;
            int offsetY = y - lastY;
            // 移动View
            LinearLayout.LayoutParams layoutParams = (LinearLayout.LayoutParams) getLayoutParams();
            layoutParams.leftMargin +=  (int)dismenX;
            layoutParams.topMargin += (int)dismenY;
            setLayoutParams(layoutParams);
            break;           
    }
    return true;
}
```
### 3.3.4 动画
### 3.3.5 scrollTo和scrollBy
### 3.3.6 Scroller



