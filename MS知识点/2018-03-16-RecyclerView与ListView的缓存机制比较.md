---
layout:     post
title:      RecyclerView与ListView的缓存机制比较
subtitle:   RecyclerView与ListView的缓存机制比较
date:       2018-03-16
author:     pxf
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Android
---
RecyclerView与ListView的缓存机制比较
===

* 1、[Bugly-Android ListView 与 RecyclerView 对比浅析--缓存机制](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653578065&idx=2&sn=25e64a8bb7b5934cf0ce2e49549a80d6&chksm=84b3b156b3c43840061c28869671da915a25cf3be54891f040a3532e1bb17f9d32e244b79e3f&scene=21#wechat_redirect)

* 2、[RecyclerView 必知必会](https://mp.weixin.qq.com/s/CzrKotyupXbYY6EY2HP_dA)

## RecyclerView回收机制

RecyclerView和ListView的回收机制非常相似，但是ListView是以View作为单位进行回收，RecyclerView是以ViewHolder作为单位进行回收。Recycler是RecyclerView回收机制的实现类，他实现了四级缓存：

*   mAttachedScrap: 缓存在屏幕上的ViewHolder。

*   mCachedViews: 缓存屏幕外的ViewHolder，默认为2个。ListView对于屏幕外的缓存都会调用`getView()`。

*   mViewCacheExtensions: 需要用户定制，默认不实现。

*   mRecyclerPool: 缓存池，多个RecyclerView共用。

在上文Layout Manager中已经介绍了RecyclerView的layout过程，但是一笔带过了`getViewForPosition()`，因此此处介绍该方法的实现。

![](http://mmbiz.qpic.cn/mmbiz_png/tnZGrhTk4deAcQicPF0tHqZw2kX1iaHP9bY7lgSD8wkOK4yQ7Y47EXWcib7vaMDHJxUfxJa4euV39q7ibicCfy1Izpw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

从上述实现可以看出，依次从mAttachedScrap, mCachedViews, mViewCacheExtension, mRecyclerPool寻找可复用的ViewHolder，如果是从mAttachedScrap或mCachedViews中获取的ViewHolder，则不会调用`onBindViewHolder()`，mAttachedScrap和mCachedViews也就是我们所说的Scrap Heap；而如果从mViewCacheExtension或mRecyclerPool中获取的ViewHolder，则会调用`onBindViewHolder()`。

RecyclerView局部刷新的实现原理也是基于RecyclerView的回收机制，即能直接复用的ViewHolder就不调用`onBindViewHolder()`。

### 总结RecyclerView 缓存机制

RecyclerView 缓存机制（Recycler）大致可以分为5级。 
* 第一级 通过mChangedScrap匹配 position或者id获取holder缓存。 

* 第二级 从mAttachedScrap中通过匹配position获取holder缓存，或者通过ChildHelper找到隐藏但是没有被移除的View，通过getChildViewHolderInt(view)方法获取holder缓存，或者 
从mCachedViews中通过匹配position获取holder缓存。 

* 第三级 从mAttachedScrap中通过匹配id获取holder缓存，或者 
从mCachedViews中通过匹配id获取holder缓存。 

* 第四级 从ViewCacheExtension获取holder缓存。 

* 第五级 通过RecyclerView 的ViewHolder缓存池获取holder。

## ListView回收机制

[解析Android ListView工作原理及其缓存机制](http://blog.csdn.net/libmill/article/details/49644743)

**RecycleBin缓存机制**

ListView这么厉害的原因，其中一部分就是因为RecycleBin缓存机制。RecycleBin缓存机制是写在AbsListView的一个内部类。所以ListView继承于AbsListView，也继承了这股力量。让我们看看它的内部几个重要的变量和方法：

* 变量： 
1. `private View[] mActiveViews` : 缓存屏幕上可见的view 
2. `private int mViewTypeCount` : ListView中的子view的不同布局类型总数 
3. `ArrayList<View>[] mScrapViews` : ListView中所有的废弃缓存。注意，这是一个数组。在ListView中，每种childView布局类型都会单独启用一个RecycleBin缓存机制。所以数组中的每一项ArrayList都对应着一种childView布局类型的废弃缓存。 
4. `ArrayList<View> mCurrentScrap` : 当前childView布局类型下的废弃缓存。

* 方法： 
1. `void fillActiveViews ()` : 此方法会将ListView中的指定元素存储到mActiveViews数组当中。 
2. `View getActiveView ()` : 从mActiveViews中获取指定的元素。取出view后，在mActiveViews里的该指定位置将被置空。所以这个mActiveViews只能使用一次，并不能复用。 
3. `void addScrapView ()` : 将一个废弃的view进行缓存。如果childView的布局类型只有一项，就直接缓存到mCurrentScrap。如果多种布局，则从mScrapViews找到相对应的废弃缓存ArrayList并缓存view。 
4. `View getScrapView ()` : 从废弃缓存中取出一个View。同理，如果childView的布局类型只有一项，就直接从mCurrentScrap中取。如果多种布局，则从mScrapViews找到相对应的缓存ArrayList再取出view。 
5. `void setViewTypeCount ()` : 为mViewTypeCount设置childView布局类型总数，并为每种类型的childView单独启用一个RecycleBin缓存机制。

只是简单介绍了这几个重要的方法，可能大伙看得也是糊里糊涂的。或许有疑问，这个mActiveViews与mCurrentScrap有点相似，好像都是缓存view的。但它俩的区别在哪呢？莫急，等下会一一解答。

先来说说**RecycleBin缓存机制的工作原理**： 

ListView每当一项子view滑出界面时，RecycleBin会调用addScrapView()方法将这个废弃的子view进行缓存。每当子view滑入界面时，RecycleBin会调用getScrapView()方法获取一个废弃已缓存的view。所以我们再看回Adapter的`getView()`方法：

```java
@Override  
public View getView(int position, View convertView, ViewGroup parent) {   
    View view;  
    if (convertView == null) {  
        view = LayoutInflater.from(context).inflate(resourceId, null);  
        ······
    } else {  
        view = convertView;  
    }   
    ······
    return view;  
} 
```

这个convertView是什么？convertView就是RecycleBin缓存机制调用getScrapView()方法获取废弃已缓存的view。`getView()`方法中有个判断，`if (convertView == null)`，当convertView为空，也就是没有废弃已缓存的view时，将调用LayoutInflater的inflate()方法加载出来布局view，这个操作是比较耗时的；当convertView不为空时，我们就直接用convertView了，而不需要再次调用LayoutInflater的inflate()方法加载出来布局view。所以如果我们没利用convertView，成千上万条数据，所有的视图都是调用LayoutInflater的inflate()方法去创建布局view的话，想想就知道这样的ListView的性能是多么的糟糕了，ListView的强大就大大打折扣了，所谓的洪荒之力也就没有了。 



