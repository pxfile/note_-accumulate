mt面试题2
===
# Java 部分

## HashMap 的key需要注意什么？hashCode与equals 在hashMap中的使用

### key注意的地方

#### 1、什么是可变对象

可变对象是指创建后自身状态能改变的对象。换句话说，可变对象是该对象在创建后它的哈希值可能被改变。

#### 2、HashMap如何存储键值对

HashMap用Key的哈希值来存储和查找键值对。
当插入一个Entry时，HashMap会计算Entry Key的哈希值。Map会根据这个哈希值把Entry插入到相应的位置。
查找时，HashMap通过计算Key的哈希值到特定位置查找这个Entry。

#### 3、在HashMap中使用可变对象作为Key带来的问题

如果HashMap Key的哈希值在存储键值对后发生改变，Map可能再也查找不到这个Entry了。

如果Key对象是可变的，那么Key的哈希值就可能改变。在HashMap中可变对象作为Key会造成数据丢失。

#### 4、如何解决

在HashMap中使用不可变对象。在HashMap中，使用String、Integer等不可变类型用作Key是非常明智的。

我们也能定义属于自己的不可变类。

如果可变对象在HashMap中被用作键，那就要小心在改变对象状态的时候，不要改变它的哈希值了。

### hashCode与equals 在hashMap中的使用

* HashMap的实现方式是数组链，不同的对象根据其哈希码（hashCode方法的返回值）找到对应的数组下标，然后存入数组。不同的对象有相同的哈希码时怎么办？这就由数组链中的链来解决了，相同哈希码的对象都放在同一条链上，该链的链头指向数组，进而形成数组链。

当第一个对象已经存入HashMap，第二个对象准备存入HashMap时，系统在查找到数组下标后若发现它们的hashCode相同（也就是冲突），会调用equals()来检查它们之间的关系，会有相应有以下两种处理方法： 
1. 如果相等，系统就不再存入第二个对象; 
2. 如果不等，系统视它们为纯粹的下标冲突，将它们放在同一条链上;

如果它们的hashCode不相同，直接存入第二个对象。

## 深clone与浅clone

### 什么是clone

**复制对象**，首先要分配一个和源对象同样大小的空间，在这个空间中创建一个新的对象。

那么在java语言中，**有几种方式可以创建对象呢**？ 

* 1\. 使用new操作符创建一个对象 

* 2\. 使用clone方法复制一个对象 

**那么这两种方式有什么相同和不同呢**？ 

* new操作符的本意是分配内存。程序执行到new操作符时， 首先去看new操作符后面的类型，因为知道了类型，才能知道要分配多大的内存空间。分配完内存之后，再调用构造函数，填充对象的各个域，这一步叫做对象的初始化，构造方法返回后，一个对象创建完毕，可以把他的引用（地址）发布到外部，在外部就可以使用这个引用操纵这个对象。

* 而clone在第一步是和new相似的， 都是分配内存，调用clone方法时，分配的内存和源对象（即调用clone方法的对象）相同，然后再使用原对象中对应的各个域，填充新对象的域， 填充完成之后，clone方法返回，一个新的相同的对象被创建，同样可以把这个新对象的引用发布到外部。

### 深clone与浅clone的区别

* 深拷贝难以完全形成深拷贝，因为这要求继承链上的所有对象都集成Cloneable接口，实现clone方法，来保证非基本类型的对象被深拷贝

* clone在平时项目的开发中可能用的不是很频繁，但是区分深拷贝和浅拷贝会让我们对java内存结构和运行方式有更深的了解。至于彻底深拷贝，几乎是不可能实现的，原因已经在上一节中进行了说明。

* 深拷贝和彻底深拷贝，在创建不可变对象时，可能对程序有着微妙的影响，可能会决定我们创建的不可变对象是不是真的不可变。clone的一个重要的应用也是用于不可变对象的创建。

## volatile与Syncronized区别
 * 1）Synchronized保证内存可见性和操作的原子性
 * 2）Volatile只能保证内存可见性
 * 3）Volatile不需要加锁，比Synchronized更轻量级，并不会阻塞线程（volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。）
 * 4）volatile标记的变量不会被编译器优化,而synchronized标记的变量可以被编译器优化（如编译器重排序的优化）.
 * 5）volatile是变量修饰符，仅能用于变量，而synchronized是一个方法或块的修饰符。
 * 6）volatile本质是在告诉JVM当前变量在寄存器中的值是不确定的，使用前，需要先从主存中读取，因此可以实现可见性。而对n=n+1,n++等操作时，volatile关键字将失效，不能起到像synchronized一样的线程同步（原子性）的效果。
 * 7）synchronized关键字是防止多个线程同时执行一段代码，那么就会影响程序执行效率，而volatile关键字在某些情况下性能要优于synchronized，但是要注意volatile关键字是无法替代synchronized关键字的，因为volatile关键字无法保证操作的原子性。

## java线程的状态？wait与notify调用时线程状态的切换形式？

### 线程的 5 中状态

![状态图](http://img.blog.csdn.net/20161011141501523)

#### 1. New 新建状态

*  当程序使用 new 关键字创建了一个线程后，该线程就处于新建状态，此时线程还未启劢，
*  当线程对象调用 start()方法时，线程启劢，进入 Runnable 状态

#### 2. Runnable 可运行（就绪）状态

当线程处于 Runnable 状态时，表示线程准备就绪，等待获取 CPU

#### 3. Running 运行（正在运行）状态

*  假如该线程获取了 CPU，则进入 Running 状态，开始执行线程体，即 run()方法中的内容
*  注意：
*   如果系统叧有 1 个 CPU，那么在任意时间点则叧有 1 条线程处于 Running 状态；
*  如果是双核系统，那么同一时间点会有 2 条线程处于 Running 状态但是，当线程数大于处理器数时，依然会是多条线程在同一个 CPU 上轮换执行 
*  当一条线程开始运行时，如果它不是一瞬间完成，那么它不可能一直处于 Running 状态，
*  线程在执行过程中会被中断，目的是让其它线程获得执行的机会，像这样线程调度的策略取决于底层平台。对于抢占式策略的平台而言，系统系统会给每个可执行的线程一小段时间来处理仸务，当该时间段（时间片）用完，系统会剥夺该线程所占资源（CPU），让其他线程获得运行机会。
*  调用 yield()方法，可以使线程由 Running 状态进入 Runnable 状态

#### 4. Block 阻塞（挂起）状态

当如下情冴下，线程会进入阻塞状态： 
* 线程调用了 sleep()方法主动放弃所占 CPU 资源 
* 线程调用了一个阻塞式 IO 方法（比如控制台输入方法），在该方法返回前，该线程被阻塞 
* 当正在执行的线程被阻塞时，其它线程就获得执行机会了。需要注意的是，当阻塞结束时，该线程将进入 Runnable 状态，而非直接进入 Running 状态

#### 5. Dead 死亡状态

 *  当线程的 run()方法执行结束，线程进入 Dead 状态
 *  需要注意的是，不要试图对一个已经死亡的线程调用 start()方法，线程死亡后将不能再次作为线程执行，系统会抛出 IllegalThreadStateException 异常

### wait与notify调用时线程状态的切换形式

* wait

Object的wait方法有三个重载方法，其中一个方法wait() 是无限期(一直)等待，直到其它线程调用notify或notifyAll方法唤醒当前的线程；另外两个方法wait(long timeout) 和wait(long timeout, int nanos)允许传入 当前线程在被唤醒之前需要等待的时间，timeout为毫秒数，nanos为纳秒数。

* notify

notify方法只唤醒一个等待（对象的）线程并使该线程开始执行。所以如果有多个线程等待一个对象，这个方法只会唤醒其中一个线程，选择哪个线程取决于操作系统对多线程管理的实现。

* notifyAll

notifyAll 会唤醒所有等待(对象的)线程，尽管哪一个线程将会第一个处理取决于操作系统的实现。

**当线程执行wait()时，会把当前的锁释放，然后让出CPU，进入等待状态**

**当执行notify/notifyAll方法时，会唤醒一个处于等待该 对象锁 的线程，然后继续往下执行，直到执行完退出对象锁锁住的区域（synchronized修饰的代码块）后再释放锁**

从这里可以看出，notify/notifyAll()执行后，并不立即释放锁，而是要等到执行完临界区中代码后，再释放。故，在实际编程中，我们应该尽量在线程调用notify/notifyAll()后，立即退出临界区。即不要在notify/notifyAll()后面再写一些耗时的代码。

## 删除集合中某一个元素的方式?
```
package test;

import java.text.MessageFormat;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Iterator;
import java.util.List;
import java.util.Locale;
import java.util.function.Predicate;

public class T {

	public static void main(String[] args) {
		//对于list添加数据，也有几种方法，性能不同，有兴趣可以自己详细了解
		List<Integer> list = new ArrayList<>(Arrays.asList(1,2,3,4,5));
		
		//List<Integer> list = new ArrayList<>();
		//Collections.addAll(list,1,2,3,4,5);
	 
		remove(list, 3);
		print(list);
	}
	
	public static void print(List<Integer> list) {
		/*for (Integer i : list) {
			System.out.println(i);
		}*/
		
		/**
		 * 利用java8新特性打印集合信息
		 */
		list.stream().forEach(System.out::println);
	}
	
	/**
	 * 删除集合某一元素的方法汇总
	 * @param list
	 * @param toRemoveValue
	 * @return
	 */
	public static List<Integer> remove(List<Integer> list, int toRemoveValue) {
		/**
		 * 方法1，利用java.util.Iterator删除某一个元素
		 */
		for (Iterator<Integer> iterator=list.iterator();iterator.hasNext();) {
			int value = iterator.next();
			if(value == toRemoveValue) {
				iterator.remove();
			}
		}
		//----------------------------------------------------------------------------
		/**
		 * 方法2，利用操作集合索引删除某一个元素
		 */
		for (int i = 0; i < list.size(); i++) {
			int value = list.get(i);
			if(value == toRemoveValue) {
				list.remove(i);
				i--;
			}
		}
		
		//----------------------------------------------------------------------------
		/**
		 * 方法3，利用反向遍历集合删除某一个元素
		 */
		for (int i = list.size() - 1;i > 0; i--) {
			int value = list.get(i);
			if(value == toRemoveValue) {
				list.remove(i);
			}
		}
		
		//----------------------------------------------------------------------------
		/**
		 * 方法4，利用增加一个新集合删除某一个元素
		 */
		List<Integer> toRemoveList = new ArrayList<>();
		for (int i = 0; i < list.size(); i++) {
			int value = list.get(i);
			if(value == toRemoveValue) {
				toRemoveList.add(value);
			}
		}
		
		list.removeAll(toRemoveList);
		
		//----------------------------------------------------------------------------
		/**
		 * 方法5，利用break取巧删除某一个元素，实际应用不常用
		 */
		for (int i = 0; i < list.size(); i++) {
			int value = list.get(i);
			if(value == toRemoveValue) {
				list.remove(i);
				break;
			}
		}
		
		//----------------------------------------------------------------------------
		/**
		 * 方法6，利用java8新特性删除某一个元素
		 */
		list.removeIf(value -> value==toRemoveValue);
		
		//----------------------------------------------------------------------------
		/**
		 * 方法7，利用java8新特性的forEach()方法删除某一个元素,其实不是删除，是过滤
		 */
		list.stream().filter(value -> value!=toRemoveValue).forEach(System.out::println);
		
		//----------------------------------------------------------------------------
		/**
		 * 方法8，利用java8新特性的forEach()方法删除某一个元素,其实不是删除，是过滤，等同上一种写法
		 */
		list.stream().forEach(value -> { if(value!=toRemoveValue) System.out.println(value);});
		
		
		return list;
	}

}

```

## 是否看过JDK 的相关源码？常用集合类的底层数据结构是怎样的?
[Java中常见数据结构：list与map -底层如何实现](http://www.cnblogs.com/nucdy/p/5867210.html)

### 1:集合
    Collection(单列集合)
        List(有序,可重复)
            ArrayList
                底层数据结构是数组,查询快,增删慢
                线程不安全,效率高
            Vector
                底层数据结构是数组,查询快,增删慢
                线程安全,效率低
            LinkedList
                底层数据结构是链表,查询慢,增删快
                线程不安全,效率高
        Set(无序,唯一)
            HashSet
                底层数据结构是哈希表。
                哈希表依赖两个方法：hashCode()和equals()
                执行顺序：
                    首先判断hashCode()值是否相同
                        是：继续执行equals(),看其返回值
                            是true:说明元素重复，不添加
                            是false:就直接添加到集合
                        否：就直接添加到集合
                最终：
                    自动生成hashCode()和equals()即可
                    
                LinkedHashSet
                    底层数据结构由链表和哈希表组成。
                    由链表保证元素有序。
                    由哈希表保证元素唯一。
            TreeSet
                底层数据结构是红黑树。(是一种自平衡的二叉树)
                如何保证元素唯一性呢?
                    根据比较的返回值是否是0来决定
                如何保证元素的排序呢?
                    两种方式
                        自然排序(元素具备比较性)
                            让元素所属的类实现Comparable接口
                        比较器排序(集合具备比较性)
                            让集合接收一个Comparator的实现类对象
    Map(双列集合)
        A:Map集合的数据结构仅仅针对键有效，与值无关。
        B:存储的是键值对形式的元素，键唯一，值可重复。
        
        HashMap
            底层数据结构是哈希表。线程不安全，效率高
                哈希表依赖两个方法：hashCode()和equals()
                执行顺序：
                    首先判断hashCode()值是否相同
                        是：继续执行equals(),看其返回值
                            是true:说明元素重复，不添加
                            是false:就直接添加到集合
                        否：就直接添加到集合
                最终：
                    自动生成hashCode()和equals()即可
            LinkedHashMap
                底层数据结构由链表和哈希表组成。
                    由链表保证元素有序。
                    由哈希表保证元素唯一。
        Hashtable
            底层数据结构是哈希表。线程安全，效率低
                哈希表依赖两个方法：hashCode()和equals()
                执行顺序：
                    首先判断hashCode()值是否相同
                        是：继续执行equals(),看其返回值
                            是true:说明元素重复，不添加
                            是false:就直接添加到集合
                        否：就直接添加到集合
                最终：
                    自动生成hashCode()和equals()即可
        TreeMap
            底层数据结构是红黑树。(是一种自平衡的二叉树)
                如何保证元素唯一性呢?
                    根据比较的返回值是否是0来决定
                如何保证元素的排序呢?
                    两种方式
                        自然排序(元素具备比较性)
                            让元素所属的类实现Comparable接口
                        比较器排序(集合具备比较性)
                            让集合接收一个Comparator的实现类对象
    
### 2.关于集合选取原则
    
    是否是键值对象形式:
        是：Map
            键是否需要排序:
                是：TreeMap
                否：HashMap
            不知道，就使用HashMap。
            
        否：Collection
            元素是否唯一:
                是：Set
                    元素是否需要排序:
                        是：TreeSet
                        否：HashSet
                    不知道，就使用HashSet
                    
                否：List
                    要安全吗:
                        是：Vector
                        否：ArrayList或者LinkedList
                            增删多：LinkedList
                            查询多：ArrayList
                        不知道，就使用ArrayList
            不知道，就使用ArrayList
            
### 3:集合的常见方法及遍历方式
    Collection:
        add()
        remove()
        contains()
        iterator()
        size()
        
        遍历：
            增强for
            迭代器
            
        |--List
            get()
            
            遍历：
                普通for
        |--Set
    
    Map:
        put()
        remove()
        containskey(),containsValue()
        keySet()
        get()
        value()
        entrySet()
        size()
        
        遍历：
            根据键找值
            根据键值对对象分别找键和值

## java集合框架的继承关系，画图说明（考察是否对集合框架了解深入和成体系）

数组虽然也可以存储对象，但长度是固定的；集合长度是可变的，数组中可以存储基本数据类型，集合只能存储对象。

　　集合类的特点：集合只用于存储对象，集合长度是可变的，集合可以存储不同类型的对象。

　　　　　　　　　　![](https://images2015.cnblogs.com/blog/1010726/201706/1010726-20170621004734695-988542448.png)

　　　　　　　　　　　![](https://images2015.cnblogs.com/blog/1010726/201706/1010726-20170621004756882-1379253225.gif)

　　上述类图中，实线边框的是实现类，比如ArrayList，LinkedList，HashMap等，折线边框的是抽象类，比如AbstractCollection，AbstractList，AbstractMap等，而点线边框的是接口，比如Collection，Iterator，List等。

## 什么是泛型？泛型擦除？（考察通用框架设计能力）
* **泛型**：
本质是参数化类型。 

* **为什么要使用**？
创建集合的时候，往集合里面添加数据，再次取出时，集合会忘记这数据类型，该对象的编译类型就会变成Object类型，否则如果想要变回原来的数据类型的时候，就要强制进行转换。创建集合的时候，我们就指定集合类型，避免这个过程。 

* **泛型擦除？** 
Java的泛型处理过程都是在编译器中进行的，编译器首先会生成bytecode码，这个过程是不包括泛型类型，泛型类型在编译的时候是被擦除的，这个过程及泛型擦除。 

* **泛型擦除的过程：** 
1将所有泛型参数用顶级父类类型替换 
2擦除所有的参数类型

## 注解原理 （结合APT）
[Android注解使用之注解编译android-apt如何切换到annotationProcessor](http://www.cnblogs.com/whoislcj/p/6148410.html)

### 前言：

    自从EventBus 3.x发布之后其通过注解预编译的方式解决了之前通过反射机制所引起的性能效率问题，其中注解预编译所采用的的就是android-apt的方式，不过最近Apt工具的作者宣布了不再维护该工具了，因为Android Studio推出了官方插件，并且可以通过gradle来简单的配置，它就是annotationProcessor，今天来学习一下如何将原来的android-apt切换到annotationProcessor。

### 什么是APT？

   APT(Annotation Processing Tool)是一种处理注释的工具,它对源代码文件进行检测找出其中的Annotation，使用Annotation进行额外的处理。 Annotation处理器在处理Annotation时可以根据源文件中的Annotation生成额外的源文件和其它的文件(文件具体内容由Annotation处理器的编写者决定),APT还会编译生成的源文件和原来的源文件，将它们一起生成class文件。

### 使用背景：

  随着Android Gradle 插件 2.2 版本的发布，Android Gradle 插件提供了名为 `annotationProcessor`  的功能来完全代替  `android-apt` ，自此`android-apt`  作者在官网发表声明证实了后续将不会继续维护  `android-apt` ，并推荐大家使用 Android 官方插件annotationProcessor。

### 切换步骤：

首先要确保Android Gradle插件版本是2.2以上，目前我们所使用的Android studio版本是2.2.3，所对应的的Android Gradle版本也是2.2.3

#### 1.）修改Project 的build.gradle配置

android-apt方式

 dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }

修改后`annotationProcessor`  方式

 dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'
    }

#### 2.）修改module的build.gradle配置

android-apt方式

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}
apply plugin: 'com.neenbedankt.android-apt'
dependencies {
    compile 'org.greenrobot:eventbus:3.0.0'
    apt'org.greenrobot:eventbus-annotation-processor:3.0.1'//apt
}

修改后`annotationProcessor`  方式，只保留dependencies 里面的引用并且把apt 换成annotationProcessor就可以了

dependencies {
    compile 'org.greenrobot:eventbus:3.0.0'
    annotationProcessor  'org.greenrobot:eventbus-annotation-processor:3.0.1'
}

#### 3.）对EventBus 3.0 使用索引的兼容

android-apt方式

apt  {
    arguments {
        eventBusIndex "org.greenrobot.eventbusperf.MyEventBusIndex"
    }
}

修改后`annotationProcessor`  方式

defaultConfig {
        applicationId "com.whoislcj.testhttp"
        minSdkVersion 21
        targetSdkVersion 24
        versionCode 1
        versionName "1.0"
        jackOptions {
            enabled true
        }
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = [ eventBusIndex : 'org.greenrobot.eventbusperf.MyEventBusIndex' ]
            }
        }
    }
### 两者对比

   最近Android N的发布，android 迎来了Java 8，要想使用Java 8的话必须使用Jack编译，android-apt只支持javac编译而annotationProcessor既支持javac同时也支持jack编译。

## 反射原理 （结合插件，热修复）
 [反射技术在android中的应用](https://pxfile.github.io/2018/03/13/%E5%8F%8D%E5%B0%84%E6%8A%80%E6%9C%AF%E5%9C%A8android%E4%B8%AD%E7%9A%84%E5%BA%94%E7%94%A8/)

## 垃圾回收，结合具体案例分析（比如onDraw中为什么不能频繁new对象）
[android--垃圾回收与内存优化](https://www.jianshu.com/p/5f5e2a608871)

执行GC操作的时候，任何线程的任何操作都会需要暂停，等待GC操作完成之后，其他操作才能够继续运行。

通常来说，单个的GC并不会占用太多时间，但是大量不停的GC操作则会显著占用帧间隔时间(16ms)。如果在帧间隔时间里面做了过多的GC操作，那么自然其他类似计算，渲染等操作的可用时间就变得少了。

### 年轻代（young generation）

年轻代是所有新对象产生的地方。当年轻代内存空间被用完时，就会触发垃圾回收。这个垃圾回收叫做**Minor GC**。年轻代被分为3个部分**——Enden区**和两个**Survivor区**。

*   大多数新建的对象都位于Eden区。
*   当Eden区被对象填满时，就会执行Minor GC。并把所有存活下来的对象转移到其中一个survivor区。
*   Minor GC同样会检查存活下来的对象，并把它们转移到另一个survivor区。这样在一段时间内，总会有一个空的survivor区。
*   经过多次GC周期后，仍然存活下来的对象会被转移到年老代内存空间。通常这是在年轻代有资格提升到年老代前通过设定年龄阈值来完成的。

### 老年代（Old Generation）

年老代内存里包含了长期存活的对象和经过多次**Minor GC**后依然存活下来的对象。通常会在老年代内存被占满时进行垃圾回收。老年代的垃圾收集叫做**Major GC**。Major GC会花费更多的时间。

### 永久代（Permanent Generation）

存放方法区，方法区中有要加载的类信息、静态变量、final类型的常量、属性和方法信息。

### **导致GC频繁执行有两个原因：**
* 1.内存抖动
* 2.瞬间产生大量的对象会严重占用Young Generation的内存区域，当达到阀值，剩余空间不够的时候，也会触发GC。即使每次分配的对象占用了很少的内存，但是他们叠加在一起会增加Heap的压力，从而触发更多其他类型的GC。这个操作有可能会影响到帧率，并使得用户感知到性能问题。

所以避免在onDraw方法中创建对象。

## 强软弱虚与引用队列 （考察GC，缓存的设计）
 [android四种引用的详解](https://pxfile.github.io/2018/03/16/Android%E5%9B%9B%E7%A7%8D%E5%BC%95%E7%94%A8%E7%9A%84%E8%AF%A6%E8%A7%A3/)

## java的类加载机制，双亲委托（结合热修复与插件化考察）

[Android 插件化和热修复知识梳理](http://www.colabug.com/1965956.html)

[Android解析ClassLoader（一）Java中的ClassLoader](http://liuwangshu.cn/application/classloader/1-java-classloader-.html)

### **双亲委托模式的特点**

类加载器查找Class所采用的是双亲委托模式，所谓双亲委托模式就是首先判断该Class是否已经加载，如果没有则不是自身去查找而是委托给父加载器进行查找，这样依次的进行递归，直到委托到最顶层的Bootstrap ClassLoader，如果Bootstrap ClassLoader找到了该Class，就会直接返回，如果没找到，则继续依次向下查找，如果还没找到则最后会交由自身去查找。
这样讲可能会有些抽象，来看下面的图。

[![](http://upload-images.jianshu.io/upload_images/1417629-cf0b87b85d4e0d7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://upload-images.jianshu.io/upload_images/1417629-cf0b87b85d4e0d7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **双亲委托模式的好处**

采取双亲委托模式主要有两点好处：

1.  避免重复加载，如果已经加载过一次Class，就不需要再次加载，而是先从缓存中直接读取。
2.  更加安全，如果不使用双亲委托模式，就可以自定义一个String类来替代系统的String类，这显然会造成安全隐患，采用双亲委托模式会使得系统的String类在Java虚拟机启动时就被加载，也就无法自定义String类来替代系统的String类，除非我们修改

类加载器搜索类的默认算法。还有一点，只有两个类名一致并且被同一个类加载器加载的类，Java虚拟机才会认为它们是同一个类，想要骗过Java虚拟机显然不会那么容易。

# Android 部分

## Android有哪些跨进程通信的方式？（考察对整个android跨进程通信的了解）

[Android 进阶13：几种进程通信方式的对比总结](http://blog.csdn.net/u011240877/article/details/72863432)

跨进程通信要求把方法调用及其数据分解至操作系统可以识别的程度，并将其从本地进程和地址空间传输至远程进程和地址空间，然后在远程进程中重新组装并执行该调用。

然后，返回值将沿相反方向传输回来。

### Android 为我们提供了以下几种进程通信机制

供开发者使用的进程通信 API）对应的文章链接如下：

*   文件
*   AIDL （基于 Binder） 

    *   [Android 进阶：进程通信之 AIDL 的使用](http://blog.csdn.net/u011240877/article/details/72765136)
    *   [Android 进阶：进程通信之 AIDL 解析](http://blog.csdn.net/u011240877/article/details/72825706)
*   Binder 

    *   [Android 进阶：进程通信之 Binder 机制浅析](http://blog.csdn.net/u011240877/article/details/72801425)
*   Messenger （基于 Binder） 

    *   [Android 进阶：进程通信之 Messenger 使用与解析](http://blog.csdn.net/u011240877/article/details/72836178)
*   ContentProvider （基于 Binder） 

    *   [Android 进阶：进程通信之 ContentProvider 内容提供者](http://blog.csdn.net/u011240877/article/details/72848608)
*   Socket 

    *   [Android 进阶：进程通信之 Socket （顺便回顾 TCP UDP）](http://blog.csdn.net/u011240877/article/details/72860483)

在上述通信机制的基础上，我们只需集中精力定义和实现 RPC 编程接口即可。

### 如何选择这几种通信方式

《Android 开发艺术探索》中总结的已经比较全面了：

![这里写图片描述](http://img.blog.csdn.net/20170605011532312?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMTI0MDg3Nw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这里再对比总结一下：

*   只有允许不同应用的客户端用 IPC 方式调用远程方法，并且想要在服务中**处理多线程**时，才有必要使用 `AIDL`
*   如果需要调用远程方法，但不需要处理并发 IPC，就应该通过实现一个 `Binder` 创建接口
*   如果您想执行 IPC，但只是传递数据，不涉及方法调用，也不需要高并发，就使用 `Messenger` 来实现接口
*   如果需要处理一对多的进程间数据共享（主要是数据的 CRUD），就使用 `ContentProvider`
*   如果要实现一对多的并发实时通信，就使用 `Socket`

## 从源码看，有什么缺陷？（考察是否了解过源码以及是否发现问题）

### AsyncTask的各种坑

* 1、在Activity中定义AsyncTask导致内存泄漏 

由于AsyncTask是Activity的内部类，所以会持有外部的一个引用，如果Activity已经退出，但是AsyncTask还没有执行完毕，那么Activity就无法释放导致内存泄漏。对于这个问题我们可以把AsyncTask定义为静态内部类并且采用弱引用。 

* 2、各版本对AsyncTask的实现不一样 

对于这个问题，可以自己扩展一下AsyncTask在其内部也对版本做出判断，对于不同版本做一些不同的处理。 

* 3、不能及时取消任务 

以4.4版本双核手机为例，如果用户在A界面发起5个任务，由于使用SerialExecutor来执行任务，那么任务将一个一个顺序执行，由于第一个任务执行时间过长，其他任务只能在队列中等待，导致阻塞，所以可以考虑把执行时间较短的任务优先加入。如果在第一个任务执行过程中，用户跳转到了B界面，而A界面发起的任务已经没有必要执行，所以我们要在Activity的生命周期结束的时候取消掉任务。如果任务没取消掉，B界面又发起新的任务，就会导致B界面的所有请求阻塞。如果有需要我们可以直接使用executeOnExecutor方法，然后直接使用THREAD_POOL_EXECUTOR线程池来执行，这样可以3个线程同时执行，并且在doInBackground方法中判断任务是否被取消，这样可以提高效率。 

* 4、曾经缺陷 

以前对于缺陷的答案可能是：AsyncTask在并发执行多个任务时发生异常。其实还是存在的，在3.0以前的系统中还是会以支持多线程并发的方式执行，128，阻塞队列可以存放10个；也就是同时执行138个任务是没有问题的；而超过138会马上出现java.util.concurrent.RejectedExecutionException； 
而在在3.0以上包括3.0的系统中会为单线程执行；
在3.0以后，无论有多少任务，都会在其内部单线程执行；

## 从当前activity启动另外一个Activity，经历了哪些流程？（考察是否了解AMS,以及binder通信）

[Android应用程序内部启动Activity过程（startActivity）](http://blog.csdn.net/luoshengyang/article/details/6703247)

* 应用程序的MainActivity通过Binder进程间通信机制通知ActivityManagerService，它要启动一个新的Activity；

* ActivityManagerService通过Binder进程间通信机制通知MainActivity进入Paused状态；

* MainActivity通过Binder进程间通信机制通知ActivityManagerService，它已经准备就绪进入Paused状态，于是ActivityManagerService就准备要在MainActivity所在的进程和任务中启动新的Activity了；

* ActivityManagerService通过Binder进程间通信机制通知MainActivity所在的ActivityThread，现在一切准备就绪，它可以真正执行Activity的启动操作了。

和[Android应用程序启动过程源代码分析](http://blog.csdn.net/luoshengyang/article/details/6689748)中启动应用程序的默认Activity相比，这里在应用程序内部启动新的Activity的过程少了中间创建新的进程这一步，这是因为新的Activity是在已有的进程和任务中执行的，无须创建新的进程和任务。

## 线程间通信：Handler机制考察

### handler 与looper，Messagequeue的关系

### handler在线程中的使用与交互

### 为什么Android中looper进入了消息循环，而没有出现卡死？(考察了解深度，Linux epoll机制)

1.Android中为什么主线程不会因为Looper.loop()里的死循环卡死？

2.没看见哪里有相关代码为这个死循环准备了一个新线程去运转？

3.Activity的生命周期这些方法这些都是在主线程里执行的吧，那这些生命周期方法是怎么实现在死循环体外能够执行起来的？

#### **(1) Android中为什么主线程不会因为Looper.loop()里的死循环卡死？**

这里涉及线程，先说说说进程/线程，**进程：**每个app运行时前首先创建一个进程，该进程是由Zygote fork出来的，用于承载App上运行的各种Activity/Service等组件。进程对于上层应用来说是完全透明的，这也是google有意为之，让App程序都是运行在Android Runtime。大多数情况一个App就运行在一个进程中，除非在AndroidManifest.xml中配置Android:process属性，或通过native代码fork进程。

**线程：**线程对应用来说非常常见，比如每次new Thread().start都会创建一个新的线程。该线程与App所在进程之间资源共享，从Linux角度来说进程与线程除了是否共享资源外，并没有本质的区别，都是一个task_struct结构体**，在CPU看来进程或线程无非就是一段可执行的代码，CPU采用CFS调度算法，保证每个task都尽可能公平的享有CPU时间片**。

有了这么准备，再说说死循环问题：

对于线程既然是一段可执行的代码，当可执行代码执行完成后，线程生命周期便该终止了，线程退出。而对于主线程，我们是绝不希望会被运行一段时间，自己就退出，那么如何保证能一直存活呢？**简单做法就是可执行代码是能一直执行下去的，死循环便能保证不会被退出**,例如，binder线程也是采用死循环的方法，通过循环方式不同与Binder驱动进行读写操作，当然并非简单地死循环，无消息时会休眠。但这里可能又引发了另一个问题，既然是死循环又如何去处理其他事务呢？通过创建新线程的方式。

真正会卡死主线程的操作是在回调方法onCreate/onStart/onResume等操作时间过长，会导致掉帧，甚至发生ANR，looper.loop本身不会导致应用卡死。

#### **(2) 没看见哪里有相关代码为这个死循环准备了一个新线程去运转？**


事实上，会在进入死循环之前便创建了新binder线程，在代码ActivityThread.main()中：

```java
public static void main(String[] args) {
        ....

        //创建Looper和MessageQueue对象，用于处理主线程的消息
        Looper.prepareMainLooper();

        //创建ActivityThread对象
        ActivityThread thread = new ActivityThread(); 

        //建立Binder通道 (创建新线程)
        thread.attach(false);

        Looper.loop(); //消息循环运行
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }

```

**thread.attach(false)；便会创建一个Binder线程（具体是指ApplicationThread，Binder的服务端，用于接收系统服务AMS发送来的事件），该Binder线程通过Handler将Message发送给主线程**，具体过程可查看 [startService流程分析](https://link.zhihu.com/?target=http%3A//gityuan.com/2016/03/06/start-service/)，这里不展开说，简单说Binder用于进程间通信，采用C/S架构。关于binder感兴趣的朋友，可查看我回答的另一个知乎问题：
[为什么Android要采用Binder作为IPC机制？ - Gityuan的回答](https://www.zhihu.com/question/39440766/answer/89210950)

另外，**ActivityThread实际上并非线程**，不像HandlerThread类，ActivityThread并没有真正继承Thread类，只是往往运行在主线程，给人以线程的感觉，其实承载ActivityThread的主线程就是由Zygote fork而创建的进程。

**主线程的死循环一直运行是不是特别消耗CPU资源呢？** 其实不然，这里就涉及到**Linux pipe/epoll机制**，简单说就是在主线程的MessageQueue没有消息时，便阻塞在loop的queue.next()中的nativePollOnce()方法里，详情见[Android消息机制1-Handler(Java层)](https://link.zhihu.com/?target=http%3A//www.yuanhh.com/2015/12/26/handler-message-framework/%23next)，此时主线程会释放CPU资源进入休眠状态，直到下个消息到达或者有事务发生，通过往pipe管道写端写入数据来唤醒主线程工作。这里采用的epoll机制，是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步I/O，即读写是阻塞的。 **所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。**

#### **(3) Activity的生命周期是怎么实现在死循环体外能够执行起来的？**


ActivityThread的内部类H继承于Handler，通过handler消息机制，简单说Handler机制用于同一个进程的线程间通信。

**Activity的生命周期都是依靠主线程的Looper.loop，当收到不同Message时则采用相应措施：**
在H.handleMessage(msg)方法中，根据接收到不同的msg，执行相应的生命周期。

比如收到msg=H.LAUNCH_ACTIVITY，则调用ActivityThread.handleLaunchActivity()方法，最终会通过反射机制，创建Activity实例，然后再执行Activity.onCreate()等方法；
再比如收到msg=H.PAUSE_ACTIVITY，则调用ActivityThread.handlePauseActivity()方法，最终会执行Activity.onPause()等方法。 上述过程，我只挑核心逻辑讲，真正该过程远比这复杂。

**主线程的消息又是哪来的呢**？当然是App进程中的其他线程通过Handler发送给主线程，请看接下来的内容：

--------------------------------------------------------------------------------------------------------------------------------------
**最后，从进程与线程间通信的角度，****通过一张图****加深大家对App运行过程的理解：**
![](https://pic1.zhimg.com/80/7fb8728164975ac86a2b0b886de2b872_hd.jpg)

**system_server进程是系统进程**，java framework框架的核心载体，里面运行了大量的系统服务，比如这里提供ApplicationThreadProxy（简称ATP），ActivityManagerService（简称AMS），这个两个服务都运行在system_server进程的不同线程中，由于ATP和AMS都是基于IBinder接口，都是binder线程，binder线程的创建与销毁都是由binder驱动来决定的。

**App进程则是我们常说的应用程序**，主线程主要负责Activity/Service等组件的生命周期以及UI相关操作都运行在这个线程； 另外，每个App进程中至少会有两个binder线程 ApplicationThread(简称AT)和ActivityManagerProxy（简称AMP），除了图中画的线程，其中还有很多线程，比如signal catcher线程等，这里就不一一列举。

Binder用于不同进程之间通信，由一个进程的Binder客户端向另一个进程的服务端发送事务，比如图中线程2向线程4发送事务；而handler用于同一个进程中不同线程的通信，比如图中线程4向主线程发送消息。

**结合图说说Activity生命周期，比如暂停Activity，流程如下：**

1.  线程1的AMS中调用线程2的ATP；（由于同一个进程的线程间资源共享，可以相互直接调用，但需要注意多线程并发问题）

2.  线程2通过binder传输到App进程的线程4；

3.  线程4通过handler消息机制，将暂停Activity的消息发送给主线程；

4.  主线程在looper.loop()中循环遍历消息，当收到暂停Activity的消息时，便将消息分发给ActivityThread.H.handleMessage()方法，再经过方法的调用，最后便会调用到Activity.onPause()，当onPause()处理完后，继续循环loop下去。

## 进程间通信：binder机制考察
[深入理解Binder通信原理及面试问题](http://blog.csdn.net/happylishang/article/details/62234127)

### android binder机制的大体架构
### binder驱动提供的能力
### binderProxy binder实体关系
### binder传输数据的大小？如何解决跨进程传递大量数据？
 
Binder传输数据的大小限制

虽然APP开发时候，Binder对程序员几乎不可见，但是作为Android的数据运输系统，Binder的影响是全面性的，所以有时候如果不了解Binder的一些限制，在出现问题的时候往往是没有任何头绪，比如在Activity之间传输BitMap的时候，如果Bitmap过大，就会引起问题，比如崩溃等，这其实就跟Binder传输数据大小的限制有关系，在上面的一次拷贝中分析过，mmap函数会为Binder数据传递映射一块连续的虚拟地址，这块虚拟内存空间其实是有大小限制的，不同的进程可能还不一样。

普通的由Zygote孵化而来的用户进程，所映射的Binder内存大小是不到1M的，准确说是 1*1024*1024) - (4096 *2) ：这个限制定义在ProcessState类中，如果传输说句超过这个大小，系统就会报错，因为Binder本身就是为了进程间频繁而灵活的通信所设计的，并不是为了拷贝大数据而使用的：

```
#define BINDER_VM_SIZE ((1*1024*1024) - (4096 *2))

```

而在内核中，其实也有个限制，是4M，不过由于APP中已经限制了不到1M，这里的限制似乎也没多大用途：

```
static int binder_mmap(struct file *filp, struct vm_area_struct *vma)
{
    int ret;
    struct vm_struct *area;
    struct binder_proc *proc = filp->private_data;
    const char *failure_string;
    struct binder_buffer *buffer;
    //限制不能超过4M
    if ((vma->vm_end - vma->vm_start) > SZ_4M)
        vma->vm_end = vma->vm_start + SZ_4M;
    。。。
    }

```

有个特殊的进程ServiceManager进程，它为自己申请的Binder内核空间是128K，这个同ServiceManager的用途是分不开的，ServcieManager主要面向系统Service，只是简单的提供一些addServcie，getService的功能，不涉及多大的数据传输，因此不需要申请多大的内存：

```
int main(int argc, char **argv)
{
    struct binder_state *bs;
    void *svcmgr = BINDER_SERVICE_MANAGER;

        // 仅仅申请了128k
    bs = binder_open(128*1024);
 if (binder_become_context_manager(bs)) {
        ALOGE("cannot become context manager (%s)\n", strerror(errno));
        return -1;
    }

    svcmgr_handle = svcmgr;
    binder_loop(bs, svcmgr_handler);
    return 0;
}   

```

## view相关

### 自定义view的方式
 
[Android自定义View的实现方法，带你一步步深入了解View(四)](http://blog.csdn.net/sinyu890807/article/details/17357967)

![](http://ou21vt4uz.bkt.clouddn.com/interview/custom_view/flow_img/view_measure.png)

#### **1、自绘控件**

自绘控件的意思就是，这个View上所展现的内容全部都是我们自己绘制出来的。绘制的代码是写在onDraw()方法中的，而这部分内容我们已经在 [](http://blog.csdn.net/guolin_blog/article/details/12921889)[Android视图绘制流程完全解析，带你一步步深入了解View(二)](http://blog.csdn.net/guolin_blog/article/details/16330267) 中学习过了。

#### **2、组合控件**

组合控件的意思就是，我们并不需要自己去绘制视图上显示的内容，而只是用系统原生的控件就好了，但我们可以将几个系统原生的控件组合到一起，这样创建出的控件就被称为组合控件。

#### **3、继承控件**

继承控件的意思就是，我们并不需要自己重头去实现一个控件，只需要去继承一个现有的控件，然后在这个控件上增加一些新的功能，就可以形成一个自定义的控件了。这种自定义控件的特点就是不仅能够按照我们的需求加入相应的功能，还可以保留原生控件的所有功能，比如 [Android PowerImageView实现，可以播放动画的强大ImageView](http://blog.csdn.net/guolin_blog/article/details/11100315) 这篇文章中介绍的PowerImageView就是一个典型的继承控件。

### Listview的回收机制

[Bugly-Android ListView 与 RecyclerView 对比浅析--缓存机制](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653578065&idx=2&sn=25e64a8bb7b5934cf0ce2e49549a80d6&chksm=84b3b156b3c43840061c28869671da915a25cf3be54891f040a3532e1bb17f9d32e244b79e3f&scene=21#wechat_redirect)

### 描述view的绘制流程，view与window之间的关系？
 
[Android Activity 、 Window 、 View之间的关系](http://blog.csdn.net/u011733020/article/details/49465707)

 Activity就像是一扇贴着窗花的窗口，Window就想上窗口上面的玻璃，而View对象就像一个个贴在玻璃上的窗花。

最后的Activity与Window View的关联在画一个图3：

![](http://img.blog.csdn.net/20151028131646957?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

Activity会调用PhoneWindow的setContentView()将layout布局添加到DecorView上，而此时的DecorView就是那个最底层的View。然后通过LayoutInflater.infalte()方法加载布局生成View对象并通过addView()方法添加到Window上，（一层一层的叠加到Window上）所以，Activity其实不是显示视图，Window才是真正的显示视图。

注：一个Activity构造的时候只能初始化一个Window(PhoneWindow)，另外这个PhoneWindow有一个View容器 mContentParent，这个View容器是一个ViewGroup，是最初始的跟视图，然后通过addView方法将View一个个层叠到mContentParent上，这些层叠的View最终放在Window这个载体上面。

### PhoneWindow的实例在哪创建
 
PhoneWindow的实例，那么这个Window对象是在哪里 赋值的呢，我们在Activity中找到attach方法如下所示：
```
final void attach(Context context, ActivityThread aThread,  
            Instrumentation instr, IBinder token, int ident,  
            Application application, Intent intent, ActivityInfo info,  
            CharSequence title, Activity parent, String id,  
            NonConfigurationInstances lastNonConfigurationInstances,  
            Configuration config, String referrer, IVoiceInteractor voiceInteractor) {  
        attachBaseContext(context);  
  
        mFragments.attachHost(null /*parent*/);  
  
        mWindow = new PhoneWindow(this);  
        mWindow.setCallback(this);  
        mWindow.setOnWindowDismissedCallback(this);  
        mWindow.getLayoutInflater().setPrivateFactory(this);  
       .......
```

## 资源与动画体系

### 解释LayoutInflater的加载流程

 [Android LayoutInflater原理分析，带你一步步深入了解View(一)](http://blog.csdn.net/sinyu890807/article/details/12921889)

有两种方法可以获取到，第一种写法如下：

```
LayoutInflater layoutInflater = LayoutInflater.from(context);  
```
当然，还有另外一种写法也可以完成同样的效果：
```
  LayoutInflater layoutInflater = (LayoutInflater) context  .getSystemService(Context.LAYOUT_INFLATER_SERVICE);  
```
其实第一种就是第二种的简单写法，只是Android给我们做了一下封装而已。得到了LayoutInflater的实例之后就可以调用它的inflate()方法来加载布局了，如下所示：

` layoutInflater.inflate(resourceId, root);  `

inflate()方法一般接收两个参数，第一个参数就是要加载的布局id，第二个参数是指给该布局的外部再嵌套一层父布局，如果不需要就直接传null。这样就成功成功创建了一个布局的实例，之后再将它添加到指定的位置就可以显示出来了。

* 1\. 如果root为null，attachToRoot将失去作用，设置任何值都没有意义。

* 2\. 如果root不为null，attachToRoot设为true，则会给加载的布局文件的指定一个父布局，即root。

* 3\. 如果root不为null，attachToRoot设为false，则会将布局文件最外层的所有layout属性进行设置，当该view被添加到父view当中时，这些layout属性会自动生效。

* 4\. 在不设置attachToRoot参数的情况下，如果root不为null，attachToRoot参数默认为true。

### 系统如何加载合适的资源？

在activity内部访问资源（字符串，图片等）是很简单的，只要getResources然后就可以得到Resources对象，有了Resources对象就可以访问各种资源了。

* 不同的Context得到的都是同一份资源。

得到资源的方式为context.getResources，而真正的实现位于ContextImpl中的getResources方法，在ContextImpl中有一个成员 private Resources mResources，它就是getResources方法返回的结果，mResources的赋值代码为：
```
mResources = mResourcesManager.getTopLevelResources(mPackageInfo.getResDir(),
Display.DEFAULT_DISPLAY, null, compatInfo, activityToken);
```
下面看一下ResourcesManager的getTopLevelResources方法，这个方法的思想是这样的：在ResourcesManager中，所有的资源对象都被存储在ArrayMap中，首先根据当前的请求参数去查找资源，如果找到了就返回，否则就创建一个资源对象放到ArrayMap中。

* 有一点需要说明的是为什么会有多个资源对象.

原因很简单，因为res下可能存在多个适配不同设备、不同分辨率、不同系统版本的目录，按照android系统的设计，不同设备在访问同一个应用的时候访问的资源可以不同，比如drawable-hdpi和drawable-xhdpi就是典型的例子。

* ResourcesManager采用单例模式，这样就保证了不同的ContextImpl访问的是同一套资源.

注意，这里说的同一套资源未必是同一个资源，因为资源可能位于不同的目录，但它一定是我们的应用的资源，或许这样来描述更准确，在设备参数和显示参数不变的情况下，不同的ContextImpl访问到的是同一份资源。设备参数不变是指手机的屏幕和android版本不变，显示参数不变是指手机的分辨率和横竖屏状态。也就是说，尽管Application、Activity、Service都有自己的ContextImpl，并且每个ContextImpl都有自己的mResources成员，但是由于它们的mResources成员都来自于唯一的ResourcesManager实例，所以它们看似不同的mResources其实都指向的是同一块内存(C语言的概念)，因此，它们的mResources都是同一个对象（在设备参数和显示参数不变的情况下）。

在横竖屏切换的情况下且应用中为横竖屏状态提供了不同的资源，处在横屏状态下的ContextImpl和处在竖屏状态下的ContextImpl访问的资源不是同一个资源对象。

assets.addAssetPath(resDir)这句话的意思是把资源目录里的资源都加载到AssetManager对象中，具体的实现在jni中，大家感兴趣自己去了解下。而资源目录就是我们的res目录，当然resDir可以是一个目录也可以是一个zip文件。有没有想过，如果我们把一个未安装的apk的路径传给这个方法，那么apk中的资源是不是就被加载到AssetManager对象里面了呢？事实证明，的确是这样，具体情况可以参见[Android apk动态加载机制的研究（二）：资源加载和activity生命周期管理](http://blog.csdn.net/singwhatiwanna/article/details/23387079)这篇文章。addAssetPath方法的定义如下，注意到它的注释里面有一个{@hide}关键字，这意味着即使它是public的，但是外界仍然无法访问它，因为android sdk导出的时候会自动忽略隐藏的api，因此只能通过反射来调用。
```
/** 
 * Add an additional set of assets to the asset manager.  This can be 
 * either a directory or ZIP file.  Not for use by applications.  Returns 
 * the cookie of the added asset, or 0 on failure. 
 * {@hide} 
 */  
public final int addAssetPath(String path) {  
    int res = addAssetPathNative(path);  
    return res;  
}  
```

有了AssetManager对象后，我们就可以创建自己的Resources对象了，代码如下:
```
try {  
    AssetManager assetManager = AssetManager.class.newInstance();  
    Method addAssetPath = assetManager.getClass().getMethod("addAssetPath", String.class);  
    addAssetPath.invoke(assetManager, mDexPath);  
    mAssetManager = assetManager;  
} catch (Exception e) {  
    e.printStackTrace();  
}  
Resources currentRes = this.getResources();  
mResources = new Resources(mAssetManager, currentRes.getDisplayMetrics(),  
        currentRes.getConfiguration());
```
有了Resources对象，我们就可以通过Resources对象来访问里面的各种资源了，通过这种方法，我们可以完成一些特殊的功能，比如换肤、换语言包、动态加载apk等.

### 设计换肤方案？

#### **换肤介绍**

App换肤主要涉及的有页面中文字的颜色、控件的背景颜色、一些图片资源和主题颜色等资源。

为了实现换肤资源不与原项目混淆，尽量降低风险，可以将这些资源封装在一个独立的Apk资源文件中。在App运行时，主程序动态的从Apk皮肤包中读取相应的资源，无需Acitvity重启即可实现皮肤的实时更换，皮肤包与原安装包相分离，从而实现插件式换肤。

#### **换肤原理**

##### **1. 如何加载皮肤资源文件**
使用插件式换肤，皮肤资源肯定不会在被封装到主工程中，要怎么加载外部的皮肤资源呢？

先看下 Apk 的打包流程
![Apk 的打包流程](https://images2018.cnblogs.com/blog/561894/201711/561894-20171129112116909-329161900.png)

这里流程中，有两个关键点

* 1.R文件的生成

R文件是一个Java文件，通过R文件我们就可以找到对应的资源。R文件就像一张映射表，帮助我们找到资源文件。

* 2.资源文件的打包生成

资源文件经过压缩打包，生成 resources 文件，通过R文件找到里面保存的对映的资源文件。在 App 内部，我们一般通过下面代码，获取资源：

```
context.getResource.getString(R.string.hello);
context.getResource.getColor(R.color.black);
context.getResource.getDrawable(R.drawable.splash);
```
这个时获取 App 内部的资源，能我们家在皮肤资源什么思路吗？加载外部资源的 Resources 能通过类似的思路吗？
我们查看下 Resources 类的源码，发现 Resources 的构造函数

```
public Resources(AssetManager assets, DisplayMetrics metrics, Configuration config) {
      this(assets, metrics, config, CompatibilityInfo.DEFAULT_COMPATIBILITY_INFO);
  }
```
这里关键是第一个参数如何获取，第二和第三个参数可以通过 Activity 获取到。我们再去看下 AssetManager 的代码，同时会发现下面的这个

```
/**
     * Add an additional set of assets to the asset manager.  This can be
     * either a directory or ZIP file.  Not for use by applications.  Returns
     * the cookie of the added asset, or 0 on failure.
     * {@hide}
     */
  public final int addAssetPath(String path) {
        synchronized (this) {
            int res = addAssetPathNative(path);
            makeStringBlocks(mStringBlocks);
            return res;
        }
    }
```
AssetManager 可以加载一个zip 格式的压缩包，而 Apk 文件不就是一个 压缩包吗。我们通过反射的方法，拿到 AssetManager，加载 Apk 内部的资源，获取到 Resources 对象，这样再想办法，把 R文件里面保存的ID获取到，这样既可以拿到对应的资源文件了。理论上我们的思路时成立的。
我们看下，如何通过代码获取 Resources 对象。
```
AssetManager assetManager = AssetManager.class.newInstance();
Method addAssetPath = assetManager.getClass().getMethod("addAssetPath", String.class);
addAssetPath.invoke(assetManager, skinPkgPath);
 
Resources superRes = context.getResources();
Resources skinResource = new Resources(assetManager,superRes.getDisplayMetrics(),superRes.getConfiguration());
```
##### **2.如何标记需要换肤的View**
找到资源文件之后，我们要接着标记需要换肤的 View 。

找到需要换肤的 View
怎么寻找哪些是我们要关注的 View 呢？ 我们还是重 View 的创建时机寻找机会。我们添加一个布局文件时，会使用 LayoutInflater的 Inflater方法，我们看下这个方法是怎么讲一个View添加到Activity 中的。
LayoutInflater 中有个接口
```
public interface Factory {
        /**
         * Hook you can supply that is called when inflating from a LayoutInflater.
         * You can use this to customize the tag names available in your XML
         * layout files.
         *
         * <p>
         * Note that it is good practice to prefix these custom names with your
         * package (i.e., com.coolcompany.apps) to avoid conflicts with system
         * names.
         *
         * @param name Tag name to be inflated.
         * @param context The context the view is being created in.
         * @param attrs Inflation attributes as specified in XML file.
         *
         * @return View Newly created view. Return null for the default
         *         behavior.
         */
        public View onCreateView(String name, Context context, AttributeSet attrs);
    }
```
根据这里的注释描述，我们可以自己实现这个接口，在 onCreateView 方法中选择我们需要标记的View，根据 AttributeSet 值，过滤不需要关注的View。

**标记 View 与对应的资源**

我们在 View 创建时，通过过滤 Attribute 属性，找到我们要标记的 View ，下面我们就把这些View的属性记下来
```
for (int i = 0; i < attrs.getAttributeCount(); i++){
            String attrName = attrs.getAttributeName(i);
            String attrValue = attrs.getAttributeValue(i);
            if(!AttrFactory.isSupportedAttr(attrName)){
                continue;
            } 
            if(attrValue.startsWith("@")){
                try {
                    int id = Integer.parseInt(attrValue.substring(1));
                    String entryName = context.getResources().getResourceEntryName(id);
                    String typeName = context.getResources().getResourceTypeName(id);
                    SkinAttr mSkinAttr = AttrFactory.get(attrName, id, entryName, typeName);
                    if (mSkinAttr != null) {
                        viewAttrs.add(mSkinAttr);
                    }
                } catch (NumberFormatException e) {
                    e.printStackTrace();
                } catch (NotFoundException e) {
                    e.printStackTrace();
                }
            }
        }
```
然后把这些 View 和属性值，一起封装保存起来
```
if(!ListUtils.isEmpty(viewAttrs)){
            SkinItem skinItem = new SkinItem();
            skinItem.view = view;
            skinItem.attrs = viewAttrs;
            mSkinItems.add(skinItem);
            if(SkinManager.getInstance().isExternalSkin()){
                skinItem.apply();
            }
    }
```
##### **3.如何做到及时更新UI**
由于我们把需要更新的View 以及属性值都保存起来了，更新的时候只要把他们取出来遍历一遍即可。
```
@Override
    public void onThemeUpdate() {
        if(!isResponseOnSkinChanging){
            return;
        }
        mSkinInflaterFactory.applySkin();
    }
```
//applySkin 的具体实现

```
public void applySkin(){
        if(ListUtils.isEmpty(mSkinItems)){
            return;
        }  
        for(SkinItem si : mSkinItems){
            if(si.view == null){
                continue;
            }
            si.apply();
        }
    }
```
##### **4.如何制作皮肤包**
皮肤包制作相对简单

* 1.创建独立工程 model，包名任意。

* 2.添加资源文件到 model 中,不需要 java 代码

* 3.运行 build.gradle 脚本，打包命令，生成apk文件，修改名称为 xxx.skin 皮肤包即可。

### bitmap内存占用的计算方式，如何优化加载图片

Android中一张图片（BitMap）占用的内存主要和以下几个因数有关：图片长度，图片宽度，单位像素占用的字节数。

一张图片（BitMap）占用的内存=图片长度x图片宽度x单位像素占用的字节数 

注：图片长度和图片宽度的单位是像素。

**图片格式一个像素占用字节** 

Alpha_8 ： 1 
Kindex ： 1 
RGB_565 ： 2
ARGB_4444 ： 2 
RGBA_8888 ： 4 
BGRA_8888 ： 4

* 1.首先计算scaledWidth和scaledHeight(源码中计算内存的需要的宽高) 
scaledWidth=int(图片宽度*手机屏幕密度/图片文件夹(hdpi)+ 0.5) 
scaledHeight=int(图片高度*手机屏幕密度/图片文件夹(hdpi)+ 0.5) 

* 2.内存计算 
total=scaledWidth*scaledHeight*占用字节例如:一个500*800的图片,图片格式为RGBA_8888格式,放在xhdpi目录下,在小米6上所占内存是 
int( 500 * 420/ 480f + 0.5) *int( 800 * 420/ 480f + 0.5) *4=1227276B

另外，需要注意这里的图片占用内存是指在Navtive中占用的内存，当然BitMap使用的绝大多数内存就是该内存。 因为我们可以简单的认为它就是BitMap所占用的内存。 Bitmap对象在不使用时,我们应该先调用recycle()，然后才它设置为null. 虽然Bitmap在被回收时可以通过BitmapFinalizer来回收内存。但是调用recycle()是一个良好的习惯.

在Android4.0之前，Bitmap的内存是分配在Native堆中，调用recycle()可以立即释放Native内存。 从Android4.0开始，Bitmap的内存就是分配在dalvik堆中，即JAVA堆中的，调用recycle()并不能立即释放Native内存。但是调用recycle()也是一个良好的习惯。

### 优化加载图片

#### 1、OOM 引起与表现

在 Android 这种移动设备上，如果代码没有处理好，很容易引发内存持续占用与泄漏，导致 `OOM（OutOfMemoryError）` 异常，进而导致 App 程序 Crash 挂掉。

在 Android 开发中，一个典型的 OOM 异常如下：

![OOM 异常](http://img.blog.csdn.net/20160307093435207)

一旦碰上了这类错误，我们往往需要去排查内存了。导致 OOM 的一些情况比较常见，大多数情况下，大家可能遇到的都是同一种情况：

*   Activity 泄漏导致；
*   层次庞大复杂的 View 视图导致；
*   大量图片持续占用导致；
*   其他资源持续未释放导致。

#### 2、Android Studio 查看内存占用

在 Android Studio 里面，我们可以在 `Monitors` 窗口中，实时对 App 内存进行监控，我们可以看出 App 的 `HeapSize` 、`已经使用的内存大小`、`剩余内存大小`以及`峰值变化`。有了这些信息，我们可以在某个页面打开和关闭时进行监控，从而对比该页面占用内存变化，可以很方便的定位问题。

![Monitors 查看内存占用](http://img.blog.csdn.net/20160307093451613)

在这张图中，如果已使用的内存大小（`Allocated`）接近到 `HeapSize` 的大小，App 将会处于非常危险的状态中，很有可能下一个操作就会直接导致 OOM，通过 Android Studio，我们可以防患于未然，在 Debug 阶段进行预防。

#### 3、adb 查看内存占用

`adb` 工具也是一个非常有用的工具，我们可以通过它来查看 App 内存占用。

##### 3.1、查看 JVM 的 `HeapSize` 等参数

通过命令 `adb shell getprop dalvik.vm.heapsize` 可以直接查看 Dalvik 虚拟机为 App 规定的最大 `HeapSize`：

![getprop dalvik.vm.heapsize](http://img.blog.csdn.net/20160307093512421)

一般来说，App 可达到的最大 HeapSize 为 `dalvik.vm.heapgrowthlimit` 所规定的大小。但是如果我们在 `AndroidManifest.xml` 中为 Application 添加 `android:largeHeap="true"` 属性，App 可达到的最大 HeapSize 则被调整为 `dalvik.vm.heapsize` 规定的值。

虽然添加 `android:largeHeap="true"` 属性将大大降低 OOM 的概率，但除非万不得已的情况下，否则不要使用该属性。出现 OOM 后，我们首先应该排查整个 App，找出内存瓶颈予以解决。

##### 3.2、查看 App 内存占用

通过命令 `adb shell dumpsys meminfo [package_name]` 可以查看 App 所占用内存：

![dumpsys meminfo 查看内存占用](http://img.blog.csdn.net/20160307093539708)

通过这个命令，App 所占资源情况一目了然，甚至我们可以看到整个 App 中 View 个数、Activity 个数——这对于排查 Activity 泄漏和优化 View 层级也是非常有帮助的。

#### 4、图片加载导致 OOM

而在一个 App 中，图片处理不恰当往往是 OOM 错误出现的元凶——因为 App 中所有图片动辄占用几十 M 的内存。如果我们能优先着手排查这一块，将会对 App 的内存优化带来 `最直接最明显` 的改观。而图片的不恰当处理操作一般有如下一些：

*   直接加载 `超大尺寸` 图片；
*   图片加载后 `未及时释放`；
*   在页面中，同时加载 `非常多` 的图片；

##### 4.1、超大尺寸图片处理

现在的手机摄像头像素比较高，摄制出来的照片尺寸非常大，比如在一款还算老旧的手机上面，拍摄的图片尺寸竟然达到了 `2368 x 4224`！因为采用 jpeg 格式的缘故，这张图片在磁盘上才1.9M，但如果我们不加任何处理，按原尺寸加载到内存中，占用的内存将会非常可观。

所以，针对大图的加载，比较常用的方法是进行 `DownSampling(向下采样)`，许多博客或技术站点对该方案有详细的描述，在此不再赘述，简单原理用代码表述如下：

```java
public static int calcInSampleSize(
        int width, int height, int requestWidth, int requestHeight) {

    int inSampleSize = 1;
    if (requestWidth <= 0 || requestHeight <= 0) {
        return inSampleSize;
    }

    if (width > requestWidth || height > requestHeight) {
        int widthRatio = Math.round((float) width / (float) requestWidth);
        int heightRatio = Math.round((float) height / (float) requestHeight);

        inSampleSize = Math.min(widthRatio, heightRatio);
    }

    return inSampleSize;
}

public static Bitmap decodeBitmapFromUri(
    Context context, Uri uri, int requestWidth, int requestHeight) {

    BitmapFactory.Options options = getResourceOptions(context, uri);
    options.inSampleSize = calcInSampleSize(
            options.outWidth, options.outHeight, requestWidth, requestHeight);
    options.inJustDecodeBounds = false;

    // ...

    ContentResolver resolver = context.getContentResolver();
    Bitmap bitmap = BitmapFactory.decodeStream(
            resolver.openInputStream(uri), new Rect(), options);
    // ...

    return bitmap;
}
```

这样，如果我们要加载一张图片到 View 上，我们可以通过 `view.getMeasuredWidth() 和 view.getMeasuredHeight()` 得到 View 的宽和高，然后按这个大小进行采样，得到的 Bitmap 将会是尺寸适合的图片，不会占用额外内存，图片在 View 上展示出来质量也比较高。

##### 4.2、及时释放图片

一般不要静态缓存图片，就算有缓存，也可以结合 **`LRU`** 机制来保证缓存图片的个数和占用内存。Android SDK 已经提供了 `LruCache` 类来实现 LRU 机制。

##### 4.3、避免同时加载大量图片

避免同一时间加载大量的图片，也可以为我们的内存优化提供不小的收益。比如，在一个 `ScrollView` 中有非常多的 ImageView，这时候，占用的内存往往非常客观，因为就算一些 View 我们在屏幕视野里面看不到，它还是持续占用内存。我们可以通过 `RecyclerView` 或者 `ListView` 来予以替换，从而达到内存优化的效果。

在我的开发过程中，就遇到了这样一个例子。一个页面用 ScrollView 来布局，里面有 26 张左右的图片，这时候，整个 App 的内存占用长期达到了 `90M` 左右！一直徘徊在 OOM 边缘。在我把这个页面用 RecyclerView 替换掉 ScrollView 后，整个 App 内存竟然下降了 `40M` 之多！！！整个 App 变得非常顺滑。

#### 5、采用开源库加载图片

现在已经有非常多的图片加载库供我们使用了，比较流行的有：`Fresco`、`Universal-Image-Loader`、`Picasso`、`Volley` 等等。这些开源库一般来说，对内存的优化已经比较全面了，比我们自己手工管理内存来的好。所以，可以根据项目的实际情况灵活选用。

比如，我目前所使用的 `Fresco` 库，就可以灵活设定图片尺寸，避免加载大尺寸的图片（`setResizeOptions`）：

```java
public static void displayImage(DraweeView draweeView, Uri uri) {
    Size size = getAppropriateSize(draweeView);

    ImageRequest request = ImageRequestBuilder
            .newBuilderWithSource(uri)
            .setResizeOptions(new ResizeOptions(size.mWidth, size.mHeight))
            .setAutoRotateEnabled(true)
            .build();

    DraweeController controller = Fresco.newDraweeControllerBuilder()
            .setUri(uri)
            .setImageRequest(request)
            .setOldController(draweeView.getController())
            .build();

    draweeView.setController(controller);
}

private static Size getAppropriateSize(View view) {
    int width = view.getMeasuredWidth();
    int height = view.getMeasuredHeight();

    if (width <= 0 || height <= 0) {
        width = view.getWidth();
        height = view.getHeight();
    }

    Size size = MiscUtils.getScreenSize();
    if (width <= 0 || height <= 0 || width > size.mWidth || height > size.mHeight) {
        width = size.mWidth;
        height = size.mHeight;
    }

    return new Size(width, height);
}
```

当然，我们还要在 `ImagePipelineConfig` 中开启 `DownSampling`（`setDownsampleEnabled(true)`）：

```java

public static void initFresco(Context context) {
    ImagePipelineConfig config = ImagePipelineConfig
            .newBuilder(context)
            .setDownsampleEnabled(true)
            .build();

    Fresco.initialize(context, config);
}
```

### Android常用动画原理，是否接触过物理动画？是否了解google新出的Spring动画库

#### Android常用动画原理

* 1、Tween动画,就是对场景里的对象不断的进行图像变化来产生动画效果(旋转、平移、放缩和渐变);
* 2、Frame动画,即顺序的播放事先做好的图像,与gif图片原理类似;
* 3、属性动画,改变对象的实际属性达到动画效果。

#### 三种动画的优缺点：

**(1)Frame Animation(帧动画)** 主要用于播放一帧帧准备好的图片，类似GIF图片，优点是使用简单方便、缺点是需要事先准备好每一帧图片；

**(2)Tween Animation(补间动画)** 仅需定义开始与结束的关键帧，而变化的中间帧由系统补上，优点是不用准备每一帧，缺点是只改变了对象绘制，而没有改变View本身属性。因此如果改变了按钮的位置，还是需要点击原来按钮所在位置才有效。

**(3)Property Animation(属性动画)** 是3.0后推出的动画，优点是使用简单、降低实现的复杂度、直接更改对象的属性、几乎可适用于任何对象而仅非View类，缺点是需要3.0以上的API支持，限制较大！但是目前国外有个开源库，可以提供低版本支持！

#### Spring动画库

[Android 中基于物理特性的动画简介](https://zhuanlan.zhihu.com/p/28239508)

**简评**:基于物理特性的动画依赖于物理学定律，这能在动画中表现出高度的现实感。

##### **什么是基于物理的动画？**

*   这是一种遵循物理学定律的动画形式
*   能够依据加速度和速度去计算和更新每一帧的动画数值
*   当受力平衡时，动画为处于恒定运动或静止状态

##### **和普通动画有什么不一样？**

*   自然：动画模仿实时的动作，看起来更自然
*   反应：当目标值发生变化时，动画保持动量（速度）并以更平滑的运动结束
*   较少的视觉干扰：动画看起来更流畅，反应灵敏度也更高

考虑在动画期间目标值需要改变的情况。

## 网络安全

#### https原理
#### 什么是 HTTPS?

HTTPS (基于安全套接字层的超文本传输协议 或者是 HTTP over SSL) 是一个 Netscape 开发的 Web 协议。

你也可以说：HTTPS = HTTP + SSL

HTTPS 在 HTTP 应用层的基础上使用安全套接字层作为子层。

#### 为什么需要 HTTPS ？

超文本传输协议 (HTTP) 是一个用来通过互联网传输和接收信息的协议。HTTP 使用请求/响应的过程，因此信息可在服务器间快速、轻松而且精确的进行传输。当你访问 Web 页面的时候你就是在使用 HTTP 协议，但 HTTP 是不安全的，可以轻松对窃听你跟 Web 服务器之间的数据传输。在很多情况下，客户和服务器之间传输的是敏感歇息，需要防止未经授权的访问。为了满足这个要求，网景公司(Netscape)推出了[HTTPS](http://www.nowamagic.net/librarys/veda/tag/https)，也就是基于安全套接字层的 HTTP 协议。

#### HTTP 和 HTTPS 的相同点

大多数情况下，HTTP 和 HTTPS 是相同的，因为都是采用同一个基础的协议，作为 HTTP 或 HTTPS 客户端——浏览器，设立一个连接到 Web 服务器指定的端口。当服务器接收到请求，它会返回一个状态码以及消息，这个回应可能是请求信息、或者指示某个错误发送的错误信息。系统使用统一资源定位器 URI 模式，因此资源可以被唯一指定。而 HTTPS 和 HTTP 唯一不同的只是一个协议头(https)的说明，其他都是一样的。

#### HTTP 和 HTTPS 的不同之处

1.  HTTP 的 URL 以 http:// 开头，而 HTTPS 的 URL 以 https:// 开头
2.  HTTP 是不安全的，而 HTTPS 是安全的
3.  HTTP 标准端口是 80 ，而 HTTPS 的标准端口是 443
4.  在 OSI 网络模型中，HTTP 工作于应用层，而 HTTPS 工作在传输层
5.  HTTP 无需加密，而 HTTPS 对传输的数据进行加密
6.  HTTP 无需证书，而 HTTPS 需要认证证书

#### HTTPS 如何工作?

使用 HTTPS 连接时，服务器要求有公钥和签名的证书。

当使用 https 连接，服务器响应初始连接，并提供它所支持的加密方法。作为回应，客户端选择一个连接方法，并且客户端和服务器端交换证书验证彼此身份。完成之后，在确保使用相同密钥的情况下传输加密信息，然后关闭连接。为了提供 https 连接支持，服务器必须有一个公钥证书，该证书包含经过证书机构认证的密钥信息，大部分证书都是通过第三方机构授权的，以保证证书是安全的。

换句话说，HTTPS 跟 HTTP 一样，只不过增加了 [SSL](http://www.nowamagic.net/librarys/veda/tag/SSL)。

HTTP 包含如下动作：

1.  浏览器打开一个 TCP 连接
2.  浏览器发送 HTTP 请求到服务器端
3.  服务器发送 HTTP 回应信息到浏览器
4.  TCP 连接关闭

SSL 包含如下动作：

1.  验证服务器端
2.  允许客户端和服务器端选择加密算法和密码，确保双方都支持
3.  验证客户端(可选)
4.  使用公钥加密技术来生成共享加密数据
5.  创建一个加密的 SSL 连接
6.  基于该 SSL 连接传递 HTTP 请求

#### 什么时候该使用 HTTPS?

银行网站、支付网关、购物网站、登录页、电子邮件以及一些企业部门的网站应该使用 HTTPS，例如：

*   PayPal: https://www.paypal.com
*   Google AdSense: https://www.google.com/adsense/

如果某个网站要求你填写信用卡信息，首先你要检查该网页是否使用 https 加密连接，如果没有，那么请不要输入任何敏感信息如信用卡号。

## 设计API接口如何保证安全调用？

[设计API接口如何保证安全调用](https://pxfile.github.io/2018/03/19/%E8%AE%BE%E8%AE%A1API%E6%8E%A5%E5%8F%A3%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E5%AE%89%E5%85%A8%E8%B0%83%E7%94%A8/)

## App构建与启动流程

### Dalvik/ART下，apk的安装与加载流程
![Apk 的打包流程](https://images2018.cnblogs.com/blog/561894/201711/561894-20171129112116909-329161900.png)
 
[ Dalvik ART下，apk的安装与加载流程](https://pxfile.github.io/2018/03/19/Dalvik-ART%E4%B8%8B-apk%E7%9A%84%E5%AE%89%E8%A3%85%E4%B8%8E%E5%8A%A0%E8%BD%BD%E6%B5%81%E7%A8%8B/)

### Android 的Contex划分，
 
[Android Context完全解析，你所不知道的Context的各种细节](http://blog.csdn.net/sinyu890807/article/details/47028975)

#### Context是个抽象类，我们可以直接通过看其类结构来说明答案：

![](http://img.blog.csdn.net/20150104163328895?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG1qNjIzNTY1Nzkx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

可以看到Activity、Service、Application都是Context的子类；

也就是说，Android系统的角度来理解：Context是一个场景，代表与操作系统的交互的一种过程。从程序的角度上来理解：Context是个抽象类，而Activity、Service、Application等都是该类的一个实现。

在仔细看一下上图：Activity、Service、Application都是继承自ContextWrapper，而ContextWrapper内部会包含一个base context，由这个base context去实现了绝大多数的方法。

#### 2、Context与ApplicationContext

XXXActivity和getApplicationContext返回的肯定不是一个对象，一个是当前Activity的实例，一个是项目的Application的实例。既然区别这么明显，那么各自的使用场景肯定不同，乱使用可能会带来一些问题。

#### 3、引用的保持

大家在编写一些类时，例如工具类，可能会编写成单例的方式，这些工具类大多需要去访问资源，也就说需要Context的参与。这个引用就用ApplicationContext，它的生命周期和我们的单例对象一致。
有人会说，我们可以软引用，嗯，软引用，假如被回收了，你不怕NullPointException么。

#### 4、Context的应用场景

![](http://img.blog.csdn.net/20150104183450879)

#### 5.Context数量

那么一个应用程序中到底有多少个Context呢？其实根据上面的Context类型我们就已经可以得出答案了。Context一共有Application、Activity和Service三种类型，因此一个应用程序中Context数量的计算公式就可以这样写：

`
  Context数量 = Activity数量 + Service数量 + 1  
`

上面的1代表着Application的数量，因为一个应用程序中可以有多个Activity和多个Service，但是只能有一个Application。

#### Application中方法的执行顺序如下图所示：

![](http://img.blog.csdn.net/20151108174114045?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### App的启动流程与生命周期的回调
[框架层理解Activity生命周期(APP启动过程)](http://blog.51cto.com/laokaddk/1206840)

### ANR的产生流程，如何监控？如何分析ANR文件

#### ANR分析 获取ANR产生的trace文件，分析traces.txt


ANR产生时, 系统会生成一个traces.txt的文件放在/data/anr/下. 可以通过[adb](https://www.jianshu.com/p/5980c8c282ef)命令将其导出到本地:

```
$adb pull data/anr/traces.txt .

```
trace信息中的**添加的中文注释**已基本说明了trace文件该怎么分析:

* 文件最上的即为最新产生的ANR的trace信息.
* 前面两行表明ANR发生的进程pid, 时间, 以及进程名字(包名).
* 寻找我们的代码点, 然后往前推, 看方法调用栈, 追溯到问题产生的根源.

#### ANR原因
*   主线程被IO操作（从4.0之后网络IO不允许在主线程中）阻塞。
*   主线程中存在耗时的计算
*   主线程中错误的操作，比如Thread.wait或者Thread.sleep等 Android系统会监控程序的响应状况，一旦出现下面两种情况，则弹出ANR对话框
*   应用在5秒内未响应用户的输入事件（如按键或者触摸）
*   BroadcastReceiver未在10秒内完成相关的处理
*   Service在特定的时间内无法处理完成 20秒

*   使用AsyncTask处理耗时IO操作。
*   使用Thread或者HandlerThread时，调用Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND)设置优先级，否则仍然会降低程序响应，因为默认Thread的优先级和主线程相同。
*   使用Handler处理工作线程结果，而不是使用Thread.wait()或者Thread.sleep()来阻塞主线程。
*   Activity的onCreate和onResume回调中尽量避免耗时的代码
*   BroadcastReceiver中onReceive代码也要尽量减少耗时，建议使用IntentService处理。


## 设计模式：

1.用到得设计模式结合项目举例
2.代理模式：动态代理静态代理，动态代理在框架设计中的使用


## 性能与优化：
1.OOM分析方式，如何监控与定位？
2.如何避免OOM？
3.CPU，内存，布局等的优化经验

[性能优化](https://pxfile.github.io/2018/03/08/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/)

## 项目架构相关：
### 组件化
### 插件化
### 热修复

### multidex，分包
[Android分包MultiDex源码分析](http://blog.csdn.net/shensky711/article/details/52845661)
[dex分包变形记](https://segmentfault.com/a/1190000004053072)

#### MultiDex的产生背景

当Android系统安装一个应用的时候，有一步是对Dex进行优化，这个过程有一个专门的工具来处理，叫DexOpt。DexOpt的执行过程是在第一次加载Dex文件的时候执行的。这个过程会生成一个ODEX文件，即Optimised Dex。执行ODex的效率会比直接执行Dex文件的效率要高很多。

但是在早期的Android系统中，DexOpt有一个问题，DexOpt会把每一个类的方法id检索起来，存在一个链表结构里面。但是这个链表的长度是用一个short类型来保存的，导致了方法id的数目不能够超过65536个。当一个项目足够大的时候，显然这个方法数的上限是不够的。尽管在新版本的Android系统中，DexOpt修复了这个问题，但是我们仍然需要对低版本的Android系统做兼容。

为了解决方法数超限的问题，需要将该dex文件拆成两个或多个，为此谷歌官方推出了multidex兼容包，配合AndroidStudio实现了一个APK包含多个dex的功能。

* * *

#### MultiDex的简要原理

我们以APK中有两个dex文件为例，第二个dex文件为classes2.dex。

1.  兼容包在Applicaion实例化之后，会检查系统版本是否支持 multidex，classes2.dex是否需要安装。
2.  如果需要安装则会从APK中解压出classes2.dex并将其拷贝到应用的沙盒目录下。
3.  通过反射将classes2.dex注入到当前的classloader中。

### 打包与CI

[android基于Android Studio的持续集成CI](http://blog.csdn.net/liuwenhan999/article/details/51484139)

Android基于AS自动化编译并发送邮件
记录[AndroidStudio][6] 自动化编译，并发送邮件出来

#### 脚本流程
* 配置编译环境
* 拉取服务器最新的代码（此处可以是git，或者是svn）
* 配置服务器代码为as编译环境的目录结构
* 配置编译版本号版本tag等
* 执行as编译脚本
* 邮件发送结果（邮件内容读取最近一天svnlog日志，as编译版本为附件）

#### 环境配置
as环境安装 AndroidStudio1.4或以上
python python2.7
bat windows 批处理
svn svn版本最新即可
j dk jdk1.6
adt as自带

#### 项目初衷
由于习惯用eclipse开发，但是每次都需要手动打包版本给测试人员测试，很麻烦所以尝试下as的自动化编译功能 
编写过程中又能学习下python，gradle，bat等多种脚步混合编写，尽量做到不依赖某一个模块

由于习惯用eclipse开发，所以只是在eclipse里面开发后提交到svn，然后每天编译一个日版本发邮件出来

首先还是要使用as IDE新建一个android的project，并且添加需要编译的项目进来，包括多依赖的项目

在使用as构建编辑目录结构的时候有几点比较坑 

* 1.就是libs下jar，和so的目录结构和在eclipse里面不一样（百度有） 
* 2.gradle编译脚本里面so和jar的添加方法也不一样（百度有） 
* 3.发邮件模块是使用的python发送的邮件，邮箱注册的outlook的 
outlook需要手机认证不然容易发送失败被当垃圾邮件

#### 配置目录代码
下载并配置as编译目录结构：

::svn下载最新代码
rm -rf tmp
svn co https://svn.url_url tmp

::初始化路径
set lll=%~dp0kasfaandroid_lbxx\
set moudle=src
rm -rf %moudle%
rm -rf libs
rm -rf build
mkdir %moudle%
mkdir %moudle%\main\
mkdir %moudle%\main\java\
mkdir %moudle%\main\res\
mkdir %moudle%\main\assets\
mkdir %moudle%\main\jniLibs\
mkdir libs\

::copy代码到编译目录
xcopy %lll%tmp\src %lll%%moudle%\main\java\ /e /q
xcopy %lll%tmp\assets %lll%%moudle%\main\assets\ /e /q
xcopy %lll%tmp\res %lll%%moudle%\main\res\ /e /q
xcopy %lll%tmp\libs %lll%%moudle%\main\jniLibs\ /e /q

::删除libs目录下面所有的jar文件，不删除so的文件夹
rm -rf %lll%%moudle%\main\jniLibs\*.*
copy %lll%tmp\AndroidManifest.xml %lll%%moudle%\main\AndroidManifest.xml

xcopy %lll%tmp\libs %lll%libs\ /e /q
::删除libs目录下面所有的文件夹，不包括jar
python %DirCorelibs%py_remove_all_dir.py %lll%libs

由于项目里面是多个lib依赖，编译目录都一样，此处就不贴代码

编译脚本
编译脚本是初始化脚本添加编译时间tag标注到app

::初始化编译脚本
rm -rf build.gradle
copy %DirCorelibs%build.gradle.model build.gradle

set currentHour=%time:~0,2%
if "%time:~0,1%"==" " set currentHour=0%time:~1,1%
@echo %date:~0,4%%date:~5,2%%date:~8,2%_%currentHour%%time:~3,2%%time:~6,2%

::替换版本号和版本
python %DirCorelibs%replaceStrTools.py %versionCode% versionCodeValue build.gradle
python %DirCorelibs%replaceStrTools.py %versionName% versionNameValue build.gradle
python %DirCorelibs%replaceStrTools.py %date:~0,4%%date:~5,2%%date:~8,2%_%currentHour%%time:~3,2%%time:~6,2% ChannelValue build.gradle

::clean，然后编译代码
rm -rf buildlog.txt
call gradle clean
call gradle build -q  2>> buildlog.txt

#### 邮件发送编译结果
编译结果判断编译的releaseapk是否存在，存在表示编译成功，否则读取as编译产生的错误日志发送邮件出来：


setlocal enabledelayedexpansion
set mydd=%date:~0,4%
set /a mydd=!mydd!+1
svn log https://svn.url_url  -r {%date:~0,4%-%date:~5,2%-%date:~8,2%}:{%mydd%-%date:~5,2%-%date:~8,2%} -v  >> log.txt

::打包好的apk发邮件出去
if not exist build\outputs\apk\vtest.apk goto nofile
goto sendmail
:nofile
::编译失败发送邮件告诉主负责人
python %DirCorelibs%sendmail.py build\outputs\apk\vtest.apk vtest.apk xxxxxxxx@qq.com SFA_Build_Faild_%versionName%_%versionCode%_%date:~0,4%%date:~5,2%%date:~8,2%_%currentHour%%time:~3,2%%time:~6,2%  buildlog.txt 

goto endsendmail
:sendmail
::编译成功发送邮件告诉主负责人
python %DirCorelibs%sendmail.py build\outputs\apk\vtest.apk vtest.apk xxxxxxx@qq.com SFA_L_%versionName%_%versionCode%_%date:~0,4%%date:~5,2%%date:~8,2%_%currentHour%%time:~3,2%%time:~6,2%  log.txt 

:endsendmail

@echo end_send_email_%time:~0,8%

#### 邮件结果
最终可以写一个at命令处理每天晚上编译一个日版本给到测试人员 
邮件截图如下 
邮件明细

邮件列表
![](http://img.blog.csdn.net/20160822123740544)
补上web前端展示编译结果页面
![](http://img.blog.csdn.net/20161021150257017)
补上web前端展示编译结果页面


