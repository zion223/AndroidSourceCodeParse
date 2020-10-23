# Activity生命周期和启动模式
## Activity的生命周期
### 1.典型情况下的生命周期



1. 针对一个特定的Activity 第一次启动时的生命周期回调 onCreate() -> onStart() -> onResume()
2. 当用户打开新的Activity时或者切换到桌面时当前Activity的回调 onPause() -> onStop()
3. 当用户回到该Activity时回调 onRestart() -> onStart() -> onResume()
4. 当用户back键返回到主界面时 onPause() -> onStop() -> onDestroy()

onStart()和onStop()这两个回调从是否可见的角度
onResume()和onPause()这两个回调从是否位于前台的角度

当前为ActivityA 从当前Activity启动ActivityB时回调 顺序?   
ActivityA onPasue()  
ActivityB onCreate()  
ActivityB onStart()  
ActivityB onResume()  
ActivityA onStop()

### 2.异常情况下的生命周期
1. 资源相关的系统配置发生改变导致Activity被杀死并且重新创建 
    
  常见如横竖屏切换时会发生,当发生时当前Activity会被销毁 onPause() onStop() onDestroy()回调均会被调用,同时onSaveInstanceState()方法会被调用 用来保存当前状态,这个方法会在onStop()之前被调用.重新创建Activity后onRestoreInstanceState()方法会被调用用来恢复Activity状态,onRestoreInstanceState()回调发生在onStart()后面.Android中的View在遇到此现象发生时会自动恢复状态

2. 资源内存不足时低优先级的Activity被系统杀死
   此情况的恢复过程与上述情况相同

AndroidManifest.xml中在activity节点下有android:ConfigChanges属性可以设置当此选项情况发生时不会重建Activity,配置后不会调用onSaveInstanceState()方法而是会调用onConfigurationChanged()方法

## Activity的启动模式
启动模式: LaunchMode

- standard: 标准模式,系统的默认模式。每次启动Activity时都会创建一个新的Activity实例,不管这个Activity实例是否已经存在。
- singleTop：栈顶复用模式。如果要启动的Activity已经在当前任务栈的栈顶，则此Activity不会被重新创建，同时onNewIntent()会被回调。
- singleTask：栈内复用模式，这是一种单实例模式，只要Activity在一个任务栈中存在，那么启动该Activity时不会创建新的Activity实例
- singleInstance: 单实例模式。加强的singleTask模式，具有singleTask的特性并且还有所加强，配置此模式的Activity只能单独的位于一个独立的任务栈中。

