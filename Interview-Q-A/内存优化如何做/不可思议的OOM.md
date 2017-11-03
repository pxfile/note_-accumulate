# 不可思议的OOM
OOM 崩溃时，java 堆内存远远低于 Android 虚拟机设定的上限，并且物理内存充足，SD 卡空间充足，既然内存充足，这时候为什么会有 OOM 崩溃呢？

在详细描述问题之前，先弄清楚一个问题：

###**什么导致了 OOM 的产生？**

下面是几个关于 Android 官方声明内存限制阈值的 API：

```
ActivityManager.getMemoryClass()：     虚拟机 java 堆大小的上限，分配对象时突破这个大小就会 OOM
ActivityManager.getLargeMemoryClass()：manifest 中设置 largeheap=true 时虚拟机 java 堆的上限
Runtime.getRuntime().maxMemory() ：    当前虚拟机实例的内存使用上限，为上述两者之一
Runtime.getRuntime().totalMemory() ：  当前已经申请的内存，包括已经使用的和还没有使用的
Runtime.getRuntime().freeMemory() ：   上一条中已经申请但是尚未使用的那部分。那么已经申请并且正在使用的部分 used=totalMemory() - freeMemory()
ActivityManager.MemoryInfo.totalMem:   设备总内存
ActivityManager.MemoryInfo.availMem:   设备当前可用内存
/proc/meminfo                                           记录设备的内存信息
```
通常认为 OOM 发生是由于 java 堆内存不够用了，即

```
Runtime.getRuntime().maxMemory() 这个指标满足不了申请堆内存大小时
```

这种 OOM 可以非常方便的验证（比如: 通过 new byte[] 的方式尝试申请超过阈值 maxMemory() 的堆内存），通常这种 OOM 的错误信息通常如下：

```
java.lang.OutOfMemoryError: Failed to allocate a XXX byte allocation with XXX free bytes an
```
发现的 OOM 案例中堆内存充裕（Runtime.getRuntime().maxMemory() 大小的堆内存还剩余很大一部分），设备当前内存也很充裕（ActivityManager.MemoryInfo.availMem 还有很多）。这些 OOM 的错误信息大致有下面两种：

1 . 这种 OOM 在 Android6.0，Android7.0 上各个机型均有发生，文中简称为 OOM 一，错误信息如下：

```
java.lang.OutOfMemoryError: Could not allocate JNI Env
```
2 . 集中发生在 Android7.0 及以上的华为手机（EmotionUI_5.0 及以上）的 OOM，简称为 OOM 二，对应错误信息如下：

```
java.lang.OutOfMemoryError: pthread_create (1040KB stack) failed: Out of memory
```

既然抛出来 OOM，一定是线程创建过程中触发了某些我们不知道的限制，既然不是 Art 虚拟机为我们设置的堆上限，那么可能是更底层的限制。Android 系统基于 linux，所以 linux 的限制对于 Android 同样适用，这些限制有：

**1 ./proc/pid/limits 描述着 linux 系统对对应进程的限制，**

下面是一个样例：

```
Limit                     Soft Limit           Hard Limit           Units     
Max cpu time              unlimited            unlimited            seconds   
Max file size             unlimited            unlimited            bytes     
Max data size             unlimited            unlimited            bytes     
Max stack size            8388608              unlimited            bytes     
Max core file size        0                    unlimited            bytes     
Max resident set          unlimited            unlimited            bytes     
Max processes             13419                13419                processes 
Max open files            1024                 4096                 files     
Max locked memory         67108864             67108864             bytes     
Max address space         unlimited            unlimited            bytes     
Max file locks            unlimited            unlimited            locks     
Max pending signals       13419                13419                signals   
Max msgqueue size         819200               819200               bytes     
Max nice priority         40                   40                   
Max realtime priority     0                    0                    
Max realtime timeout      unlimited            unlimited            us
```
**用排除法筛选上面样例中的 limits:**

**Max stack size，Max processes** 的限制是整个系统的，不是针对某个进程的，排除 ;

**Max locked memory ，**排除，后面会分析，线程创建过程中分配线程私有 stack 使用的 mmap 调用没有设置 

**MAP_LOCKED，**所以这个限制与线程创建过程无关 ;

**Max pending signals，**c 层信号个数阈值，无关，排除 ;

**Max msgqueue size，Android IPC**机制不支持消息队列，排除。

剩下的 limits 项中，**Max open files** 这一项限制最可疑 **Max open files** 表示 **每个进程最大打开文件的数目**，进程 **每打开一个文件就会产生一个文件描述符 fd（记录在 /proc/pid/fd 下面）**，这个限制表明 **fd 的数目不能超过 Max open files 规定的数目。**
后面分析线程创建过程中会发现过程中涉有及到文件描述符。

**2.  /proc/sys/kernel 中描述的限制**

这些限制中与线程相关的是 **/proc/sys/kernel/threads-max，**规定了每个进程创建线程数目的上限，所以线程创建导致 OOM 的原因也有可能与这个限制相关。
 
**3. 验证**
 
下面对上述的推断进行验证，分两步：
本地验证和线上验收。

* **本地验证：在本地验证推断，**试图复现如下所示错误消息一致的 OOM

* **线上验收：下发插件，**验收线上用户 OOM 时确实是由于上面的推断的原因导致的。

###**本地验证**

**实验一：** 触发大量网络连接（每个连接处于独立的线程中）并保持，每打开一个 socket 都会增加一个 fd（/proc/pid/fd 下多一项）

**注：不只有这一种增加 fd 数的方式，也可以用其他方法，比如打开文件，创建 handlerthread 等等**

**实验预期：**当进程 fd 数（可以通过 ls /proc/pid/fd | wc -l 获得）突破 /proc/pid/limits 中规定的 Max open files 时，产生 OOM;

**实验结果：**当 fd 数目到达 /proc/pid/limits 中规定的 Max open files 时，继续开线程确实会导致 OOM 的产生。

**错误信息及堆栈如下：**

```
E/art: ashmem_create_region failed for 'indirect ref table': Too many open files
E/AndroidRuntime: FATAL EXCEPTION: main
                  Process: com.netease.demo.oom, PID: 2435
                  java.lang.OutOfMemoryError: Could not allocate JNI Env
                      at java.lang.Thread.nativeCreate(Native Method)
                      at java.lang.Thread.start(Thread.java:730)
                      ......
```
可以看出，此 OOM 发生时的错误信息确与线上发现的 OOM 一的“Could not allocate JNI Env” 吻合，**因此线上上报的 OOM 一 可能 就是由 FD 数超限导致的**，不过最终确定需要到线上进行验证 (下一小节). 此外从 ART 虚拟机的 Log 中看出，还有一个关键的信息 

```
“ art: ashmem_create_region failed for 'indirect ref table': Too many open files”，
```
后面会用于问题定位及解释。

**实验二：** 创建大量的空线程（不做任何事情，直接 sleep）

**实验预期：**

当线程数（可以在 /proc/pid/status 中的 threads 项实时查看）超过 /proc/sys/kernel/threads-max 中规定的上限时产生 OOM 崩溃。

**实验结果：**

在 Android7.0 及以上的华为手机（EmotionUI_5.0 及以上）的手机产生 OOM，这些手机的线程数限制都很小 (应该是华为 rom 特意修改的 limits)，每个进程只允许最大同时开 500 个线程，因此很容易复现了。
OOM 时错误信息如下：

```
W libc    : pthread_create failed: clone failed: Out of memory
W art     : Throwing OutOfMemoryError "pthread_create (1040KB stack) failed: Out of memory"
E AndroidRuntime: FATAL EXCEPTION: main
                  Process: com.netease.demo.oom, PID: 4973
                  java.lang.OutOfMemoryError: pthread_create (1040KB stack) failed: Out of memory
                      at java.lang.Thread.nativeCreate(Native Method)
                      at java.lang.Thread.start(Thread.java:745)
                      ......
```
可以看出 **错误信息与我们线上遇到的 OOM 二吻合："pthread_create (1040KB stack) failed: Out of memory" 另外 ART 虚拟机还有一个关键 Log：“pthread_create failed: clone failed: Out of memory”，**
后面会用于问题定位及解释。

1 . 其他 Rom 的手机线程数的上限都比较大，不容易复现上述问题。但是，**对于 32 位的系统，当进程的逻辑地址空间不够的时候也会产生 OOM, 每个线程通常需要 mapp 1MB 左右的 stack 空间（stack 大小可以自行设置）**，32 为系统进程逻辑地址 4GB，用户空间少于 3GB。逻辑地址空间不够（**已用逻辑空间地址可以查看 /proc/pid/status 中的 VmPeak/VmSize 记录**），此时创建线程产生的 OOM 具有如下信息：

```
W/libc: pthread_create failed: couldn't allocate 1069056-bytes mapped space: Out of memory
W/art: Throwing OutOfMemoryError "pthread_create (1040KB stack) failed: Try again"
E/AndroidRuntime: FATAL EXCEPTION: main
                  Process: com.netease.demo.oom, PID: 8638
                  java.lang.OutOfMemoryError: pthread_create (1040KB stack) failed: Try again
                       at java.lang.Thread.nativeCreate(Native Method)
                       at java.lang.Thread.start(Thread.java:1063)
                       ......
```

**线上验收及问题解决**

本地尝试复现的 OOM 错误信息中与线上 OOM 一情况比较吻合，与线上 OOM 二的情况比较吻合，但线上的 OOM 一真的时 FD 数目超限，OOM 二真的是由于华为手机线程数超限的原因导致的吗？最终确定还需要取线上设备的数据进行验证．

**验证方法：**

下发插件到线上用户，当 Thread.UncaughtExceptionHandler 捕获到 OutOfMemoryError 时记录 /proc/pid 目录下的如下信息：

* 1. /proc/pid/fd 目录下文件数 (fd 数)
* 2. /proc/pid/status 中 threads 项（当前线程数目）
* 3. OOM 的日志信息（出了堆栈信息还包含其他的一些 warning 信息

**线上 OOM 一验证**

发生 OOM 一的线上设备中采集到的信息：

* 1. /proc/pid/fd 目录下文件数与 /proc/pid/limits 中的 Max open files 数目持平，证明 FD 数目已经满了 ;
* 2. 崩溃时日志信息与图 [3-5] 基本一致

由此，证明 线上的 OOM 一确实是由于 FD 数目过多导致的 OOM，**推断验证成功．**

**OOM 一的定位与解决：**

最终原因是 App 中使用的长连接库再某些时候会有瞬时发出大量 http 请求的 bug(导致 FD 数激增)，已修复
线上 OOM 二验证 集中在华为系统的 OOM 二崩溃时收集到的信息样例如下，（**收集的样例中包含的 devicemodel 有 VKY-AL00，TRT-AL00A，BLN-AL20，BLN-AL10，DLI-AL10，TRT-TL10，WAS-AL00 等**）：

* 1. /proc/pid/status 中 threads 记录全部到达上限：Threads: 500

* 2. 崩溃时日志信息与图 [3-6] 基本一致
推断验证成功，即 线程数受限导致创建线程时 clone failed 导致了线上的 OOM 二。

**OOM 二的定位与解决：**
关于 App 业务代码中的问题还在定位修复中

###**结论**

**导致 OOM 发生的原因**

综上，可以导致 OOM 的原因有以下几种：

* 1. 文件描述符 (fd) 数目超限，即 proc/pid/fd 下文件数目突破 /proc/pid/limits 中的限制。可能的发生场景有：短时间内大量请求导致 socket 的 fd 数激增，大量（重复）打开文件等 ;
* 2. 线程数超限，即proc/pid/status中记录的线程数（threads 项）突破 /proc/sys/kernel/threads-max 中规定的最大线程数。可能的发生场景有：app 内多线程使用不合理，如多个不共享线程池的 OKhttpclient 等等 ;
* 3. 传统的 java 堆内存超限，即申请堆内存大小超过了 Runtime.getRuntime().maxMemory();
* 4. （低概率）32 为系统进程逻辑空间被占满导致 OOM;

* 5. 其他。
 
**监控措施**
 
**可以利用 linux 的 inotify 机制进行监控:**
watch /proc/pid/fd来监控 app 打开文件的情况,
watch /proc/pid/task来监控线程使用情况．

**Demo**

POC(Proof of concept) 代码参见：
https://github.com/piece-the-world/OOMDemo


