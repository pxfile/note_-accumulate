##Handler机制？ MessageQueue，Looper等。

* 3.1 一个线程是否只有一个Looper？
* 3.2 如何保证一个线程只有一个Looper？
*  参考：ThreadLocal

Android的消息机制其实在android的开发过程中指的也就是Handler的运行机制，这也就引出了android中常见的面试问题：

>简述Handler、Looper、MessageQueue的含义，以及它们之间的关系
简述Handler的运行机制
说明Handler、Looper以及Message之间的关系
Handler机制为什么这么重要呢？

我们知道android设备作为一台移动设备，不管是内存或者还是它的性能都会受到一定的限制：过多的使用内存会使内存溢出（OOM）；另外一方面，大量的使用CPU的资 源，一般是指CPU做了大量耗时的操作，会导致手机变得很卡甚至会出现程序无法响应的情况，即出现ANR异常（Application Not Responding）。ANR异常对于我们来说的话其实并不陌生：在UI线程中如果5秒之内没有相应的输入事件或者是BroadcastReceiver中超过10秒没有完成返回的话就会触发ANR异常，这样就要求我们必须要在写代码的时候格外注意不能将大量耗时的操作放在UI线程中去执行，例如网络访问、数据库操作、读取大容量的文件资源、分析位图资源等…

>Handler机制的主要作用

Android系统的这些特性导致开发者在进行开发的时候必须要将比较耗时的操作远离UI线程（ActivityThread），单独的将这些操作放入到子线程中进行执行，在子线程完成代码执行之后将等到的结果数据返回到UI线程（android系统规定在子线程中不允许访问UI），同时UI线程根据这些返回的数据更新界面UI，从而实现了人机之间友好的交互。

那么对于以上的结论，我们会发现如下的问题：

在子线程执行完毕的代码如何才能将数据返回到UI线程呢？
在这个过程中，android系统到底都做了些什么呢？
接下来我们就开始介绍Handler的具体的运行机制

>Handler机制简介

Handler运行机制是需要MessageQueue、Looper、Handler三者的相互协调工作，但是实际上这三者就是一个整体，只不过我们在开发的过程中只是接触到了Handler一个而已。Handler的主要作用就是将一个任务切换到指定的线程中去执行。而这样的特性正好可以用来解决在子线程中无法访问UI的矛盾。 
那么接下来我们就来了解下MessageQueue、Looper、Handler的运行机制，以及它们最基本的运行原理。

>Handler机制之MessageQueue介绍

MessageQueue通常翻译为“消息队列”，它内部存储了一组数据，以队列的形式对外提供插入和删除操作。虽然被称之为消息队列，但是实际上它的数据结构却是采用的单链表的结构来存储消息列表（单链表在插入和删除操作上效率比较高）。 
MessageQueue主要包含两个操作：插入（enqueueMessage）和读取（next）。

插入：enqueueMessage()方法是往消息队列中插入一条数据
读取：next()方法用于消息的读取，读取操作本身也会伴随着消息的删除操作，即从消息队列中取出一条数据并将该数据从消息队列中删除。
对于enqueueMessage和next方法的源码请读者去自行查看，因为这两个方法占得篇幅较大，在这里就不贴出来了，本处就说明下这两个方法的主要作用。需要指出的是，next方法是一个无限循环的方法，如果消息队列中没有消息，那么next方法会一直堵塞，有新消息的事后，next方法会返回这条消息并从链表中删除该消息。

>Handler机制之Looper介绍

Looper可以理解为消息循环，因为MessageQueue只是一个存储消息队列，不能去处理消息，所以需要Looper无限循环的去查看是否有新的消息，如果有的话就处理消息，否则就一直等待（阻塞）。 从宏观的角度来分析的话，每一个异步线程，都维护着唯一的一个Looper，每一个Looper会初始化（维护）一个MessageQueue，之后进入一个无限循环一直在读取MessageQueue中存储的消息，如果没有消息那就一直阻塞等待。

* Looper中有2个比较重要的方法：

Prepare();
Loop();
Looper.prepare()简介

Handler的工作需要Looper，没有Looper的线程就会报错，所以就必须要通过Looper.prepare()方法为当前线程创建一个Looper，之后用Looper.loop()方法来开启消息循环。但是对于我们开发者来说，当我们在UI线程中实例化handler的时候并没有要求对Looper进行初始化，照样可以是运行没有任何问题的。其原因就是因为UI线程在被创建的时候就会初始化Looper，这样也就明白了在UI线程中可以默认使用handler的原因了。

Looper.prepare()源码如下：

```
/** Initialize the current thread as a looper.
      * This gives you a chance to create handlers that then reference
      * this looper, before actually starting the loop. Be sure to call
      * {@link #loop()} after calling this method, and end it by calling
      * {@link #quit()}.
      */
    public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```

我们发现该方法比较简单，prepare方法完成了Looper对象的初始化，对于源码中所说的sThreadLocal对象来说，它是ThreadLocal的实例化对象，对于ThreadLocal的详细分析介绍请[点击这里](http://blog.csdn.net/u011619211/article/details/51388430)。
 
通过以上对于源码的理解，我们可以知道该方法主要做了如下工作：

检查是否实例化了ThreadLocal
如果已经实例化ThreadLocal，则将所在线程中Looper对象存储进去
另外，下面我们通过一个Demo来展示下Handle初始化是需要事先调用Looper.perpare()的必要性：

```
**
 * @author zwjian
 * @des init Looper testActivity
 */
public class InitLooperActivity extends Activity {


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        new Thread("Thread#1") {

            @Override
            public void run() {
                super.run();
                Handler handler = new Handler();

            }
        }.start();

        new Thread("Thread#2") {

            @Override
            public void run() {
                super.run();
                Looper.prepare();
                Handler handler = new Handler();
                Looper.loop();

            }
        }.start();
    }
}

```

上述代码很简单，我们来看一下运行结果： 
![](http://img.blog.csdn.net/20160518122706797)

我们可以发现该运行时异常指的就是在Thread#1中没有初始化Looper导致handler实例化失败（后面我们会讲到这个异常是怎么抛出的）。

* Looper.loop()简介

该方法的主要功能就是不断从MessageQueue中读取消息，交给消息的target属性的dispatchMessage去处理。

其源码展示如下：
```
/**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            msg.target.dispatchMessage(msg);

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }

在该方法中，我们可以看到通过之前存储在threadLocal对象中的looper对象得到了该线程中对应looper对象中维护的MessageQueue消息队列，之后进入了一个无限循环，对消息队列中的数据进行读取，并且会把消息通过

msg.target.dispatchMessage(msg);进行处理，对于msg.target来说，其实它就是与之绑定的handler实例，在下面我们会进行详细分析。

以上就是对于Looper的详细介绍了，对于Looper的总结可以如下：

* 与当前的线程绑定，保证一个线程只有一个Looper实例，同时一个Looper实例只有一个MessageQueue
* loop方法不断的从MessageQueue中去取消息，交给消息的target属性的dispatchMessage去处理

>Handler机制之Handler介绍

handler的工作主要是消息的发送和接收过程。消息的发送可以通过post和send等一系列的方法来实现。但是需要说明的是，通过post的方法最后还是通过send来实现的。

通常我们在使用handler之前都会进行初始化，比如说比较常见的更新UI线程，我们会在声明的时候直接初始化。那么问题来了，handler是怎样和Looper以及MessageQueue联系起来的呢？它在子线程中发送的消息怎么发送到了MessageQueue中呢？

下面我们通过查看Handler的源码来进行这些疑点的解释：

首先我们先来看下Handler的构造函数：

```
public Handler(Callback callback, boolean async) {
            if (FIND_POTENTIAL_LEAKS) {
                final Class<? extends Handler> klass = getClass();
                if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                        (klass.getModifiers() & Modifier.STATIC) == 0) {
                    Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                            klass.getCanonicalName());
                }
            }

            mLooper = Looper.myLooper();
            if (mLooper == null) {
                throw new RuntimeException(
                        "Can't create handler inside thread that has not called Looper.prepare()");
            }
            mQueue = mLooper.mQueue;
            mCallback = callback;
            mAsynchronous = async;
        }
```
   
   很显然，我们可以发现在该构造函数中完成了该handler与looper的绑定，以及获得该looper对象中的消息队列等操作。同时在对mLooper进行判空的时候我们就可以发现一句熟悉的异常：”Can’t create handler inside thread that has not called Looper.prepare()”，而这个异常就是在前文中我们在没有调用Looper.prepare()方法而直接实例化handler时所报的异常信息了。

接下来我们来看一下handler发送消息的过程：

一般我们使用handler发送消息的函数可以有好几个，在这里我们使用如下代码进行发送：

` handler.sendMessage(new Message()); `

接下来我们就去看看它的发送过程是怎样的：

```

 public final boolean sendMessage(Message msg)
    {
        return sendMessageDelayed(msg, 0);
    }
```

```
public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
```

```
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```

```
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

最后调用的是handler的enqueueMessage方法，在该方法中我们发现了一条熟悉的语句：msg.target=this;这句话就是对msg.target赋值为handler对象本身，正好对应上前文在looper.loop()方法中msg.target.dispatchMessage(msg);

同时我们一路跟下来发现最后调用的就是queue.enqueueMessage方法，而这个方法就是我们前面介绍的消息队列中进行插入消息的那个方法。这样一来，我们就明确了handler发送的消息其实最后是进入了消息队列中，那插入消息到消息队列之后的操作是什么呢？我们接着往下看。

通过前面我们所说的Looper中的loop方法，Looper . Loop()方法中通过无限循环对消息队列进行读取消息，得到消息之后通过消息的target属性中的disPatchMessage()方法回调handlerMessage等方法。而handlerMessage方法就是我们在UI线程中实例化handler的过程中重写的handlerMessage方法。

disPatchMessage()源码如下：

```
/**
     * Handle system messages here.
     */
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```

我们发现在这个函数中其实有两种处理，一个是执行handleCallback(msg);另外一个就是执行handleMessage(msg);对应源码如下：

```
private static void handleCallback(Message message) {
        message.callback.run();
    }
```


```

/**
     * Subclasses must implement this to receive messages.
     */
    public void handleMessage(Message msg) {
    }
```

我们知道在我们实例化handler的时候，如果采用post方法，一般要new 一个Runnable对象，具体如下：

```
handler = new Handler();
        handler.post(new Runnable() {
            @Override
            public void run() {
               //to do change UI
            }
        })；
```
或者是这样的形式：

```
handler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                switch (msg.what) {
                    case xxx:
                        //to do  change UI
                        break;
                }

            }
        };
```

而所谓的disPatchMessage所要做的就是对这两种操作进行回调，之后执行开发者自行重写的代码，这样就实现了从一个线程中发送消息，在另外一个线程收到消息的全过程。

下面我们照样是通过一个简单的Demo来说明下：

相应的xml布局文件：

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.example.zwjian.myapplication.HandlerAct">

    <TextView
        android:id="@+id/test_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/toolbar"
        android:layout_margin="20dp"
        android:text="你是一个坏人" />
</RelativeLayout>
```
对应的Activity

```
public class HandlerAct extends AppCompatActivity {
    private static final int HANDLER_TEST_VALUE = 0X10;
    private Handler handler;
    private TextView test_tv;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_handler);

        test_tv = (TextView) findViewById(R.id.test_tv);
        //first method
        handler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                switch (msg.what) {
                    case HANDLER_TEST_VALUE:
                        test_tv.setText("我是第一个方法：谁说的？你才是一个坏人！");
                        break;
                }

            }
        };
        //another method
        handler = new Handler();
        handler.post(new Runnable() {
            @Override
            public void run() {
                test_tv.setText("我是第二个方法：谁说的？你才是一个坏人！");
            }
        });

        new Thread("Thread#2") {

            @Override
            public void run() {
                super.run();
//                handler.sendEmptyMessage(HANDLER_TEST_VALUE);
                handler.sendMessage(new Message());
            }
        }.start();
    }

}
```

以上，我们通过在thread#2中发送了一条消息，在UI线程中进行接收，当然这里写了2种接收方式，读者可以自行去注释掉某一种去进行演示，最后可以通过这个demo发现这两种方式都是可以被回调执行的，通过执行我们可以看到textView的显示发生了相对应的变化，这样，我们就完成了handler消息的传递的全过程。

>最后，我们来总结下handler机制的全过程：

* 首先Looper.prepare()会在当前线程保存一个looper对象，并且会维护一个消息队列messageQueue，而且规定了messageQueue在每个线程中只会存在唯一的一个
* Looper.loop()方法会使线程进入一个无限循环，不断地从消息队列中获取消息，之后回调msg.target.disPatchMessage方法。
* 我们在实例化handler的过程中，会先得到当前所在线程的looper对象，之后得到与该looper对象相对应的消息队列。（代码见handler的构造方法）
* 当我们发送消息的时候，即handler.sendMessage或者handler.post，会将msg中的target赋值为handler自身，之后加入到消息队列中。
* 在第三步实现实例化handler的过程中，我们一般会重写handlerMessage方法（使用post方法需要实现run方法），而这些方法将会在第二步中的msg.target.disPatchMessage方法中被回调，从而实现了message从一个线程到另外一个线程的传递。