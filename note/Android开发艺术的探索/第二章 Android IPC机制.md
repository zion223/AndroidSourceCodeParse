# 第二章 IPC机制

## 2.1 Android IPC简介  
&emsp; IPC 是Inter-Process Communication的缩写，含义为进程间通信或跨进程通信。是指两个进程间进行数据交换的过程。    
&emsp; 进程和线程的区别和共同点?
- 线程：线程是CPU调度的最小单元，同时线程是一种有限的系统资源。
- 进程：进程一般指是一个执行单元，在PC和移动设备上是指一个程序或者一个应用。

&emsp;一个进程可以包含多个线程，即进程和线程之前是包含和被包含的关系。在Android中，主线程也叫UI线程，在UI线程中才能更新界面View。  

&emsp;IPC不是Android中独有的，任何一个操作系统都需要有相应的IPC机制，比如Linux中可以通过命名管道、共享内容、信号量来进行进程间通信。Android中特有的进程间通信方式就是Binder，通过Socket也可以实现进程间通信。

## 2.2 Android中的多进程模式

### 2.2.1 开启多进程模式
&emsp;正常情况下 Android中的多进程指的是一个应用中存在多个进程的情况，在Android中开启多进程的方式只有一种，就是在AndroidManifest.xml中给四大组件中指定android:process属性。默认不指定此属性时，默认的进程名就是当前应用的包名。

### 2.2.2 多进程模式的运行机制
&emsp;Android 会为每一个应用分配一个单独的虚拟机，或者说为每一个单独的进程分配一个独立的虚拟机，不同的虚拟机在内存分配上有不同的地址空间，这就导致不同的虚拟机中访问同一个类的对象会产生多份内存副本。  
所有运行在不同进程的四大组件，只要他们之间需要通过内存来共享数据，都会共享失败(内存地址不同)，这就是多进程带来的影响。一般来说，多进程会造成如下几个方面的问题。
- 静态成员和单例模式失效
- 线程同步机制完全失效
- SharedPreperences的可靠性下降
- Application会多次创建

## 2.3 IPC基础概念介绍
&emsp;Serializable和Parcelable接口可以完成对象的序列化过程，当我们需要通过Intent和Binder传输数据时需要使用Serializable或者Parcelable。还有的时候需要把对象持久化到存储设备上或者通过网络传输给其他客户端时，这个时候也需要进行对象的序列化。
### 2.3.1 Serializable接口 
&emsp;Serializable是Java所提供的一个序列化接口，是一个空接口，为对象提供标准的序列化和反序列化操作。
### 2.3.2 Parcelable接口 
&emsp;Parcelable接口是Android平台提供的序列化接口。实现此接口也可以实现类的序列化和反序列化操作。

&emsp;二者的相同点和区别?  

  |               | 提供方   | 性能   | 使用  |   功能
|  ----         | ----    |----    |----   |   ----
| Serializable  | Java    | 差     | 简单  | 实现序列化 
| Parcelable    | Android | 好     |复杂   | 实现序列化 

&emsp;Parcelable是Android推荐的序列化方式。
### 2.3.3 Binder

## 2.4 Android中的IPC方式

### 2.4.1 使用Bundle
### 2.4.2 使用文件共享
### 2.4.3 使用Messenger
### 2.4.4 使用AIDL
### 2.4.5 使用ContentProvider
### 2.4.6 使用Socket

## 2.5 Binder连接池
## 2.6 选用合适的IPC方式




