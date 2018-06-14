---
layout:     post
title:      Android的AMS和WMS理解
subtitle:   Android的AMS和WMS理解
date:       2018-03-12
author:     pxf
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Android
---
Android的AMS和WMS理解
==

## 浅谈ActivityManagerService

### ActivityManagerService
AmS---ActivityManagerService.java，android系统服务，Activity管理的服务端，用于管理activity的各种行为，控制activity的生命周期，派发消息事件，低内存管理等，实现了IBinder接口，可以用于进程间通信

### ApplicationThread
ApplicationThread.java 实现了IBinder 接口，activity整个框架中客户端和服务器端(AmS)通信的接口，同时也是类ActivityThread的内部类，这样就可以吧AmS和ActivityThread绑在了一起，这种结构有点像23中java设计模式中结构性模型中代理模式的样子。。。。

### ActivityThread
ActivityThread.java ApplicationThread所绑定的客户端就是ActivityThread。ActivityThread这个类在activity客户端很重要：

* 它是应用程序的入口，众所周知。java程序的入口是main()方法，同样，AmS拉起一个新的进程，同时启动一个主线程的时候，主线程就是从ActivityThread.main方法开始执行，他会初始化一些对象，然后自己进入消息等待队列，也就是Looper.loop(),一旦进入loop()方法，线程就进入死循环，再也不会退出，一直在等待别人给他消息，然后执行这个消息，这也是事件驱动模型的原理

* 它是Activity客户端的管理类，由他来决定，什么时候调用onCreate()。什么时候调用onResume()方法，当Activity发起一个请求时，比如startActivity()，它会处理这个请求，然后调用其他的人来做具体的事

### Instrumentation
Instrumentation.java, 这个类除了跟android的测试有关之外，还是activity管理中实际做事情的人。比如，startActivity(), 在某种情况下，就是调用这个类，然后调用到ams.当然有个时候是通过ApplicationThread去访问Ams的。
问题：有的同学就会想了，既然ApplicationThread可以访问Ams，那么我们应用程序是不是可以调用ApplicationThread类来访问ams，直接启动Activity呢？
答案是不可以的。因为ApplicationThread是ActivityThead的私有内部类，外界无法访问。而ActivityThread是hide的，所以，我们应用程序不能通过sdk去访问ActivityThread类。同理，拿到Ams引用的接口是ActivityManagerNative类，而这个类也是hide的。所以应用程序也不能直接操作ams.

### Activity
Activity.java，这个类很熟悉，android最重要的组件之一，其实从java角度来看，他一点也不特殊，本身就是一个java类，他会被创建，同时也会被垃圾回收机制回收，只不过它受AmS管理，所以他才有她的生命周期，显得比较重要。

### ActivityResult
ActivityResult.java。Activity管理服务类中，activity的记录缓冲，话句话说，就是客户端启动一个activity，AmS会对他进行缓冲，类型就是ActivityResult，我们可能想，为什么不直接缓冲Activity类型的值，而要重新定义一个ActivityResult类型呢

   因为Activity类没有实现IBinder接口，不能用于进程之间通信，而这个缓冲值是AmS和客户端通用的，所以必须IBinder接口。而且Activity设计的时候，也没有打算让它用于进程通信，看他的成员变量就可以知道，里面有很多的庞大对象，如list map等，这些对象不适合跨进程传输的，因为java里面跨进程调用就是序列化和反序列化，那些庞大对象在序列化和反序列化的时候，会消耗大量的时候和资源。

### AMS是什么？

全名是ActivityManagerService，就是Activity管理机制的服务器端，属于一个系统服务，位置systemProcess进程中，

可以从多个角度来看ams：

    1 从java角度来看，ams就是一个java对象，实现了Ibinder接口，所以它是一个用于进程之间通信的接口，这个对象初始化是在systemServer.java 的run()方法里面。
```
Slog.i(TAG, "Activity Manager");
context = ActivityManagerService.main(factoryTest);
```
这是一段很普通的java静态工厂代码，我们在这里创建我们的Ams对象。
初始化方法
```
public static final Context main(int factoryTest) {
AThread thr = new AThread();
        thr.start();  //启动一个线程去初始化Ams对象
        synchronized (thr) {
            while (thr.mService == null) {
                try {
                    thr.wait(); //等待AThread初始化Ams完成
                } catch (InterruptedException e) {
                }
            }
}
//…
         //…..
}
```
AThread的run()方法

```
public void run() {
            Looper.prepare();
            android.os.Process.setThreadPriority(
                    android.os.Process.THREAD_PRIORITY_FOREGROUND);
            android.os.Process.setCanSelfBackground(false);

            ActivityManagerService m = new ActivityManagerService();  //初始化Ams对象 

            synchronized (this) {
                mService = m;
                notifyAll();  //初始化完成 ，notify main方法的wait()方法，告诉它已经完成，可以往下面走了。
            }
     //……
   Looper.loop();  //自己进入消息循环等待状态
}
```

#### 思考：
这里是另外启动了一个线程AThread去创建Ams对象，然后用wait/notify机制来实现线程之间的同步。
* 创建一个对象完全不必要另外开启个线程来处理，直接new出来不就可以了。有画蛇添足的感觉
* AThread在创建了Ams对象之后，进入消息循环状态而不是直接退出。但是看代码没有发现有向这个线程的handler发送消息的地方，也就是说AThread线程的状态接下来会永远阻塞。不明白这样做有什么意义。

* 思考结果：
它是AThread的run方法里面new ActivityManagerService()， 而ActivityManagerService里面有个变量mHandler，这样的话mHandler绑定的线程就是AThread这个线程了。通过向mHandler发消息，AThread就不停地从消息队列获取消息，处理消息。所以，上面2问题都已经解决。

#### AMS是一个服务

   ActivityManagerService从名字就可以看出，它是一个服务，用来管理Activity，而且是一个系统服务，就是包管理服务，电池管理服务，震动管理服务等。

#### AMS是一个Binder

ams实现了Ibinder接口，所以它是一个Binder，这意味着他不但可以用于进程间通信，还是一个线程，因为一个Binder就是一个线程。

  如果我们启动一个hello World安卓用于程序，里面不另外启动其他线程，这个里面最少要启动4个线程

   * main线程，只是程序的主线程，也是日常用到的最多的线程，也叫UI线程，因为android的组件是非线程安全的，所以只允许UI/MAIN线程来操作。

  * GC线程，java有垃圾回收机制，每个java程序都有一个专门负责垃圾回收的线程，

  * Binder1 就是我们的ApplicationThread，这个类实现了Ibinder接口，用于进程之间通信，具体来说，就是我们程序和AMS通信的工具

  *  Binder2 就是我们的ViewRoot.W对象，他也是实现了IBinder接口，就是用于我们的应用程序和wms通信的工具。

  wms就是WindowManagerServicer ，和ams差不多的概念，不过他是管理窗口的系统服务。

####  AmS是一个单独的线程

* 首先系统服务不可能运行在systemProcess进程的主进程中，因为这么多系统服务如果都是运行在主线程中的话，肯定会发生阻塞，比如这个线程正在管理Activity，然后手机来震动了，那就没有来得及处理震动服务，所以一般的服务都是运行在单独的线程里面

* AmS实现了Ibinder接口，所以他就是一个单独的进程，就是这个线程来处理AmS对Activity的管理，

* **疑问**: 如果Ams只有一个线程来执行Activity管理的话，会不会发生阻塞。比如两个客户端同时发起请求。
我自己的看法是应该不会。因为手机操作系统是单界面的操作系统，就是说，在同一个时刻，只有一个Activity能展示在用户面前(TabActivity看起来同时展示了多个Activity，但安卓做了特殊的处理，暂不讨论)，不像windows是多任务多窗口的操作系统。单窗口的话，那么发送的onPause(),onResume()之类的请求时，在同一时刻，应该只有一个人发起。

#### Ams管理Activity流程
以Activity.startActivityForResult()为例说明,一般我们见的最多的是调用Activity.startActivity()方法。不过startActivity()的内部实现也是调用startActivityForResult()方法，只不过requestCode=-1
Activity.startActivity()开启一个Activity是异步的，换句话来说，Activity不必等待ams返回的结果，直接往下面执行，当把接下来的代码执行完毕以后，他就进入消息循环等待队列，继续等待被人给他的消息，否则就一直阻塞，不过ams很快就会给他发送一个暂停当前activity的消息

##  [WMS](https://www.aliyun.com/jiaocheng/android_1772.html)

## [Android中APP、AMS、WMS的Binder IPC](http://blog.csdn.net/qq526459753/article/details/51034625)
