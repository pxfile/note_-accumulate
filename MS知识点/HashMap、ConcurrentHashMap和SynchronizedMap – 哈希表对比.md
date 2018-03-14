HashMap、ConcurrentHashMap和SynchronizedMap – 哈希表对比
===


在Java中，`HashMap`是一个非常有用的数据结构。几乎每一个Java应用都会使用到它。我之前的博文中有介绍过[如何实现一个线程安全的缓存](http://crunchify.com/implement-simple-threadsafe-cache-using-hashmap-without-using-synchronized-collection/)，在这个例子中，我就使用到了`HashMap`。然而，需要注意的是，**`HashMap`本身并不是一个线程安全的Collection类**。

## 常见问题

*   `ConcurrentHashMap`和`Collections.synchronizedMap(Map)`分别是什么？
*   `ConcurrentHashMap`和`Collections.synchronizedMap(Map)`在性能上有什么区别？
*   `ConcurrentHashMap`和`Collections.synchronizedMap(Map)`的优劣对比？
*   关于`HashMap`和`ConcurrentHashMap`的常见面试问题

在这篇文章中，将涵盖以上的问题，并解释我们应该如何将`HashMap`变得线程安全。

## 原因

Map是一个通过键值对的形势来存储元素的容器，其中，要求key保持唯一，并且每一个key唯一对应一个value。如果你在设计一个高度并行化的程序，并且需要在不同的线程中去读取或者修改一个Map对象，那么使用线程安全的Map则是一个理想的选择。最典型的一个例子就是生产者－消费者模式，生产者不断的修改Map而消费者同时也在读取Map中的值。

那么对于Map来说，什么是线程安全呢？简单的来说，就是当多个线程同时在使用一个Map的时候，至少有一个线程对Map的结构进行了修改，那么必须保证这个修改被立即同步到其他线程中去，避免其他线程获取到错误的值。

## 解决方案

有两种方法可以解决`HashMap`的线程安全问题：

*   Java的`Collections`库中的`synchronizedMap()`方法
*   使用`ConcurrentHashMap`

_译者注：其实还有第三种方法，使用`Hashtable`。不过`Hashtable`是Java 1.1提供的旧有类，从性能上和使用上都不如其他的替代类，因此已经不推荐使用_

```java
//Hashtable
Map<String, String> normalMap = new Hashtable<String, String>();

//synchronizedMap
synchronizedHashMap = Collections.synchronizedMap(new HashMap<String, String>());

//ConcurrentHashMap
concurrentHashMap = new ConcurrentHashMap<String, String>();12345678
```

### ConcurrentHashMap

*   当你程序需要高度的并行化的时候，你应该使用`ConcurrentHashMap`
*   尽管没有同步整个Map，但是它仍然是线程安全的
*   读操作非常快，而写操作则是通过加锁完成的
*   在对象层次上不存在锁（即不会阻塞线程）
*   锁的粒度设置的非常好，只对哈希表的某一个key加锁
*   `ConcurrentHashMap`不会抛出`ConcurrentModificationException`，即使一个线程在遍历的同时，另一个线程尝试进行修改。
*   `ConcurrentHashMap`会使用多个锁

### SynchronizedHashMap

*   会同步整个对象
*   每一次的读写操作都需要加锁
*   对整个对象加锁会极大降低性能
*   这相当于只允许同一时间内至多一个线程操作整个Map，而其他线程必须等待
*   它有可能造成资源冲突（某些线程等待较长时间）
*   `SynchronizedHashMap`会返回`Iterator`，当遍历时进行修改会抛出异常