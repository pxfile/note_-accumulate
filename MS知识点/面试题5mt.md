面试题mt5
===


# Android引入广播机制的用意?

* a:从MVC的角度考虑(应用程序内) 

　其实回答这个问题的时候还可以这样问，android为什么要有那4大组件，现在的移动开发模型基本上也是照搬的web那一套MVC架构，只不过是改了点嫁妆而已。android的四大组件本质上就是为了实现移动或者说嵌入式设备上的MVC架构，它们之间有时候是一种相互依存的关系，有时候又是一种补充关系，**引入广播机制可以方便几大组件的信息和数据交互**。 

* b：程序间互通消息(例如在自己的应用程序内监听系统来电) 

* c：效率上(参考UDP的广播协议在局域网的方便性) 

* d：设计模式上(反转控制的一种应用，类似监听者模式)

# Android系统的架构

![Android系统的架构](http://my.csdn.net/uploads/201208/12/1344764997_8716.jpg)

# View, surfaceView, GLSurfaceView有什么区别。

view是最基础的，必须在UI主线程内更新画面，速度较慢。
SurfaceView 是view的子类，类似使用双缓机制，在新的线程中更新画面所以刷新界面速度比view快
GLSurfaceView 是SurfaceView的子类，opengl 专用的

# Manifest.xml文件中主要包括哪些信息？

manifest：根节点，描述了package中所有的内容。
uses-permission：请求你的package正常运作所需赋予的安全许可。
permission： 声明了安全许可来限制哪些程序能你package中的组件和功能。
instrumentation：声明了用来测试此package或其他package指令组件的代码。
application：包含package中application级别组件声明的根节点。
activity：Activity是用来与用户交互的主要工具。
receiver：IntentReceiver能使的application获得数据的改变或者发生的操作，即使它当前不在运行。
service：Service是能在后台运行任意时间的组件。
provider：ContentProvider是用来管理持久化数据并发布给其他应用程序使用的组件。

# 根据自己的理解描述下Android数字签名。

(1)所有的应用程序都必须有数字证书，Android系统不会安装一个没有数字证书的应用程序
(2)Android程序包使用的数字证书可以是自签名的，不需要一个权威的数字证书机构签名认证
(3)如果要正式发布一个Android程序，必须使用一个合适的私钥生成的数字证书来给程序签名，而不能使用adt插件或者ant工具生成的调试证书来发布。
(4)数字证书都是有有效期的，Android只是在应用程序安装的时候才会检查证书的有效期。如果程序已经安装在系统中，即使证书过期也不会影响程序的正常功能。

# Socket通信编程

## 客户端编程步骤：

1、 创建客户端套接字(指定服务器端IP地址与端口号)

2、 连接(Android 创建Socket时会自动连接)

3、 与服务器端进行通信

4、 关闭套接字

## 服务器端:

1.创建一个ServerSocket，用于监听客户端Socket的连接请求

2.采用循环不断接受来自客户端的请求

3.每当接受到客户端Socket的请求，服务器端也对应产生一个Socket