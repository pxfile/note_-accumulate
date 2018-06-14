Binder机制
===
# Binder
直观来说，Binder是Android中的一个类，实现了IBinder接口。从IPC角度来说，Binder是Android中的一种跨进程通信的方式。从Android Framework角度老说，Binder是ServiceManager连接各种Manager（ActivityManager，WindowManager等）和相应ManagerService的桥梁。从Android应用层来说，Binder是客户端和服务器进行通信的媒介。

# 多进程的场景

 **应用需要采用多进程的模式**

* 模块需要运行在单独的进程中
* 为了增加应用使用的内存空间，需要多进程来获取多份内存空间（单个应用最大内存做了限制）

**使用多进程的方法**

* 在4大组件（Activity，Service，Reciver,ContentProvider)在清单文件中指定android:process属性
除此之外没有别的方法
* 非常规的方法，通过JNI在native层去fork一个新进程（但这属于特殊方式，不是常用的创建多进程的方式）

```
(1)
<activity
...
android:process=":remote"/>

(2)
<activity
...
android:process="com.ryg.chapter_2.remote"/>
```
(1)“：”指的是当前进程名前夫家上当前的包名，属于当前应用的私有进程，其他应用的组件不能和他跑在同一个进程中

(2)属于全局进程，可以通过ShareUID和它跑在同一个进程

**多进程会造成以下几方面的问题**

* 静态成员和单例模式完全失效
* 线程同步机制完全失效
* SharedPreferences的可靠性下降
* Application会多次创建

# IPC概念
**Parcelable和Serializable**

Serializable是Java中的序列化接口，使用简单，开销大
Parcelable使用麻烦，效率高（Android推荐）

* 注 通过Parcelable将对象序列化到存储设备中或者将对象序列化后通过网络传输也都可以，但是过程会复杂，在这两种情况下建议使用Serializable

# Android中的IPC方式
* **使用Bundle**

	* 适用于简单的进程通信	

* **使用文件共享** 
 
	 * 适合在对数据同步要求不高的进程之间通信，并且要妥善处理并发读写的问题 
	
	 * SharedPreference文件是一个特例，由于系统对他的读写有一定的缓存策略，即在内存中会有一份SharedPreference文件的缓存，因此在多进程的模式下，系统对他的读写就变的不可靠，当面对高并发的读写访问时很大几率会丢失数据，因此不建议早进程通信中使用SharePreference。

*  **使用Messager**
	
	* Messager是一种轻量级的IPC方案，它的低层实现是AIDL
	
	* Messager以串行方式处理客户端发来的消息，不适合并发请求
	
	* Messager作用主要是为了传递消息，它无法做到跨进程调用服务端的方法

*  **使用AIDL**

	*

*  **使用ContentProvider**

	* ContentProvider底层实现也是Binder

    * 使用过程比AIDL简单很多，因为系统为我们做了封装，我们无需关心底层细节即可轻松实现IPC
*   **使用Socket**

     * 远程Service建立一个TCP服务，然后在主界面中连接TCP服务，连接上了以后就可以给服务端发消息了