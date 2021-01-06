# Android 系统启动

## 2.1 init进程启动过程
init进程是Android系统中用户空间的第一个进程，进程号为1，是Android系统启动过程中的一个关键的步骤，它的作用有创建Zygote和属性服务等。

### 2.1.1 引入init进程
Android的启动流程步骤如下  
1.  启动电源及系统启动  
    当电源键按下时引导芯片与预定义的地方开始执行，加载引导程序Bootloader到RAM，然后执行。
2. 引导程序到Bootloader
   Bootloader是Android操作系统开始运行前的一个小程序，主要作用是将系统OS拉起来。
3. Linux内核启动  
    内核启动，完成系统设置，在系统中寻找init.rc文件，启动init进程。
4. init进程启动
   init进程用来初始化和启动属性服务，用来启动Zygote进程。

### 2.1.2 init进程的入口函数
&ensp;&ensp;init进程的入口函数，文件位置 /system/core/init/init.cpp
init的main函数中做了很多事情，开始时创建和挂载启动所需的文件目录，都是系统运行时目录，只有在系统运行时才会存在，系统停止时会消失。  
&ensp;&ensp; main方法中的signal_handler_init函数用来设置子进程信号处理函数，它被定义在/system/core/init/signal_handler.cpp中，主要用于防止init进程成为僵尸进程



