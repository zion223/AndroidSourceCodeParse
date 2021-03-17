# **Activity生命周期和启动模式**
## Activity的生命周期

<img src="image/activity_lifecycle.png" style="zoom:75%"/>  

### 1.典型情况下的生命周期


- 针对一个特定的Activity 第一次启动时的生命周期回调 onCreate() -> onStart() -> onResume()
- 当用户打开新的Activity时或者切换到桌面时当前Activity的回调 onPause() -> onStop()
- 当用户回到该Activity时回调 onRestart() -> onStart() -> onResume()
- 当用户按下back键返回到主界面时 onPause() -> onStop() -> onDestroy()

onStart()和onStop()这两个回调从是否可见的角度  
onResume()和onPause()这两个回调从是否位于前台的角度

当前为ActivityA 从当前Activity启动ActivityB时回调 顺序?   
```
  ActivityA onPasue()  
  ActivityB onCreate()  
  ActivityB onStart()  
  ActivityB onResume()  
  ActivityA onStop()
```


### 2.异常情况下的生命周期  

1. 资源相关的系统配置发生改变导致Activity被杀死并且重新创建 
    
&emsp; 常见如横竖屏切换时会发生，当发生时当前Activity会被销毁并且重建 onPause() onStop() onDestroy()回调均会被调用，同时onSaveInstanceState()方法会被调用 用来保存当前状态，这个方法会在onStop()之前被调用。重新创建Activity后onRestoreInstanceState()方法会被调用用来恢复Activity状态，onRestoreInstanceState()回调发生在onStart()后面。Android中的View如EditText在遇到此现象发生时会自动恢复状态。  
  多窗口模式的切换也会导致此现象发生。

2. 资源内存不足时低优先级的Activity被系统杀死
&emsp; 此情况的恢复过程与上述情况相同

AndroidManifest.xml中在activity节点下有android:ConfigChanges属性可以设置当此选项情况发生时不会重建Activity，配置后不会调用onSaveInstanceState()方法而是会调用onConfigurationChanged()方法

## Activity的启动模式  
启动模式: LaunchMode

- standard: 标准模式。系统的默认模式，每次启动Activity时都会创建一个新的Activity实例，不管这个Activity实例是否已经存在。
- singleTop: 栈顶复用模式。如果要启动的Activity已经在当前任务栈的栈顶，则此Activity不会被重新创建，同时onNewIntent()会被回调。
- singleTask: 栈内复用模式。这是一种**单实例模式**，只要Activity在一个任务栈中存在，那么启动该Activity时不会创建新的Activity实例，和singleTop一样系统也会回调其onNewIntent()
- singleInstance: 单实例模式。加强的singleTask模式，它除了具有singleTask的特性并且还有所加强，配置此模式的Activity只能单独的位于一个独立的任务栈中。

### singleTask模式场景:
  - 当前任务栈S1中有ABC，这时启动以singleTask模式配置的Activity D，此Activity需要的任务栈为S2，由于S2和D都不存在，则系统会先创建S2然后创建D的实例放入S2中。
  - 如果Activity D需要的任务栈是S1，则系统直接创建D实例然后放入S1
  - 如果Activity D需要的任务栈是S1，并且当前S1栈内的情况为ADBC四个Activity，此时D不会被重新创建，系统会把D切换到栈顶并且调用onNewIntent()，由于singleTask具有clearTop效果，于是BC会被出栈，最终任务栈S1中的情况为AD。


### 任务栈
&emsp;TaskAffinity：这个参数标识了当前Activity所需要的的任务栈的名字，默认情况下所有Activity启动的任务栈的名字为当前应用的包名。TaskAffinity属性主要和singleTask启动模式或者allowTaskReParenting属性配合使用，任务栈分为前台任务栈和后台任务栈。后台任务栈的Activity处于暂停状态，用户可以切换将后台任务栈再次调到前台。  
&emsp;&nbsp;TaskAffinity和singleTask模式配合使用时，新启动的Activity将会运行在和TaskAffinity相同名字的任务栈中。  
&emsp;&nbsp;TaskAffinity和allowTaskReParenting结合使用的时候。当应用A启动了应用B的一个ActivityC，这个Activity的allowTaskReParenting属性设置为true，那么当应用B被启动后，此ActivityC会从应用A的任务栈转移到任务B的任务栈。因此会直接启动ActivityC而不是应用B的默认Activity。

### 指定Activity启动方式
  - 通过AndroidManifest.xml中指定 android:launchMode
  - Intent启动时设置标志位(优先级更高)
