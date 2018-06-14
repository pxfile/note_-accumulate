Android-Volley
===

[Android Volley完全解析(四)，带你从源码的角度理解Volley](https://blog.csdn.net/guolin_blog/article/details/17656437)

### Volley到底有哪些特点呢？

*   自动调度网络请求
*   多个并发的网络连接
*   通过使用标准的HTTP缓存机制保持磁盘和内存响应的一致
*   支持请求优先级
*   支持取消请求的强大API，可以取消单个请求或多个
*   易于定制
*   健壮性：便于正确的更新UI和获取数据
*   包含调试和追踪工具

### Volley优点

1.  可以取消请求
2.  容易扩展，面向接口编程
3.  网络请求线程NetworkDispatcher默认开启了4个，可以优化，通过手机CPU数量

### Volley缺点

1.  在BasicNetwork中判断了statusCode(statusCode < 200 || statusCode > 299)，如何符合条件直接抛出IOException()，不够合理

2.  导致401等其他状态抛出IOException 
    解决方案 : 
    [http://blog.csdn.net/kufeiyun/article/details/44646145](http://blog.csdn.net/kufeiyun/article/details/44646145) 
    [http://stackoverflow.com/questions/30476584/android-volley-strange-error-with-http-code-401-java-io-ioexception-no-authe](http://stackoverflow.com/questions/30476584/android-volley-strange-error-with-http-code-401-java-io-ioexception-no-authe)

3.  图片加载性能一般

![](https://img-blog.csdn.net/20170921155555120?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamlhbmtldWZv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

# 概念介绍

## 1.  对`Http`协议的抽象
    `Requeset`
    顾名思义，对请求的封装，实现了`Comparable`接口，因为在`Volley`中是可以指定请求的优先级的，实现`Comparable`是为了在`Request`任务队列中进行排序,优先级高的`Request`会被优先调度执行。
    `NetworkResponse`
    `Http`响应的封装，其中包括返回的状态码 头部 数据等。
    `Response`
    给调用者返回的结果封装，它比`NetworkResponse`更加简单，只包含三个东西：数据 异常 和 `Cache`数据。
    `Network`
    对`HttpClient`的抽象，接受一个`Request`，返回一个`NetworkResponse`

## 2.  反序列化抽象
    所谓反序列化，就是将网络中传输的对象变成一个`Java`对象，`Volley`中是通过扩展`Request`类来实现不同的反序列化功能，如`JsonRequest StringRequest`，我们也可以通过自己扩展一些`Request`子类，来实现对请求流的各种定制。

## 3.  请求工作流抽象
`RequestQueue`
用来管理各种请求队列，其中包含有4个队列
a) 所有请求集合，通过`RequestQueue.add()`添加的`Request`都会被添加进来，当请求结束之后删除。
b) 所有等待`Request`，这是`Volley`做的一点优化，想象一下，我们同时发出了三个一模一样的`Request`，此时底层其实不必真正走三个网络请求，而只需要走一个请求即可。所以`Request1`被`add`之后会被调度执行，而`Request2` 和`Request3`被加进来时，如果`Request1`还未执行完毕，那么`Request2`和 `Request3`只需要等着`Request1`的结果即可。
c) 缓存队列，其中的`Request`需要执行查找缓存的工作
d) 网络工作队列 其中的`Request`需要被执行网络请求的工作

`NetworkDispatcher`
执行网络`Request`的线程，它会从网络工作队列中取出一个请求，并执行。`Volley`默认有四个线程作为执行网络请求的线程。

`CacheDispatcher`
执行`Cache`查找的线程，它会从缓存队列中取出一个请求，然后查找该请求的本地缓存。`Volley`只有一个线程执行`Cache`任务。

`ResponseDelivery`
请求数据分发器，可以发布`Request`执行的结果。

`Cache`
对`Cache`的封装，主要定义了如何存储，获取缓存，存取依据`Request`中的`getCacheKey()`来描述。

# Volley执行流程

![](https://upload-images.jianshu.io/upload_images/699911-4f50fb7c44adb5ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/411)

## 步骤一，获取RequestQueue

Volley.newRequestQueue(context)方法来获取一个RequestQueue对象
```
public static RequestQueue newRequestQueue(Context context) {
    return newRequestQueue(context, null);
}

public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
    ....
}
```

如果HttpStack是等于null的，则去创建一个HttpStack对象，这里会判断如果手机系统版本号是大于9的，则创建一个HurlStack的实例，否则就创建一个HttpClientStack的实例。实际上HurlStack的内部就是使用HttpURLConnection进行网络通讯的，而HttpClientStack的内部则是使用HttpClient进行网络通讯的

创建好了HttpStack之后，接下来又创建了一个Network对象，它是用于根据传入的HttpStack对象来处理网络请求的，紧接着new出一个RequestQueue对象，并调用它的start()方法进行启动，然后将RequestQueue返回，这样newRequestQueue()的方法就执行结束了。

# 步骤二，创建CacheDispatcher，NetworkDispatcher

在start方法中创建了一个CacheDispatcher的实例，然后调用了它的start()方法，接着在一个for循环里去创建NetworkDispatcher的实例，并分别调用它们的start()方法。这里的CacheDispatcher和NetworkDispatcher都是继承自Thread的，而默认情况下for循环会执行四次，也就是说当调用了Volley.newRequestQueue(context)之后，就会有五个线程一直在后台运行，不断等待网络请求的到来，其中CacheDispatcher是缓存线程，NetworkDispatcher是网络请求线程。

# 步骤三，RequestQueue的add()方法将Request传入

得到了RequestQueue之后，我们只需要构建出相应的Request，然后调用RequestQueue的add()方法将Request传入就可以完成网络请求操作了

add()方法的内部判断当前的请求是否可以缓存，如果不能缓存则在第12行直接将这条请求加入网络请求队列，可以缓存的话则在第33行将这条请求加入缓存队列。在默认情况下，每条请求都是可以缓存的，当然我们也可以调用Request的setShouldCache(false)方法来改变这一默认行为。

那么既然默认每条请求都是可以缓存的，自然就被添加到了缓存队列中，于是一直在后台等待的缓存线程就要开始运行起来了，
看到一个while(true)循环，说明缓存线程始终是在运行的，接着在第23行会尝试从缓存当中取出响应结果，如何为空的话则把这条请求加入到网络请求队列中，如果不为空的话再判断该缓存是否已过期，如果已经过期了则同样把这条请求加入到网络请求队列中，否则就认为不需要重发网络请求，直接使用缓存中的数据即可。CacheDispatcher中的run()方法中在第39行调用Request的parseNetworkResponse()方法来对数据进行解析，再往后就是将解析出来的数据进行回调了，

#  步骤四，NetworkDispatcher处理网络请求

看一下NetworkDispatcher中是怎么处理网络请求队列的，在第7行我们看到了类似的while(true)循环，说明网络请求线程也是在不断运行的。在第28行的时候会调用Network的performRequest()方法来去发送网络请求，而Network是一个接口，这里具体的实现是BasicNetwork，我们来看下它的performRequest()方法中大多都是一些网络请求细节方面的东西，我们并不需要太多关心，需要注意的是在第14行调用了HttpStack的performRequest()方法，这里的HttpStack就是在一开始调用newRequestQueue()方法是创建的实例，默认情况下如果系统版本号大于9就创建的HurlStack对象，否则创建HttpClientStack对象。前面已经说过，这两个对象的内部实际就是分别使用HttpURLConnection和HttpClient来发送网络请求的，我们就不再跟进去阅读了，之后会将服务器返回的数据组装成一个NetworkResponse对象进行返回

在NetworkDispatcher中收到了NetworkResponse这个返回值后又会调用Request的parseNetworkResponse()方法来解析NetworkResponse中的数据，以及将数据写入到缓存，这个方法的实现是交给Request的子类来完成的，因为不同种类的Request解析的方式也肯定不同。还记得我们在上一篇文章中学习的自定义Request的方式吗？其中parseNetworkResponse()这个方法就是必须要重写的。

# 步骤五，返回数据到主线程

在解析完了NetworkResponse中的数据之后，又会调用ExecutorDelivery的postResponse()方法来回调解析出的数据，其中，在mResponsePoster的execute()方法中传入了一个ResponseDeliveryRunnable对象，就可以保证该对象中的run()方法就是在主线程当中运行的了

# 过期处理

硬过期 软过期
我们可以看到`Cache`中有两个字段来描述缓存过期： `Cache.ttl` vs `Cache.softTtl`。什么区别呢？如果`ttl`过期，那么这个缓存永远不会被使用了；如果`softTtl`没有过期，这个数据直接返回；如果`softTtl`过期，那么这次请求将有两次返回，第一次返回这个`Cahce`，第二次返回网络请求的结果。想想，这个是不是满足我们很多场景呢？先进入页面展示缓存，然后再刷新页面；如果这个缓存太久了，可以等待网络数据回来之后再展示数据，是不是很赞？


