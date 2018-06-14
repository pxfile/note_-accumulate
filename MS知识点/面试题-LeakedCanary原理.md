面试题-LeakedCanary原理
===
如果检测到有内存泄漏，手机桌面会多出一个图标，点进去查看可以看见泄漏信息。

![](https://img-blog.csdn.net/20171010200228346?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGlhb2hhbmx1bw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
图-1 内存泄漏示意图

## 检测原理

### 监听

1. `ActivityRefWatcher`中，注册Activity生命周期监听接口，当Activity onDestroy()被调用时，将当前Activity加入内存泄漏监听队列；

在Android中，当一个`Activity`走完`onDestroy`生命周期后，说明该页面已经被销毁了，应该被系统GC回收。通过`Application.registerActivityLifecycleCallbacks()`方法注册`Activity`生命周期的监听，每当一个`Activity`页面销毁时候，获取到这个`Activity`去检测这个`Activity`是否真的被系统GC。

### 检测

2. `RefWatcher`中，`watch`方法将对象用`WeakReference`引起来,监听对象是否内存泄漏

这里的原理是：`WeakReference`和`ReferenceQueue<Object>`配合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列

检测方法就很简单了，主动GC，触发`WeakReference`被GC，同时检测GC前后，ReferenceQueue是否包含被监听对象；如果不包含，则说明该对象没有被GC，一定存在到GC Roots的强引用链，也就是发生了泄漏。

当获取了待分析的对象后，需要确定这个对象是否产生了内存泄漏。

通过`WeakReference` + `ReferenceQueue`来判断对象是否被系统GC回收，WeakReference 创建时，可以传入一个 ReferenceQueue 对象。当被 WeakReference 引用的对象的生命周期结束，一旦被 GC 检查到，GC 将会把该对象添加到 ReferenceQueue 中，待ReferenceQueue处理。当 GC 过后对象一直不被加入 ReferenceQueue，它可能存在内存泄漏。

当我们初步确定待分析对象未被GC回收时候，手动触发GC，二次确认。

## 源码分析

检测过程主要分为两个部分

*   监听Activity销毁，并判断是否存在内存泄漏
*   找到内存泄漏的对象的引用路径

#### 判断是否存在内存泄漏

从上面Activity生命周期监听回调可以看见，每次当`Activity`执行完`onDestroy`生命周期，会调用`RefWatcher`去分析是否存在内存泄漏。

如果被系统回收了，直接就返回DONE；如果没有被系统回收，可能存在内存泄漏，为了保证结果的准确性，调用`gcTrigger.runGc();`，手动触发系统GC，然后再尝试移除待分析对象，如果还存在，说明存在内存泄漏。

确定有内存泄漏后，是发送了一个Intent，启动了IntentService服务，启动Service后，会在`onHandleIntent`分析，找到内存泄漏对象的引用关系。

### 找到引用关系
启动分析泄漏的服务后，分析最终得到结果这里用到了Square的另一个库haha，哈哈哈哈哈，名字真的就是叫这个，开源地址：[https://github.com/square/haha](https://github.com/square/haha)。

首先获取到内存中的heap堆快照，然后对快照做了去重处理，去除一些重复的强引用关系。拿到待分析的类，去快照里面找引用关系，并将结果返回。

服务拿到分析结果后，启动了另一个IntentService服务，服务启动后，判断是否需要将内存泄漏信息存到本地，如果需要就存到本地，然后设置消息通知的基本信息，通过`LeakCanaryInternals.showNotification`方法调用系统自身的通知栏通知，告诉用户应用有内存泄漏。

### 总结

LeakCanary对于内存泄漏的检测非常有效，但也并不是所有的内存泄漏都能检测出来。

*   无法检测出Service中的内存泄漏问题
*   如果最底层的MainActivity一直未走`onDestroy`生命周期(它在Activity栈的最底层)，无法检测出它的调用栈的内存泄漏。

