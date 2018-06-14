SharedPreferences 多进程解决方案
===

## SharedPreferences支持进程同步吗?怎么让它支持

### 1. SharedPreferences不支持进程同步

一个进程的情况，经常采用SharePreference来做，但是SharePreference不支持多进程，它基于单个文件的，默认是没有考虑同步互斥，而且，APP对SP对象做了缓存，不好互斥同步.

MODE_MULTI_PROCESS的作用是什么?

在getSharedPreferences的时候, 会强制让SP进行一次读取操作，从而保证数据是最新的. 但是若频繁多进程进行读写 . 若某个进程持有了一个外部sp对象, 那么不能保证数据是最新的. 因为刚刚被别的进程更新了.

### 2.考虑用ContentProvider来实现SharedPreferences的进程同步.

ContentProvider基于Binder，不存在进程间互斥问题，对于同步，也做了很好的封装，不需要开发者额外实现。 

另外ContentProvider的每次操作都会重新getSP. 保证了sp的一致性.

## SharedPreferences 多进程解决方案

* [SharedPreferences 多进程解决方案](https://juejin.im/entry/590833711b69e60058eb34b9)

* [Android SharedPreference 支持多进程](https://www.jianshu.com/p/875d13458538)