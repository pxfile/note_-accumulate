# **电话面试题**

 1.ArrayList 和 Hashmap 简单说一些,区别,底层的数据结构.

**ArrayList**属于List集合，有序的，元素可重复，有索引，属于Collection单列集合，一次存一个元素

**HashMap**属于Map集合，双列集合，一次存一对集合，两个元素（对象）存在着映射关系

**ArrayList**：底层是数组结构，查询快，增删慢，不同步。

**HashMap**：底层数据结构是哈希表；允许使用null键和null值，不同步，效率高

2.Handler 消息机制

* 首先Looper.prepare()会在当前线程保存一个looper对象，并且会维护一个消息队列messageQueue，而且规定了messageQueue在每个线程中只会存在唯一的一个

* Looper.loop()方法会使线程进入一个无限循环，不断地从消息队列中获取消息，之后回调msg.target.disPatchMessage方法。

* 我们在实例化handler的过程中，会先得到当前所在线程的looper对象，之后得到与该looper对象相对应的消息队列。（代码见handler的构造方法）

* 当我们发送消息的时候，即handler.sendMessage或者handler.post，会将msg中的target赋值为handler自身，之后加入到消息队列中。

* 在第三步实现实例化handler的过程中，我们一般会重写handlerMessage方法（使用post方法需要实现run方法），而这些方法将会在第二步中的msg.target.disPatchMessage方法中被回调，从而实现了message从一个线程到另外一个线程的传递。

[**Handler机制？ MessageQueue，Looper等**](https://github.com/pxfile/note_accumulate/blob/master/Interview-Q-A/Handler%E6%9C%BA%E5%88%B6%EF%BC%9F%20MessageQueue%EF%BC%8CLooper%E7%AD%89/Handler%E6%9C%BA%E5%88%B6%EF%BC%9F%20MessageQueue%EF%BC%8CLooper%E7%AD%89.md)

3.引起内存泄漏的场景

4.多线程的使用场景?

5.常用的线程池有哪几种?

6.在公司做了什么?团队规模?为什么离职?

# **面试中实际涉及到的问题**

## **第一轮**

1.知道哪些单例模式,写一个线程安全的单例,并分析为什么是线程安全的?
```java
public class Singleton {
	// Private constructor prevents instantiation from other classes
	private Singleton() { }

	/**
	* SingletonHolder is loaded on the first execution of Singleton.getInstance() 
	* or the first access to SingletonHolder.INSTANCE, not before.
	*/
	private static class SingletonHolder { 
			public static final Singleton INSTANCE = new Singleton();
	}

	public static Singleton getInstance() {
			return SingletonHolder.INSTANCE;
	}
}
```
2.Java中的集合有哪些?解释一下HashMap?底部的数据结构?散列表冲突的处理方法,散列表是一个**HashMap**的数据结构，HashMap是采用**链表**处理冲突的

3.解释一下什么是MVP架构,画出图解,一句话解释MVP和MVC的区别?

4.Handle消息机制?在使用Handler的时候要注意哪些东西,是否会引起内存泄漏?画一下Handler机制的图解?

5.是否做过性能优化?已经采取了哪些措施进行优化?

6.引起内存泄漏的原因是什么?以及你是怎么解决的?

这些问题应该都是比较基础的问题,每个开发者都应该是非常熟悉并能详细叙述的.这一轮的面试官问的技术都是平时用到的.

## **第二轮**

1.关于并发理解多少?说几个并发的集合?

2.Handler 消息机制图解?

3.在项目中做了哪些东西?

4.画图说明View 事件传递机制?并举一个例子阐述

5.类加载机制,如何换肤,换肤插件中存在的问题?hotfix是否用过,原理是否了解?

6.说说项目中用到了哪些设计模式,说了一下策略模式和观察者模式?

7.会JS么?有Hybid开发经验么?

8.说一下快排的思想?手写代码

9.堆有哪些数据结构?

对于这轮米那是明显感觉到压力,知识的纵向了解也比较深,应该是个leader.

## **第三轮**

1.介绍一下在项目中的角色?

2.遇到困难是怎么解决的?

3.如何与人相处,与别人意见相左的时候是怎么解决的,并举生活中的一个例子.

由于每个人的立场、观点、阅历经验等不同，面对同一问题同一情况存在不同观点、意见相左的情况是很正常的情况
首先 我会保持冷静理智的心态，绝不会因为别人的意见与自己相左就有情绪、甚至抱怨，而是会多从别人的角度来考虑问题，尽量找出彼此之间的分歧，如果出现确实不能协调的情况，也应以大局为重，一定做到对事不对人；同时也要多做自我总结、分析，看看是否是自己的问题呢，毕竟人无完人，不能一出现冲突就总想着是别人错了，而是应该多从自身角度寻找原因。
其次 我会尝试从多种方式在适当的时候与对方加强沟通，只有以诚恳的态度通过沟通、交流，才能增进彼此的理解，减少彼此之间的猜疑、对立，才能使工作顺利开展下去；
最后 我认为最重要的就是一定要多听别人的意见，领导的意见、同事的意见、无论对方是否支持自己的观点，都应广泛倾听，正所谓兼听则明、偏信则暗，也只有如此，才能真正形成良好的人际氛围，利于工作的开展。

4.有没有压力特别大的时候?

这个应该是项目经理了,问的问题偏向于生活性格方面.

## **其他技术问题总结**

**必问点**：Handler，多线程，http 与 https 的区别，事件分发与冲突，内存优化，内存泄漏等。

**选问点**：设计模式，Fragment 与 Activity之间的通信，自定义 View，组件化架构等。

几乎不怎么问的：MVP，屏幕适配，动画等。

以后的一些面试方向：网络原理（五层模型的实现等），热更新，增量更新，JNI NDK（硬件公司重点之重点），跨进程通信（浅：广播，AIDL 。深：Binder 原理），framework 等。

大公司一般不会问你很花哨的东西，原理，基础理论居多。小公司会问一些稍微应用层多一些的东西，例如 recyclerview 或者是瀑布流什么的。但是作为开发者来说，不断的学习，重复，记在脑子里比什么都重要。

## **找工作的过程就是解决一个相对复杂问题的过程。可以按照以下四部进行准备**：

1.  定义问题

2.  划分问题

3.  逐个突破

4.  系统化

接下来一步步的看一下具体细节。

### 1 定义问题

首先，定义一下我们解决的是什么问题。在这里，因为我们是Android方向，所以可以简单定义为：“我们要找到一个Android方向的工作（或相关的工作），工作要尽量好”。

> 这个定义很模糊，什么是尽量的好呢？有的人看中薪资，有的人看中五险一金，各种福利等等。在这里，我们不考虑个人主观因素占比较大的问题。我们只考虑更加可控的东西。就是通过个人努力可以获得效果的问题。

我们再思考一下“找到一个Android方向的工作”起决定性的因素是哪一个呢？

答案是面试。当然一个人过去做过的项目，拿过的奖也至关重要。但是到了这个马上就要面试的时间节点，过去的已经过去，无法改变，能控制的只有现在。没有项目无关紧要，关键的是现在如何准备面试。

目标：我要通过面试，拿到offer（或者我要通过多家公司面试，拿到多家公司的offer，选择最合心意的公司去工作）。当然，这句话表达的太宽泛，并没有什么指导意义。定义问题很重要，而更重要的是如何划分问题，这一步才是具有指导意义，能够落到实践中去的内容。

### 2 划分问题

Android面试需要准备内容的大致划分：（括号内为重要程度，最多5颗星）

*   Android相关知识、Java相关知识、设计模式（5）

*   算法、数据结构（5）

*   如何写简历、如何面试（4）

*   项目、比赛获奖（4）

*   操作系统、网络、数据库（3）

#### 细分

以下细分内容，网络等计算机基础方面还不是很全面，持续更新中。

我会逐步更新各个知识点相关博客或资源，如果需要，建议关注。

# Android

##   Context的理解

#### 定义

* * *

Context通常被翻译成上下文，它在Android中代表场景。一个Context就是一个场景，一个场景代表着一组用户与应用交互的过程。比如听音乐、打电话、发短信，这都分别对应着一个场景。

#### 实现

* * *

Context是一个抽象类，定义了访问应用环境全局信息的接口；包括访问应用程序资源、打开Activity(startActivity())、启动Service(startService())、发送广播，文件读写等。Activity，Service以及Application均继承自Context，所以我们经常使用的startActivity()，getResource()，getSharedPreference()，getExternalFilesDir()，deleteDatabase()等等方法都来自于Context。

Activity，Service，Application并没有直接继承Context，而是继承自ContextWrapper。ContextWrapper是Context的包装类，内部包含一个Context的引用，指向Context的具体实现类ContextImpl。ContextWrapper内部的所有方法直接调用ContextImpl对应的方法。

ContextImpl在ActivityThread创建Activity，Service以及Application时被实例化。所以每个应用都有多个ContextImpl实例。ContextImpl内部包含一个指向ActivityThread.PackageInfo的引用mPackageInfo。ContextImpl内部方法功能都是通过调用mPackageInfo对象来实现的。

ActivityThread.PackageInfo是一个重量级类，每一个应用程序仅有一个ActivityThread.PackageInfo实例，所有ContextImpl内部的mPackageInfo都指向ActivityThread.PackageInfo的唯一实例。

#### 使用区别

* * *

Application、Activity以及Service均继承自Context，在初始化ContextImpl时使用的参数不一样，所以对应的使用场景也不一样。

##   [Activity生命周期、启动模式、IntentFilter匹配规则](http://www.jianshu.com/p/3b675e85f78a)

##   IPC：Serialzable、Parcelable、Binder、Socket

##   View事件体系

*   View绘制流程

*   RemoteViews（不重要）

*   Drawable（不重要）

##   动画、绘图

##   window、wm、wms

##   四大组件启动、工作流程（Activity至少看一下，AMS）

##   消息机制：looper、handler、MQ

##   线程、线程池、多线程

##   bitmap加载、缓存：LRUCache、DiskLruCache、LinkHashMap

##   CrashHandler（一般）

当crash发生时，系统会回调UncaughtExceptionHandler

应用发生Crash在所难免，但是如何采集crash信息以供后续开发处理这类问题呢？利用Thread类的uncaughtException 方法，在UNcaughtException方法中获取异常信息，在合适的时机将异常信息存在sdk卡中或者上传到服务器。

开发人员分析异常信息然后修复。同时当发生crash时弹出一个对话框提示用户应用crash，然后再推出，这样用户体验更好。

`setDefaultUncaughtExceptionHandler`方法！`defaultUncaughtHandler`是Thread类的静态成员变量，所以如果我们将自定义的`UncaughtExceptionHandler`设置给Thread的话，那么当前进程内的所有线程都能使用这个UncaughtExceptionHandler来处理异常了。

```
public static void setDefaultUncaughtExceptionHandler(UncaughtExceptionHandler handler) {
    Thread.defaultUncaughtHandler = handler;
}

```

(2)作者实现了一个简易版本的UncaughtExceptionHandler类的子类`CrashHandler`，[源码传送门](https://github.com/singwhatiwanna/android-art-res/blob/master/Chapter_13/CrashTest/src/com/ryg/crashtest/CrashHandler.java)
CrashHandler的使用方式就是在Application的`onCreate`方法中设置一下即可

```
//在这里为应用设置异常处理程序，然后我们的程序才能捕获未处理的异常
CrashHandler crashHandler = CrashHandler.getInstance();
crashHandler.init(this);
```

##   multidex（一般）

##   Fragment、Service、SQLite、Webview

##   [Android内存泄漏场景及解决方法](http://www.jianshu.com/p/f35ca324c285)

##   ANR的原因、解决方法

##   开源库（一般要求看过源码，知道原理）：Retrofit、RxAndroid、EventBus、Picasso（优点）、OKhttp3

##   持续集成Jenkins（不重要）

1.  一个自动构建过程，包括自动编译、分发、部署和测试等。
2.  一个代码存储库，即需要版本控制软件来保障代码的可维护性，同时作为构建过程的素材库。
3.  一个持续集成服务器。本文中介绍的 Jenkins 就是一个配置简单和使用方便的持续集成服务器。

##   单元测试、测试用例（一般）

##   插件化：Atlas、OSGI（一般）

# Java

*   Java基础：比如接口和抽象类的区别等

*   Java内存管理：工作内存和主内存等

*   垃圾回收：回收算法、如何判断对象可以回收、新生代老年代等

*   并发

    锁：sychronized、lock（CAS）；volatile；并发集合：CopyOnWriteArrayList、ConcurrentHashMap、RemoteCallbackList（Android的IPC用到）、LinkedHashMap；

*   集合

    Map、Set、List

    Queue、Stack

    HashMap、HashTable、ConcurrentHashMap：实现原理，区别等

    LinkedHashMap

#### 设计模式（六大原则：SOLID + 迪米特）

*   单例模式：获取各种service

*   工厂方法：activity、service（onStart）

*   责任链：Android事件分发

*   builder：dialog、Picasso

*   观察者：listview更新、EventBus

*   适配器：listview adapter

#### 算法、数据结构

排序

*   冒泡排序

*   选择排序

*   归并

*   堆排序

*   插入排序

*   快速排序

*   希尔排序

*   桶排序

*   基数排序

字符匹配：KMP算法

二分查找

二叉树遍历、翻转、重构；二叉查找树

红黑树

AVL树、哈夫曼树、B树（一般）

#### 网络

[已整理博客，点击查看网络相关问题及其解答](http://www.jianshu.com/p/97f77927db0f)

基本是围绕OSI七层模型展开，首先是各层的功能、每层有哪些协议。

深入主要考察应用层和传输层：

应用层：

*   HTTP报文格式、头部有哪些字段

*   HTTP状态码

*   HTTP和HTTPS的区别

*   HTTPS中SSL/TLS加密的握手过程

*   HTTP一次连接的具体过程

*   GET、POST的区别

*   DNS解析过程

*   Cookie、Session原理

传输层：

*   TCP/IP四层模型（和OSI的层次对应关系）

*   TCP三次握手、四次握手的过程，状态变化和原因

*   TCP、UDP区别

*   TCP拥控、流控原理

*   Socket原理

#### 操作系统、数据库

线程状态及其切换

线程、进程区别

（数据库重要程度相对低一些，正在整理中，后续会更新）

#### 简历、面试、项目

篇幅较大，会有另外博客进行探讨，敬请关注

### 3 逐个突破

可以自己去网上找一些博客、书籍，进行各个知识点的突破，要有耐心，找到一个心仪的工作非一日之功。

一方面，我会陆续更新一些专业知识和面试相关的博客。

另一方面，把我自己的一些资源分享给大家。

*   博客

    GitYuan（gityuan.com）、罗升阳（CSDN）、邓凡平（CSDN）、任玉刚（CSDN）

*   书籍

    Android 4高级编程、Android开发艺术探索、Android源码设计模式、Android 50 hacks、Android应用性能优化最佳实践、Efficient Java、深入Java虚拟机、Java并发编程、Think in Java

*   刷题

    牛客网、LeetCode

### 4 系统化

系统化其实就是当你把一整个相关的知识都看过看懂之后，进行总结和建立各模块之间关系的过程。

每个人大脑“操作系统”是由概念和概念之间的联系的过程。系统化一方面可以加深知识的记忆，另一方面提供了另一个角度去理解这些概念，加大了概念的深入程度。

建议多做记录、总结，多在各模块、各学科之间建立联系，抽取统一适用的知识和智慧。

## **面试官：说说Java的HashMap吧。**

我：首先HashMap是由数组+链表的方式实现的，是无序，非线程安全的数据结构。

如果想使用有序的Map，可以使用TreeMap跟LinkedHashMap，其中TreeMap使用红黑树实现，排序规则由重写compare方法决定。LinkedHashMap则是维护了一个链表，根据构造参数选择在put或者get操作的时候进行顺序调整。

如果想使用线程安全的Map，可以使用HashTable跟ConcurrentHashMap，由于HashTable性能较低，重点说下ConcurrentHashMap。ConcurrentHashMap在Java7跟Java8中实现方式有较大不同。

Java7三要素：Segment，Lock，HashEntry。

Java8三要素：Node，CAS，Synchronized。

