JAVA/Android 多线程实现方式及并发与同步
===

# 概述

说到线程，就不得不先说线程和进程的关系，这里先简单解释一下，进程是系统的执行单位，一般一个应用程序即是一个进程，程序启动时系统默认有一个主线程，即是UI线程，我们知道不能做耗时任务，否则ANR程序无响应。这时需要借助子线程实现，即多线程。由于线程是系统CPU的最小单位，用多线程其实就是为了更好的利用cpu的资源。

## 多线程实现方式

### 1、继承Thread类，重写run函数方法：
```
class xx extends Thread{
    public void run(){
        Thread.sleep(1000);    //线程休眠1000毫秒，sleep使线程进入Block状态，并释放资源
    }
}
xx.start();    //启动线程，run函数运行
```
### 2、实现Runnable接口，重写run函数方法：
```
Runnable run =new Runnable() {
    @Override
    public void run() {
        
    }
}
```
### 3、实现Callable接口，重写call函数方法：
```
Callable call =new Callable() {
    @Override
    public Object call() throws Exception {
        return null;
    }
}
```
### 小结：Callable 与 Runnable 对比。

* 相同：都是可被其它线程执行的任务。

* 不同：
     ①Callable规定的方法是call()，而Runnable规定的方法是run().
    ②Callable的任务执行后可返回值，而Runnable的任务是不能返回值的
    ③call()方法可抛出异常，而run()方法是不能抛出异常的。
    ④运行Callable任务可拿到一个Future对象，Future表示异步计算的结果。通过Future对象可了解任务执行情况,可取消任务的执行。

## 线程状态

* 1、wait()。使一个线程处于等待状态，并且释放所有持有对象的lock锁，直到notify()/notifyAll()被唤醒后放到锁定池(lock blocked pool )，释放同步锁使线程回到可运行状态（Runnable）。

* 2、sleep()。使一个线程处于睡眠状态，是一个静态方法，调用此方法要捕捉Interrupted异常，醒来后进入runnable状态，等待JVM调度。

* 3、notify()。使一个等待状态的线程唤醒，注意并不能确切唤醒等待状态线程，是由JVM决定且不按优先级。

* 4、allnotify()。使所有等待状态的线程唤醒，注意并不是给所有线程上锁，而是让它们竞争。

* 5、join()。使一个线程中断，IO完成会回到Runnable状态，等待JVM的调度。

* 6、Synchronized()。使Running状态的线程加同步锁使其进入(lock blocked pool ),同步锁被释放进入可运行状态(Runnable)。 

补充：在runnable状态的线程是处于被调度的线程，此时的调度顺序是不一定的。Thread类中的yield方法可以让一个running状态的线程转入runnable。

## 基础概念

* 1、 并行。多个cpu实例或多台机器同时执行一段代码，是真正的同时。

* 2、并发。通过cpu调度算法，让用户看上去同时执行，实际上从cpu操作层面不是真正的同时。

*  3、线程安全。指在并发的情况之下，该代码经过多线程使用，线程的调度顺序不影响任何结果。线程不安全就意味着线程的调度顺序会影响最终结果，比如某段代码不加事务去并发访问。

*  4、线程同步。指的是通过人为的控制和调度，保证共享资源的多线程访问成为线程安全，来保证结果的准确。如某段代码加入@synchronized关键字。线程安全的优先级高于性能优化。

*  5、原子性。一个操作或者一系列操作，要么全部执行要么全部不执行。数据库中的“事物”就是个典型的院子操作。

*  6、可见性。当一个线程修改了共享属性的值，其它线程能立刻看到共享属性值的更改。比如JMM分为主存和工作内存，共享属性的修改过程是在主存中读取并复制到工作内存中，在工作内存中修改完成之后，再刷新主存中的值。若线程A在工作内存中修改完成但还来得及刷新主存中的值，这时线程B访问该属性的值仍是旧值。这样可见性就没法保证。

*  7、有序性。程序运行时代码逻辑的顺序在实际执行中不一定有序，为了提高性能，编译器和处理器都会对代码进行重新排序。前提是，重新排序的结果要和单线程执行程序顺序一致。

## Synchronized 同步

由于java的每个对象都有一个内置锁，当用此关键字修饰方法时， 内置锁会保护整个方法。在调用该方法前，需要获得内置锁，否则就处于阻塞状态。补充： synchronized关键字也可以修饰静态方法，此时如果调用该静态方法，将会锁住整个类。

### 1、方法同步。

* 给方法增加synchronized修饰符就可以成为同步方法，可以是静态方法、非静态方法，但不能是抽象方法、接口方法。

小示例：
```
public synchronized void aMethod() { 
    // do something 
} 

public static synchronized void anotherMethod() { 
    // do something 
} 
```
* **使用详解**：线程在执行同步方法时是具有排它性的。当任意一个线程进入到一个对象的任意一个同步方法时，这个对象的所有同步方法都被锁定了，在此期间，其他任何线程都不能访问这个对象的任意一个同步方法，直到这个线程执行完它所调用的同步方法并从中退出，从而导致它释放了该对象的同步锁之后。在一个对象被某个线程锁定之后，其他线程是可以访问这个对象的所有非同步方法的。

### 2、块同步。

* 同步块是通过锁定一个指定的对象，来对块中的代码进行同步.

* 同步方法和同步块之间的相互制约只限于同一个对象之间，静态同步方法只受它所属类的其它静态同步方法的制约，而跟这个类的实例没有关系。

* 如果一个对象既有同步方法，又有同步块，那么当其中任意一个同步方法或者同步块被某个线程执行时，这个对象就被锁定了，其他线程无法在此时访问这个对象的同步方法，也不能执行同步块。

### 3、使用方法同步保护共享数据。

示例：
```
public class ThreadTest implements Runnable{

public synchronized void run(){
　　for(int i=0;i<10;i++) {
　　　　System.out.print(" " + i);
　　}
}

public static void main(String[] args) {
　　Runnable r1 = new ThreadTest(); 
　　Runnable r2 = new ThreadTest();
　　Thread t1 = new Thread(r1);
　　Thread t2 = new Thread(r2);
　　t1.start();
　　t2.start();
}}
```
**示例详解**：

* 代码中可见，run()被加上了synchronized 关键字，但保护的并不是共享数据。因为程序中两个线程对象 t1、t2 其实是另外两个线程对象 r1、r2 的线程，这个听起来绕，但是一眼你就能看明白；因为不同的线程对象的数据是不同的，即 r1,r2 有各自的run()方法，所以输出结果就无法预知。

* 这时使用 synchronized 关键字可以让某个时刻只有一个线程可以访问该对象synchronized数据。每个对象都有一个“锁标志”，当这个对象的一个线程访问这个对象的某个synchronized 数据时，这个对象的所有被synchronized 修饰的数据将被上锁（因为“锁标志”被当前线程拿走了），只有当前线程访问完它要访问的synchronized 数据时，当前线程才会释放“锁标志”，这样同一个对象的其它线程才有机会访问synchronized 数据。

接下来，我们把 r2 给注释掉， 即只保留一个 r 对象。如下：
```
public class ThreadTest implements Runnable{

public synchronized void run(){
　　for(int i=0;i<10;i++){
　　　　System.out.print(" " + i);
　　}
}

public static void main(String[] args){
　　Runnable r = new ThreadTest();
　　Thread t1 = new Thread(r);
　　Thread t2 = new Thread(r);
　　t1.start();
　　t2.start();
}} 
```
**示例详解**：

* 如果你运行1000 次这个程序，它的输出结果也一定每次都是：0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9。

* 因为这里的synchronized 保护的是共享数据。t1,t2 是同一个对象（r）的两个线程，当其中的一个线程（例如：t1）开始执行run()方法时，由于run()受synchronized保护，所以同一个对象的其他线程（t2）无法访问synchronized 方法（run 方法）。只有当t1执行完后t2 才有机会执行。

### 4、使用块同步，示例：
```
public class ThreadTest implements Runnable{
   public void run(){
      synchronized(this){  //与上面示例不同于关键字使用
           for(int i=0;i<10;i++){
               System.out.print(" " + i);
           }
      } 
   }
   public static void main(String[] args){
       Runnable r = new ThreadTest();
       Thread t1 = new Thread(r);
       Thread t2 = new Thread(r);
       t1.start();
       t2.start();
   }
} 
```
**示例详解**：

这个与上面示例的运行结果也一样的。这里是把保护范围缩到最小，this 代表 ‘这个对象’   。没有必要把整个run()保护起来，run()中的代码只有一个for循环，所以只要保护for 循环就可以了。

## Volatile 同步

* a.volatile关键字为域变量的访问提供了一种免锁机制

* b.使用volatile修饰域相当于告诉虚拟机该域可能会被其他线程更新

* c.因此每次使用该域就要重新计算，而不是使用寄存器中的值 

* d.volatile不会提供任何原子操作，它也不能用来修饰final类型的变量 

例如： 

在上面的例子当中，只需在account前面加上volatile修饰，即可实现线程同步。 

代码实例：
```
        //只给出要修改的代码，其余代码与上同
        class Bank {
            //需要同步的变量加上volatile
            private volatile int account = 100;
 
            public int getAccount() {
                return account;
            }
            //这里不再需要synchronized 
            public void save(int money) {
                account += money;
            }
        ｝
```
**注**：多线程中的非同步问题主要出现在对域的读写上，如果让域自身避免这个问题，则就不需要修改操作该域的方法。 

用final域，有锁保护的域和volatile域可以避免非同步的问题。

## 重入锁同步

在 [Java](http://www.2cto.com/kf/ware/Java/)SE5.0中 新增了一个 java.util.concurrent 包来支持同步。 

ReentrantLock类是可重入、互斥、实现了Lock接口的锁，它与使用synchronized方法和快具有相同的基本行为和语义，并且扩展了其能力。 ReenreantLock类的常用方法有：

ReentrantLock() : 创建一个ReentrantLock实例 
lock() : 获得锁 
unlock() : 释放锁 

注：ReentrantLock()还有一个可以创建公平锁的构造方法，但由于能大幅度降低程序运行效率，不推荐使用 例如：
```
        class Bank {
            private int account = 100;
            //需要声明这个锁
            private Lock lock = new ReentrantLock();
            public int getAccount() {
                return account;
            }
            //这里不再需要synchronized 
            public void save(int money) {
                lock.lock();
                try{
                    account += money;
                }finally{
                    lock.unlock();
                }
            }
        ｝
```
**注：关于Lock对象和synchronized关键字的选择：** 

        a.最好两个都不用，使用一种java.util.concurrent包提供的机制，能够帮助用户处理所有与锁相关的代码。 

        b.如果synchronized关键字能满足用户的需求，就用synchronized，因为它能简化代码 

        c.如果需要更高级的功能，就用ReentrantLock类，此时要注意及时释放锁，否则会出现死锁，通常在finally代码释放锁 

## 局部变量同步

    如果使用ThreadLocal管理变量，则每一个使用该变量的线程都获得该变量的副本，副本之间相互独立，这样每一个线程都可以随意修改自己的变量副本，而不会对其他线程产生影响。 ThreadLocal 类的常用方法：

* ThreadLocal() : 创建一个线程本地变量 
* get() : 返回此线程局部变量的当前线程副本中的值 
* initialValue() : 返回此线程局部变量的当前线程的"初始值"
* set(T value) : 将此线程局部变量的当前线程副本中的值设置为value

例如：
```
public class Bank{
            //使用ThreadLocal类管理共享变量account
            private static ThreadLocal<Integer> account = new ThreadLocal<Integer>(){
                @Override
                protected Integer initialValue(){
                    return 100;
                }
            };
            public void save(int money){
                account.set(account.get()+money);
            }
            public int getAccount(){
                return account.get();
            }
        }
```
**注：ThreadLocal与同步机制**

        a.ThreadLocal与同步机制都是为了解决多线程中相同变量的访问冲突问题。

        b.前者采用以"空间换时间"的方法，后者采用以"时间换空间"的方式

## 阻塞队列同步

前面同步方式都是在底层实现的线程同步，但是我们在实际开发当中，应当尽量远离底层结构。 使用javaSE5.0版本中新增的java.util.concurrent包将有助于简化开发。

本小节主要是使用LinkedBlockingQueue<E>来实现线程的同步 LinkedBlockingQueue<E>是一个基于已连接节点的，范围任意的blocking queue。

队列是先进先出的顺序（FIFO），关于队列以后会详细讲解~LinkedBlockingQueue 类常用方法 LinkedBlockingQueue() : 创建一个容量为Integer.MAX_VALUE的LinkedBlockingQueue put(E e) : 在队尾添加一个元素，如果队列满则阻塞 size() : 返回队列中的元素个数 take() : 移除并返回队头元素，如果队列空则阻塞代码实例： 实现商家生产商品和买卖商品的同步

注：BlockingQueue<E>定义了阻塞队列的常用方法，尤其是三种添加元素的方法，我们要多加注意，当队列满时：

　　add()方法会抛出异常

　　offer()方法返回false

　　put()方法会阻塞

## 原子变量同步

需要使用线程同步的根本原因在于对普通变量的操作不是原子的。

**那么什么是原子操作呢**？

原子操作就是指将读取变量值、修改变量值、保存变量值看成一个整体来操作即-这几种行为要么同时完成，要么都不完成。在java的**util.concurrent.atomic包中提供了创建了原子类型变量的工具类**，使用该类可以简化线程同步。其中**AtomicInteger** 表可以用原子方式更新int的值，可用在应用程序中(如以原子方式增加的计数器)，但不能用于替换Integer；可扩展Number，允许那些处理机遇数字类的工具和实用工具进行统一访问。

**AtomicInteger类常用方法：**

AtomicInteger(int initialValue) : 创建具有给定初始值的新的

AtomicIntegeraddAddGet(int dalta) : 以原子方式将给定值与当前值相加

get() : 获取当前值

**代码实例：**
```
class Bank {
    private AtomicInteger account = new AtomicInteger(100);
    public AtomicInteger getAccount() {
        return account; 
    } 
    public void save(int money) {
        account.addAndGet(money);
    }
```
**补充--原子操作主要有：**　　

对于引用变量和大多数原始变量(long和double除外)的读写操作；　　

对于所有使用volatile修饰的变量(包括long和double)的读写操作。
