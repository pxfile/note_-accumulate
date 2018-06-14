面试流程
===

一面/二面	Java	基础	(滴答)	
几个作用域的关系（private，public，protected，啥都不写）

 	 
一面/二面	Java	基础	 	interface 和abstract class区别	 	 
一面/二面	Java	基础	 	内部类与外部类的关系	 	 
一面/二面	Java	基础	 	多态	 	 
一面/二面	Java	基础	 	线程的生命周期？	 	 
一面/二面	Java	基础	 	
多线程、线程池的使用

 	 
一面/二面	Java	基础	(滴答)	synchronized在方法前和在类名前有什么区别？ 	 	 
一面/二面	Java	高级	(滴答)	
类加载过程

http://www.ibm.com/developerworks/cn/java/j-lo-classloader/

 	 
一面/二面	Java	高级	(滴答)	
注解，泛型，反射的使用

如何定义一个注解（http://www.cnblogs.com/jiyuqi/p/3678539.html）

反射的原理，Android中的使用场景

 	 
一面/二面	Java	高级	(滴答)	
泛型中的占位符  T和？有什么区别？

http://zhidao.baidu.com/question/576202129.html

 	 
一面/二面	Java	高级	 	
JVM（gc过程）

 	 
一面/二面	Java	高级	 	
NIO

 	 
一面/二面	Java	高级	(滴答)	强、弱、软引用	 	 
一面/二面	Java	高级	(滴答)	设计模式考察(IoC，guice注入框架)	 	 
一面/二面	Android	基础	 	
如何学习安卓的

 	 
一面/二面	Android	基础	(滴答)	是否看过google的开发者指南	 	 
一面/二面	Android	基础	 	
是否看过安卓源代码

（代码树上的源代码，sdk中的API源代码，APIDemo）

 	 
一面/二面	Android	基础	(滴答)	
Activity的生命周期

（基本的几个状态，下拉statusbar会触发哪些过程？）
不会对Activity的生命周期影响
弹出Dialog也是（亲测）

 	 
一面/二面	Android	基础	(滴答)	
 Fragment的生命周期，和Activity如何交互
（addFragment之后会走哪些过程呢？replaceFragment呢？）
[Android Fragment 真正的完全解析（上）](https://blog.csdn.net/lmj623565791/article/details/37970961)
 	 
一面/二面	Android	基础	 	 Service的两种启动方式，区别（start，bind）	 	 
一面/二面	Android	基础	 	 provider（写法，如何提供给外部使用，如何限制访问）	 	 
一面/二面	Android	基础	(滴答)	 Manifest里面的配置项考察 （选一两个就好了，例如广播优先级的问题）	 	 
一面/二面	Android	基础	(滴答)	 ListView如何优化（GDD2009）	 	 
一面/二面	Android	基础	(滴答)	 列表页的图片加载如何处理（ImagePool）	 	 
一面/二面	Android	基础	(滴答)	 9.png（上下左右加点的含义）	 	 
一面/二面	Android	基础	 	各种布局（考察一两个样子就好了）	 	 
一面/二面	Android	基础	 	
如何分析布局的优劣，如何优化布局（hierarchyviewer）

 	 
一面/二面	Android	高级	 	
Activity的TASK概念理解（四种launchMode，区别）

 	 
一面/二面	Android	高级	 	
aidl的使用

 	 
一面/二面	Android	高级	(滴答)	
AsyncTask的使用

 	 
一面/二面	Android	高级	(滴答)	
Handler的使用（屏幕灭了之后Handler是否还运行？）

* **一旦手机在休眠的时候，手机的cpu也休眠了，创建的线程也会sleep**，所以为了让手机屏幕黑屏之后，上传线程可以继续运行，就必须保存手机CPU一直处于运行状态，综合网上所查找的资料，发现可以通过使用android的[PowerManager和PowerManager.WakeLock](http://www.cnblogs.com/keyindex/archive/2010/09/06/1819504.html)这两个类来控制
 	 
一面/二面	Android	高级	(滴答)	
Loader的使用

Loader + fragment方式

CursorLoader + ContentProvider用法，

ContentResolver NotifyChange

 	 
一面/二面	Android	高级	(滴答)	
View的描绘原理，例如调用requestLayout之后走哪几个方法？
或者说requestLayout，postInvalidate，invalidate()的区别

* postInvalidate与invalidate方法的作用是一样的，都是使View树重绘，但两者的使用条件不同，postInvalidate是在非UI线程中调用，invalidate则是在UI线程中调用。

![View的描绘原理](http://upload-images.jianshu.io/upload_images/1734948-b4493f7b0234dd69.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 	 
一面/二面	Android	高级	(滴答)	viewGroup的考察(touch事件分发机制)	 	 
一面/二面	Android	高级	 	
View和SurfaceView的区别

*   View：显示视图，内置画布，提供图形绘制函数、触屏事件、按键事件函数等；**必须在UI主线程内更新画面，速度较慢**
*   SurfaceView：基于view视图进行拓展的视图类，更适合2D游戏的开发；**是view的子类，类似使用双缓机制，在新的线程中更新画面所以刷新界面速度比view快。**
*   GLSurfaceView：基于SurfaceView视图再次进行拓展的视图类，专用于3D游戏开发的视图；**是SurfaceView的子类，openGL专用。**

在2D游戏开发中，大致可以分为两种游戏框架，View和SurfaceView。 View和SurfaceView区别：

*   View：必须在UI的主线程中更新画面，用于被动更新画面。

*   surfaceView：UI线程和子线程中都可以。在一个新启动的线程中重新绘制画面，主动更新画面。

UI的主线程中更新画面 可能会引发问题，比如你更新画面的时间过长，那么你的主UI线程会被你正在画的函数阻塞。那么将无法响应按键，触屏等消息。当使用surfaceView 由于是在新的线程中更新画面所以不会阻塞你的UI主线程。但这也带来了另外一个问题，就是事件同步，涉及到线程同步。所以基于以上，根据游戏特点，一般分成两类。

*   1 被动更新画面的。比如棋类，这种用view就好了。因为画面的更新是依赖于 onTouch 来更新，可以直接使用 invalidate。 因为这种情况下，这一次Touch和下一次的Touch需要的时间比较长些，不会产生影响。
*   2 主动更新。比如一个人在一直跑动。这就需要一个单独的thread不停的重绘人的状态，避免阻塞main UI thread。所以显然view不合适，需要surfaceView来控制。

一般2D游戏开发使用SurfaceView足够，因为它也是google专们扩展用于2D游戏开发的画布
　　使用普通的游戏画布（Android中2D专用游戏画布）中进行绘制图片，然后在GLSurfaceView（Android中3D游戏专用画布）中渲染图片的对比中发现GLSurfaceView的效率高于SurfaceView的30倍；GLSurfaceView的效率主要是因为机器硬件的GPU加速，现在flash技术也有了GPU加速技术；
　　下面总结一下：

　　一般2D游戏使用SurfaceView足够，所以不要认为什么都要使用GLSurfaceView（openGL），而且 GLSurfaceView的弊端在于适配能力差，因为很多机型中是没有GPU加速的。
 	 
一面/二面	Android	高级	 	

**何为双缓冲，View里面如何实现双缓冲**

android中实现view的更新有两种方法,一个是invalidate(应用在UI线程中),另一个是postinvalidate(应用在非UI线程中)

双缓冲

闪烁是图形编辑中的一个常见问题,当进行复杂的绘制操作的时候会导致呈现的图像闪烁或具有其他不可接受的外观.双缓冲的使用解决这些问题.双缓冲使用内存缓冲区来解决由多重绘制操作造成的闪烁问题.当使用双缓冲时,首先在内存缓冲区里完成所有的绘制操作,而不是在屏幕上直接进行绘图,当所有绘制操作完成后,把内存缓冲区完成的图像直接复制到屏幕.因为在屏幕上只执行一个图形操作,所以消除了由复杂绘制操作所造成的图像闪烁问题.

在android中实现双缓冲,可以使用一个后台画布backcanvas,先把所有绘制操作都在这个上边进行,等图画好了,然后再把backcanvas拷贝到与屏幕关联的canvas上去,如下:
```

Bitmap bitmapBase = new Bitmap();

Canvas backcanvas = new Canvas(bitmapBase);

backcanvas.draw()...//画图

Canvas c = lockCanvas(null);

c.drawbitmap(bitmapBase);//把应经画好的图像输出到屏幕上

unlock(c)...

```	 

一面/二面	Android	高级	 	
**Bitmap的内存大小计算方法**

（像素 ＊ 每像素占用的内存  java  C各一份）

长*宽*每个像素所占的字节数
 	 

一面/二面	Android	高级	 	
**opengl es（基本概念（EGL，GLSurfaceView， Renderer），renderscript）**

 	 
一面/二面	Android	高级	 	
**图片操作相关（camera，投影矩阵变换）**

 	 
一面/二面	Android	高级	 	
**notify操作（4.1新特性，notify显示大图，多行文字）**
[Android Notification自定义通知样式你要知道的事](https://blog.csdn.net/u011200604/article/details/52470770)
 	 
一面/二面	Android	高级	 	音频视频耳机操作	 	 
一面/二面	Android	高级	(滴答)	 APK的打包过程（混淆，zipalign）	 	 
一面/二面	Android	高级	(滴答)	ProGuard的作用，基本语法
[Android proguard 详解](https://blog.csdn.net/dai_zhenliang/article/details/42423575)	 	 
一面/二面	Android	高级	 	JNI/NDK (Android.mk )	 	 
一面/二面	Android	高级	 	系统休眠机制，PowerManager的WakeLock的使用	 	 
一面/二面	Android	高级	 	
AlarmManager的使用（RTC、RTC_WAKEUP、ELAPSED_REALTIME、

ELAPSED_REALTIME_WAKEUP的区别；setInexactRepeating()方法的优点）

 	 
一面/二面	Android	高级	 	
渠道包的做成，参考美团的技术博客：

http://tech.meituan.com/mt-apk-packaging.html

 	 
一面/二面	Android	高级	 	
gradle做成适配包，参考美团的技术博客：

http://tech.meituan.com/mt-apk-adaptation.html

 	 
一面/二面	Android	高级	 	
自动化测试工具的使用

Robotium Instrument Monkeyrunner

 	 
一面/二面	Android	高级	 	内存泄露分析(MAT，DDMS)	 	 
一面/二面	DB	基础	(滴答)	
SQL文考察（where，order by，group by，having）

 	 
一面/二面	DB	基础	 	
sqlite数据中事务的实现机制

 	 
一面/二面	DB	高级	 	其他数据库是否使用过，例如mysql，oracle	 	 
一面/二面	DB	高级	(滴答)	是否有使用过ORM框架，如greendao之类	 	 
一面/二面	网络	基础	(滴答)	
HttpClient的使用场景（多个HTTP请求，复用连接池）

 	 
一面/二面	网络	基础	 	
HttpURLConnection的使用场景（轻量，单一）

 	 
一面/二面	网络	基础	 	
http协议(如何实现断点续传？考察range属性)

 	 
一面/二面	网络	基础	 	浏览器输入一个网址后发生了什么事情	 	 
一面/二面	网络	基础	 	
如何获取并设置代理（wap网络）

 	 
一面/二面	网络	高级	 	
是否有网络性能优化的经验(json, xml, gzip, google protocol buffer , Thrift, okhttp)

 	 
一面/二面	网络	高级	 	是否研究过okhttp的源码，https://github.com/square/okhttp/	 	 
一面/二面	网络	高级	 	是否了解移动网络特性（IM软件ack心跳包发送的规则）	 	 
二面	代码能力	基础	 	
写一段代码，例如用Java写个栈

 	 
一面/二面	业务知识	基础	 	
统计(业务统计怎么实现的，使用umeng？)

 	 
一面/二面	业务知识	基础	 	
定位(网络定位，gps定位，基站定位)

 	 
一面/二面	业务知识	基础	 	
网络模块使用的是自己写的还是开源的

 	 
一面/二面	业务知识	基础	 	
push长连接的实现

 	 
一面/二面	业务知识	基础	 	
xmpp协议

 	 
一面/二面	业务知识	基础	 	
地图SDK相关

 	 
一面/二面	业务知识	基础	 	
各种开源控件的使用（actionbarsherlock，下拉的list，

support-v4，volley，picasso，okhttp等等）

 	 
一面/二面	业务知识	基础	 	
开源框架的使用（roboguice，guice，spring for android，

greendao之类的）

 	 
一面/二面	业务知识	基础	(滴答)	
各种工具（gradle，maven，git，shell脚本）

 	 
一面/二面	算法	基础	 	 5个数，7次比较排序	 	 
三面	看过的书	基础	(滴答)	看过哪些书	 	 
三面	基础素质	基础	(滴答)	
主动性

 	 
三面	基础素质	基础	 	
是否乐于分享

 	 
三面	基础素质	基础	(滴答)	沟通表达能力	 	 
三面	基础素质	基础	(滴答)	
学习能力

 	 
三面	基础素质	基础	(滴答)	
处理问题的能力（例如问他做这些项目中遇到的难题，如何解决的）

 	 
三面	基础素质	基础	(滴答)	平时比较常访问的网站	 	 
三面	基础素质	基础	 	是否了解美团，关注过美团，使用过美团。	 	 
三面	基础素质	基础	(滴答)	考察是否具有产品sense