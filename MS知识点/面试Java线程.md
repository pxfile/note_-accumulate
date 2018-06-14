面试Java线程
===


# Thread的run（）与start（）的区别
## 1.start（）方法来启动线程，真正实现了多线程运行，这时无需等待run方法体代码执行完毕而直接继续执行下面的代码：

通过调用Thread类的start()方法来启动一个线程，
这时此线程是处于就绪状态，
并没有运行。
然后通过此Thread类调用方法run()来完成其运行操作的，
这里方法run()称为线程体，
它包含了要执行的这个线程的内容，
Run方法运行结束，
此线程终止，
而CPU再运行其它线程，

## 2.run（）方法当作普通方法的方式调用，程序还是要顺序执行，还是要等待run方法体执行完毕后才可继续执行下面的代码：

而如果直接用Run方法，
这只是调用一个方法而已，
程序中依然只有主线程--这一个线程，
其程序执行路径还是只有一条，
这样就没有达到写线程的目的。
 

# [使用Semaphore控制某个方法允许并发访问的线程的个数](https://blog.csdn.net/anhenzhufeng/article/details/70225415)
Semaphore两个重要的方法就是

semaphore.acquire() 请求一个信号量，这时候的信号量个数-1（一旦没有可使用的信号量，也即信号量个数变为负数时，再次请求的时候就会阻塞，直到其他线程释放了信号量）

semaphore.release() 释放一个信号量，此时信号量个数+1
`
private Semaphore mSemaphore = new Semaphore(5);
`
# [Android java 中如何优雅的结束线程](https://blog.csdn.net/feidu804677682/article/details/43270375)

# Java Synchronize 和 Lock 的区别与用法

## 我把两者的区别分类到了一个表中，方便大家对比：

| 类别 | synchronized | Lock |
| --- | :-: | :-: |
| 存在层次 | Java的关键字，在jvm层面上 | 是一个类 |
| 锁的释放 | 1、以获取锁的线程执行完同步代码，释放锁 2、线程执行发生异常，jvm会让线程释放锁 | 在finally中必须释放锁，不然容易造成线程死锁 |
| 锁的获取 | 假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待 | 分情况而定，Lock有多个锁获取的方式，具体下面会说道，大致就是可以尝试获得锁，线程可以不用一直等待 |
| 锁状态 | 无法判断 | 可以判断 |
| 锁类型 | 可重入 不可中断 非公平 | 可重入 可判断 可公平（两者皆可） |
| 性能 | 少量同步 | 大量同步 |

## 一、synchronized和lock的用法区别

* synchronized：在需要同步的对象中加入此控制，synchronized可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象。


* lock：需要显示指定起始位置和终止位置。一般使用ReentrantLock类做为锁，多个线程中必须要使用一个ReentrantLock类做为对象才能保证锁的生效。且在加锁和解锁处需要通过lock()和unlock()显示指出。所以一般会在finally块中写unlock()以防死锁。

用法区别比较简单，这里不赘述了，如果不懂的可以看看Java基本语法。

## 二、synchronized和lock性能区别


* synchronized是托管给JVM执行的，而lock是java写的控制锁的代码。在Java1.5中，synchronize是性能低效的。因为这是一个重量级操作，需要调用操作接口，导致有可能加锁消耗的系统时间比加锁以外的操作还多。相比之下使用Java提供的Lock对象，性能更高一些。但是到了Java1.6，发生了变化。synchronize在语义上很清晰，可以进行很多优化，有适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等。导致在Java1.6上synchronize的性能并不比Lock差。官方也表示，他们也更支持synchronize，在未来的版本中还有优化余地。

* 说到这里，还是想提一下这2中机制的具体区别。据我所知，synchronized原始采用的是CPU悲观锁机制，即线程获得的是独占锁。独占锁意味着其他线程只能依靠阻塞来等待线程释放锁。而在CPU转换线程阻塞时会引起线程上下文切换，当有很多线程竞争锁的时候，会引起CPU频繁的上下文切换导致效率很低。

* 而Lock用的是乐观锁方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁实现的机制就是CAS操作（Compare and Swap）。我们可以进一步研究ReentrantLock的源代码，会发现其中比较重要的获得锁的一个方法是compareAndSetState。这里其实就是调用的CPU提供的特殊指令。

* 现代的CPU提供了指令，可以自动更新共享数据，而且能够检测到其他线程的干扰，而 compareAndSet() 就用这些代替了锁定。这个算法称作非阻塞算法，意思是一个线程的失败或者挂起不应该影响其他线程的失败或挂起的算法。

## 三、synchronized和lock用途区别

synchronized原语和ReentrantLock在一般情况下没有什么区别，但是在非常复杂的同步应用中，请考虑使用ReentrantLock，特别是遇到下面2种需求的时候。

* 1.某个线程在等待一个锁的控制权的这段时间需要中断

* 2.需要分开处理一些wait-notify，ReentrantLock里面的Condition应用，能够控制notify哪个线程

* 3.具有公平锁功能，每个到来的线程都将排队等候

# [synchronized和lock以及synchronized和volatile的区别](https://blog.csdn.net/u011296485/article/details/78007065)

# 什么是线程池，如何使用?
[线程、多线程与线程池总结](https://www.jianshu.com/p/b8197dd2934c)

# 