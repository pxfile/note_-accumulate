android中得ANR
===


ANR即Application Not Responding 程序无响应 

由系统的Activity Manager和Window Manager系统服务监视

1.在5秒内没有响应输入的事件（例如 按键按下 屏幕触摸）

2.BroadcastReceiver在10秒内没有执行完毕

这两种情况下会提示ANR 提示用户关闭应用或等待

如何避免？

在UI线程不做耗时操作  使用loader AsyncTask handler 

# 2、典型的ANR问题场景

 · 应用程序UI线程存在耗时操作，例如在UI线程中进行网络请求、数据库操作或者文件操作，可能会导致UI线程无法及时处理用户输入等。当然在Android4.0之后，如果在UI线程中进行网络操作，将会抛出NetworkOnMainThreadException异常。

 · 应用程序的UI线程等待子线程释放某个锁，从而无法处理用户的输入。

 · 耗时的动画需要大量的计算工作，可能导致CPU负载过量。

1.耗时的网络访问

2.大量的数据读写

3.数据库操作

4.硬件操作（比如camera)

5.调用thread的join()方法、sleep()方法、wait()方法或者等待线程锁的时候

6.service binder的数量达到上限

7.system server中发生WatchDog ANR

8.service忙导致超时无响应

9.其他线程持有锁，导致主线程等待超时

10.其它线程终止或崩溃导致主线程一直等待

# 3、ANR的定位和分析

  当发生ANR时，开发者可以通过结合Logcat日志和生成的位于手机内部存储的/data/anr/traces.txt文件进行定位和分析。

那么如何避免ANR的发生呢或者说ANR的解决办法是什么呢？

1.避免在主线程执行耗时操作，所有耗时操作应新开一个子线程完成，然后再在主线程更新UI。

2.BroadcastReceiver要执行耗时操作时应启动一个service，将耗时操作交给service来完成。

3.避免在Intent Receiver里启动一个Activity，因为它会创建一个新的画面，并从当前用户正在运行的程序上抢夺焦点。如果你的应用程序在响应Intent广 播时需要向用户展示什么，你应该使用Notification Manager来实现。

1、ANR排错一般有三种类型

1.  KeyDispatchTimeout(5 seconds) --主要是类型按键或触摸事件在特定时间内无响应
2.  BroadcastTimeout(10 seconds) --BroadcastReceiver在特定时间内无法处理完成
3.  ServiceTimeout(20 secends) --小概率事件 Service在特定的时间内无法处理完成

2、哪些操作会导致ANR 在主线程执行以下操作：

1.  高耗时的操作，如图像变换
2.  磁盘读写，数据库读写操作
3.  大量的创建新对象

3、如何避免

1.  UI线程尽量只做跟UI相关的工作
2.  耗时的操作(比如数据库操作，I/O，连接网络或者别的有可能阻塞UI线程的操作)把它放在单独的线程处理
3.  尽量用Handler来处理UIThread和别的Thread之间的交互

4、解决的逻辑

1.  使用AsyncTask
    1.  在doInBackground()方法中执行耗时操作
    2.  在onPostExecuted()更新UI
2.  使用Handler实现异步任务
    1.  在子线程中处理耗时操作
    2.  处理完成之后，通过handler.sendMessage()传递处理结果
    3.  在handler的handleMessage()方法中更新UI
    4.  或者使用handler.post()方法将消息放到Looper中

5、如何排查

1.  首先分析log
2.  从trace.txt文件查看调用stack，adb pull data/anr/traces.txt ./mytraces.txt
3.  看代码
4.  仔细查看ANR的成因(iowait?block?memoryleak?)

6、监测ANR的Watchdog

最近出来一个叫LeakCanary

# FC(Force Close)
## 什么时候会出现

1.  Error
2.  OOM，内存溢出
3.  StackOverFlowError
4.  Runtime,比如说空指针异常

## 解决的办法

1.  注意内存的使用和管理
2.  使用Thread.UncaughtExceptionHandler接口
