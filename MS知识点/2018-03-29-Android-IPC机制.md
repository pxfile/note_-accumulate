Android-Binder机制
===
Binder是Android中一个类，实现了IBinder接口，在Linux中没有。

**Android framework角度来说**，Binder是ServiceManager连接各种Mananger（ActivityManager，WIndowManager等）和相应ManagerService的桥梁；

**Android应用层来说**，Binder是客户端和服务器进行通信的媒介，当bindService时，服务器会返回一个包含了服务端业务调用的Binder对象，通过这个对象客户端就可以湖区服务端提供的服务或者数据，这个服务包括普通服务和AIDL的服务。

用AIDL分析Binder机制，新建一个AIDL示例，SDK会自动为我们生产AIDL对应的Binder类。

# Bundle

四大组件支持使用intent传递bundle数据，bundle实现了parcelable接口，所以支持在不同进程间通信。

我们可以在bundle中附加我们需要的传输的信息并通过intent发送出去。发送的数据是基本数据类型或者是实现了Parcelable活serializable的对象。

# 文件共享
a进程把数据写入文件，b进程通过读取文件或者数据。可以传送文本和序列化对象到文件，另一个进程反序列化恢复对象。

* 写入文件
```
ObjectOutputStream object=new ObjectOutputStream(new FileOutputStream(cacheFile));

object.writeObject(user);

```
* 读取文件
```
ObjectInputStream object=new ObjectInputStream(FileInputStream(cacheFile));

User user=(User)object.readObject();
```
* 局限性
不支持并发读写，适合在对数据同步要求不高的进程间进行通信

* sharepreference
sharepreference内部有一定的缓存策略，内存中会有一份sharepreference文件的缓存，在多进程模式下，系统对它的读写变的不靠谱。高并发的读写会丢失数据，不建议在多进程通信使用。

# Messager
轻量级的IPC方案，底层实现是AIDL。Messager对AIDL做了封装，是的我们可以更简便的使用进程间通信。它一次处理一个请求，因此在服务端我们不考虑线程同步问题。messager不支持跨进程调用服务端的方法。

## 服务端进程

服务端创建一个Service来处理客户端的请求，创建一个handler用来创建Messager，在Service中的onBind中返回这个Messager对象底层的binder即可。

## 客户端进程

绑定服务端的Service，用Service中返回的IBinder对象创建一个Messager，通过这个Messager口可以给服务端发消息了，消息的类型是Message对象。

如果需要服务端能回应客户端，在客户端创建一个handler并创建一个新的Messager，并把这个Messager对象通过message的replyTo参数传递给服务端，服务端通过这个replyTo参数就可以回应客户端。

# AIDL

## 服务端
创建一个Service监听客户端的连接请求，创建一个AIDL文件，这个AIDL文件中是暴露给这个客户端的接口，创建一个类继承AIDL接口中的Stub类并实现Stub中的抽象方法，在service中onBind方法中返回这个类的对象。

## 客户端
绑定服务端的Service，绑定后将服务端返回的Binder对象转成AIDL接口所属的类型，然后调用AIDL中的方法。

## AIDL中支持的数据类型
基本数据类型，String和charSequence，List（ArrayList），hashmap，parcelable对象，AIDL文件

# ContentProvider
系统中的ContentProvider，通讯录，日程表信息等。

ContentProvider中的query，update，insert，delete运行在ContentProvider的进程即Binder线程池中，除了onCreate运行在主线程中。

# Binder池