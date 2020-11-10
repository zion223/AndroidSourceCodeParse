# Android 动画深入分析

Android中的动画分为三种View动画、帧动画和属性动画，其中帧动画也属于View动画的一种，只不过它和平移、旋转等常见的View动画在表现形式上略有不同而已。View动画通过对场景里的对象不断做图像变换(平移、缩放、旋转、透明度)从而产生动画效果，它是一种渐进式动画,并且View动画支持自定义。帧动画通过播放一系列图像从而产生动画效果，可以简单的理解为图片切换动画，很显然，如果图片过大会产生OOM，属性动画通过动态的改变对象的属性从而产生动画效果。

## View动画
View动画的作用对象是View,它支持四种动画效果，分别是平移动画、缩放动画、旋转动画、透明度动画。

### View动画的种类
View动画的四种变化效果对应的Animation的四个子类：TranslateAnimation、AlphaAnimation、RotateAnimation、ScaleAnimation。

|   名称       | xml标签        | 子类                |   效果
|  ----        |  ----      |----                 |   ----
| 平移动画      | translate  | TranslateAnimation  | 移动View 
| 缩放动画      | scale      |ScaleAnimation       | 放大或缩小View
| 旋转动画      | rotate     |RotateAnimation     | 旋转View
| 透明度动画    | alpha      |AlphaAnimation      | 改变View的透明度
使用View动画，要先创建动画的XML文件，文件的路径为:res/anim/filename.xml
``` xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:shareInterpolator="true"
    android:fillAfter="true">
    
    <translate
        android:fromXDelta="float"
        android:toXDelta="float"
        android:fromYDelta="float"
        android:toYDelta="float"/>
     <scale
        android:fromXScale="float"
        android:toXScale="float"
        android:fromYScale="float"
        android:toYScale="float"
        android:pivotX="float"
        android:pivotY="float"/>    
     <rotate
        android:fromDegrees="float"
        android:toDegrees="float"
        android:pivotY="float"
        android:pivotX="float"/>
    <alpha 
        android:fromAlpha="float"
        android:toAlpha="float"/>

</set>

```







