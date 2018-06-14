Android-EvenBus
===

#####  subscribe()方法的实现
![register](https://upload-images.jianshu.io/upload_images/1485091-8bf39ad48834f39c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

#### 事件分发过程源码分析
![post](https://upload-images.jianshu.io/upload_images/1485091-b7b63f83d65903d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

#### 解除注册源码分析
最终分别从`typesBySubscriber`和`subscriptions`里分别移除订阅者以及相关信息即可.

广泛的使用`EventBus`做消息的通知,可以说是目前消息通知里最好用
的项目.但是业内对`EventBus`的主要争论点是在于`EventBus`使用反射会出现性能问题,关于反射的性能问题可以参考[这篇文章](https://link.jianshu.com?t=http://www.cnblogs.com/zhishan/p/3195771.html),

实际上在`EventBus`里我们可以看到不仅可以使用注解处理器预处理获取订阅信息,`EventBus`也会将订阅者的方法缓存到`METHOD_CACHE`里避免重复查找,所以只有在最后
`invoke()`方法的时候会比直接调用多出一些性能损耗,但是这些对于我们移动端来说是完全可以忽略的.所以盲目的说因为性能问题而觉得`EventBus`不值得使用显然是不负责任的.


