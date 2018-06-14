---
layout:     post
title:      MVC，MVP，MVVM
subtitle:   MVC，MVP，MVVM
date:       2018-03-16
author:     pxf
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Android
---
MVC，MVP，MVVM
===


[Android App的设计架构：MVC,MVP,MVVM与架构经验谈](http://blog.csdn.net/lovingkid/article/details/50917907)

### MVC 
View  xml
Controller Activity
Model 建立的数据机构和相关的类

1.View传达指令到Controller
2.Controller完成业务逻辑后，要求Model改变状态
3.Model将新的数据发送给View，更新视图，用户得到反馈 

MVC可分为2种 

1.通过View接受指令
2.通过Controller接受指令

缺点

在Android开发中，Activity并不是一个标准的MVC模式中的Controller，它的首要职责是加载应用的布局和初始化用户界面 ，并接受并处理来自用户的操作请求，进而作出响应。随着界面及其逻辑的复杂度不断提升， Activity类的职责不断增加，以致变得庞大臃肿。

### MVP
在App开发过程中，经常出现的问题就是某部分的代码过 ，虽然做模块划分和接隔离，但也很难完全避免。从实践中看到，这多的出现在UI部分，也就是Activity 。想象下， 个2000+ 以上基本 带注释的Activity，我的第 反应就是想吐。Activity内容过多的原因其实很好解释，因为Activity本身需要担负 与用户之间的操作交互，界面的展示， 是单纯的Controller或View。 且现在部分的Activity还对整个App 起到类似iOS中的【ViewController】的作 ，这又带入的逻辑代码，造成Activity的臃肿。为 解决这个问题，让我们引MVP框架。

将Controller改成了Presenter，同时改变了通信方向
1.Presenter和View是可以双向通信的
2.Presenter和Model也是可以双向通信的
3.View和Model之间不能通信，都是通过Presenter传递的
4.View和薄，没有任何业务逻辑，而Presenter很厚，所有的业务逻辑都在这处理

### MVVM

* 优点

将Presenter改成了Viewmodel，通信方式基本和MVP差不多，最大的区别是MVVM有双向绑定，View的变动反映在ViewModel上，ViewModel的修改也会更新View

* 缺点

View层的Activity通过DataBinding生成Binding实例,把这个实例传递给ViewModel，ViewModel层通过把自身与Binding实例绑定，从而实现View中layout与ViewModel的双向绑定。mvvm的缺点数据绑定使得 Bug 很难被调试。你看到界面异常了，有可能是你 View 的代码有 Bug，也可能是 Model 的代码有问题。数据绑定使得一个位置的 Bug 被快速传递到别的位置，要定位原始出问题的地方就变得不那么容易了。

