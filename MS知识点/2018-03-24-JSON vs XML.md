---
layout:     post
title:      Android JSON vs XML
date:       2018-03-24
author:     pxf
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Android
---
JSON vs XML
===

# JSON vs XML 优缺点

## 1）JSON定义

JSON是一种轻量级的数据交互格式，具有良好的可读性和快速编写的特性。业内主流技术为其提供了完整的解决方案（有点类似于正则表达式，获得了当今大部分语言的支持），从而可以在不同平台间进行数据交互。JSON采用兼容性很高的文本格式，同时也具备类似于C语言体系的行为。

## 2）XML定义

XML表示扩展标记语言 (Extensible Markup Language,XML) ，用于标记电子文件使其具有结构性的标记语言，可以用来标记数据、定义数据类型，是一种允许用户对自己的标记语言进行定义的源语言。XML是标准通用标记语言 (SGML) 的子集，非常适合 Web 传输。

## 3）XML和JSON的优缺点对比

### 可读性方面

JSON和XML的数据可读性基本相同，JSON和XML的可读性可谓不相上下，一边是建议的语法，一边是规范的标签形式，XML可读性较好些。

### 可扩展性方面

XML天生有很好的扩展性，JSON当然也有，没有什么是XML能扩展，JSON不能的。

### 编码难度方面

XML有丰富的编码工具，比如Dom4j、JDom等，JSON也有json.org提供的工具，但是JSON的编码明显比XML容易许多，即使不借助工具也能写出JSON的代码，可是要写好XML就不太容易了。

### 数据体积方面

JSON相对于XML来讲，数据的体积小，传递的速度更快些。

### 数据交互方面

JSON与JavaScript的交互更加方便，更容易解析处理，更好的数据交互。

### 传输速度方面

JSON的传输速度要远远快于XML。

# 对比

## JSON相比XML的不同之处

*   没有结束标签
*   更短
*   读写的速度更快
*   能够使用内建的 JavaScript eval() 方法进行解析
*   使用数组
*   不使用保留字

总之： JSON 比 XML 更小、更快，更易解析。

## XML和JSON的区别：

### 组成部分

* XML的主要组成成分：

XML是element、attribute和element content。

* JSON的主要组成成分：

JSON是object、array、string、number、boolean(true/false)和null。


XML要表示一个object(指name-value pair的集合)，最初可能会使用element作为object，每个key-value pair 用 attribute 表示：

1.  <student  name="soゝso"  age="27"/>

但如个某个 value 也是 object，那么就不可以当作attribute:

```
<student name="soゝso" age="27">
    <address>
        <country>中国</country>
        <province>北京市</province>
        <city>朝阳区</city>
        <district>北京市朝阳区东四环远洋国际中心A座1906 </district>
    </address>
</student>
```

那么，什么时候用element，什么时候用attribute，就已经是一个问题了。

而JSON因为有object这种类型，可以自然地映射，不需考虑上述的问题，自然地得到以下的格式。

```
{
    "name": "John",
    "age" : 10,
    "address" : {
        "country" : "中国",
        "province" : "北京市",
        "city" : "朝阳区",
        "district" : "北京市朝阳区东四环远洋国际中心A座1906",
    }
}
```
## `One More Thing…`

XML需要选择怎么处理element content的换行，而JSON string则不须作这个选择。

XML只有文字，没有预设的数字格式，而JSON则有明确的number格式，这样在locale上也安全。

XML映射数组没大问题，就是数组元素tag比较重复冗余。JSON 比较易读。

JSON的true/false/null也能容易统一至一般编程语言的对应语义。

XML文档可以附上DTD、Schema，还有一堆的诸如XPath之类规范，使用自定义XML元素或属性，能很方便地给数据附加各种约束条件和关联额外信息，从数据表达能力上看，XML强于Json，但是很多场景并不需要这么复杂的重量级的东西，轻便灵活的Json就显得很受欢迎了。

打个比方，如果完成某件事有两种方式：一种简单的，一个复杂的。你选哪个？

JSON与XML相比就是这样的。

# Android中解析XML格式数据的方法 
[Android中解析XML格式数据的方法](http://www.cnblogs.com/neillee/p/7281687.html)


[Android XML数据解析](http://www.runoob.com/w3cnote/android-tutorial-xml.html)

## 详解

### SAX

`SAX(Simple API for XML)` 使用流式处理的方式，它并不记录所读内容的相关信息。

它是一种以事件为驱动的XML API，解析速度快，占用内存少。使用回调函数来实现。

缺点是不能倒退。

### DOM

`DOM(Document Object Model)` 是一种用于XML文档的对象模型，可用于直接访问 XML 文档的各个部分。

它是一次性全部将内容加载在内存中，生成一个树状结构,它没有涉及回调和复杂的状态管理。

缺点是加载大文档时效率低下。

### PULL

`Pull` 内置于 Android 系统中。也是官方解析布局文件所使用的方式。

Pull 与 SAX 有点类似，都提供了类似的事件，如开始元素和结束元素。

不同的是，SAX 的事件驱动是回调相应方法，需要提供回调的方法，而后在 SAX 内部自动调用相应的方法。

而Pull解析器并没有强制要求提供触发的方法。因为他触发的事件不是一个方法，而是一个数字。它使用方便，效率高。

## 三、比较

SAX、DOM、Pull 的比较:

内存占用：SAX、Pull比DOM要好；

编程方式：SAX 采用事件驱动，在相应事件触发的时候，会调用用户编好的方法，也即每解析一类 XML，就要编写一个新的适合该类XML的处理类。DOM 是 W3C 的规范，Pull 简洁。

访问与修改：SAX 采用流式解析，DOM 随机访问。

访问方式：SAX，Pull 解析的方式是同步的，DOM 逐字逐句。
