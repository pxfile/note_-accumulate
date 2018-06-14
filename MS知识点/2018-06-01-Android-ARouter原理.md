Android-ARouter原理
===
[Alibaba-ARouter 源码分析笔记](https://www.jianshu.com/p/16e578a89555)

## 原生跳转和路由跳转的差异。

（1）显示跳转需要依赖于类，而路由跳转是通过url索引，无需依赖

（2）隐式是通过AndroidMainfest集中管理，协作开发困难，

（3）原生需要在AndroidMainfest里面注册，而路由是用注解来注册

（4）原生只要启动了startActivity就交由Android控制，而路由是使用AOP切面编程可以作控制

## 优点

* **更简便灵活的配置方式** 
* **可控制的跳转过程（拦截器）** 
* **可监听/处理跳转结果**

## ARouter 编译过程分析

首先我们知道 ARouter 的自动注册的实现是利用了编译期自定义注解的处理来完成的。我们先来看看 ARouter 定义了哪些自定义注解（此部分源码位于 [arouter-annotation](https://link.jianshu.com?t=https://github.com/alibaba/ARouter/tree/master/arouter-annotation)）：
1.`@Route` 是 ARouter 最重要的注解，也是路由最基本的节点，使用该注解标注的类将被自动添加至路由表中。


