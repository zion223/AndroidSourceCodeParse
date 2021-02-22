# 理解Window和WindowManager

  Window表示窗口的概念，某些情况下需要在桌面上显示一个类似悬浮窗的东西，这时需要使用Window来实现。Window是一个抽象类，具体实现是PhoneWindow。


## 8.1 Window和WindowManager
使用WindowManager添加Window的代码如下所示
```java
        mButton = new Button(this);
        mButton.setText("button");
        mLayoutParams = new WindowManager.LayoutParams(WindowManager.LayoutParams.WRAP_CONTENT, WindowManager.LayoutParams.WRAP_CONTENT,0,0, PixelFormat.TRANSLUCENT);
        mLayoutParams.flags = WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL| WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE| WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED;
        mLayoutParams.gravity = Gravity.LEFT| Gravity.TOP;
        mLayoutParams.x = 100;
        mLayoutParams.y = 300;
        
        getWindowManager().addView(mButton,mLayoutParams);
```
上面的代码可以将一个Button添加到屏幕坐标(100, 300)的位置上。WindowManager.LayoutParams中的flags和type这两个参数比较重要。 

flags参数表示Window的属性，常见的属性如下。
- FLAG_NOT_FOCUSABLE 表示window不需要获取焦点，也不需要接收输入事件，此标记同时启用FLAG_NOT_TOUCH_MODAL，最终事件会直接传递给下层具有焦点的Window
- FLAG_NOT_TOUCH_MODAL 和FLAG_NOT_FOCUSABLE同理
- FLAG_SHOW_WHEN_LOCKED window可以显示在锁屏的界面上

type参数表示Window的类型，**Window有三种类型，分别是应用Window、子Window和系统Window**。
- 应用类Window对应着一个Activity
- 子Window需要依赖于特定的父Window，比如Dialog
- 系统Window是需要声明权限才能创建的Window，比如Toast和系统状态栏

Window的分层的，每个Window都有对应的z-ordered，层级大的会覆盖在层级小Window上面，在三大类Window中，应用Window的层级是1~99，子Window的层级是1000~1999，系统Window的范围是2000~9999。如果想要Window位于所有Window的最顶层，那么采用较大的层级即可。如果采用系统的层级需要申请权限。
```java
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
```
WindowManager继承了ViewManager实现了其定义的方法，分别是添加View、更新View和移除View。
```java
    public void addView(View view, ViewGroup.LayoutParams params);
    public void updateViewLayout(View view, ViewGroup.LayoutParams params);
    public void removeView(View view);
```
## 8.2 Window的内部机制

Window是一个抽象的概念，每一个Window都对应一个View和ViewRootImpl，Window和View通过ViewRootImpl来建立联系，因此Window是以View的形式存在的。

### 8.2.1 Window的添加过程
Window的添加过程需要通过WindowManager的addView实现，WindowManager的实现类是WindowManagerImpl类，最终是调用WindowManagerGlobal的addView方法。

### 8.2.2 Window的删除过程

见代码注释
### 8.2.3 Window的更新过程

```java

    public void updateViewLayout(View view, ViewGroup.LayoutParams params) {
        ...
        final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams)params;
        // 设置布局参数
        view.setLayoutParams(wparams);

        synchronized (mLock) {
            int index = findViewLocked(view, true);
            ViewRootImpl root = mRoots.get(index);
            // 移除旧的参数 添加新的参数
            mParams.remove(index);
            mParams.add(index, wparams);
            root.setLayoutParams(wparams, false);
        }
    }
```
## 8.3 Window的创建过程