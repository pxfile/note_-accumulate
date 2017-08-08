##精通Android下的Handler机制，并能熟练使用 

* Message：消息；其中包含了消息ID，消息对象以及处理的数据等，由MessageQueue统一列队，终由Handler处理

* Handler：处理者；负责Message发送消息及处理。Handler通过与Looper进行沟通，从而使用Handler时，需要实现handlerMessage(Message msg)方法来对特定的Message进行处理，例如更新UI等（主线程中才行）

* MessageQueue：消息队列；用来存放Handler发送过来的消息，并按照FIFO（先入先出队列）规则执行。当然，存放Message并非实际意义的保存，而是将Message以链表的方式串联起来的，等Looper的抽取。

* Looper：消息泵，不断从MessageQueue中抽取Message执行。因此，一个线程中的MessageQueue需要一个Looper进行管理。Looper是当前线程创建的时候产生的（UI Thread即主线程是系统帮忙创建的Looper，而如果在子线程中，需要手动在创建线程后立即创建Looper[调用Looper.prepare()方法]）。也就是说，会在当前线程上绑定一个Looper对象。

* Thread：线程；负责调度消息循环，即消息循环的执行场所。

**知识要点**    


> 一、说明   

* 1、handler应该由处理消息的线程创建。

* 2、handler与创建它的线程相关联，而且也只与创建它的线程相关联。handler运行在创建它的线程中，所以，如果在handler中进行耗时的操作，会阻塞创建它的线程。

> 二、一些知识点        

* 1、Android的线程分为有消息循环的线程和没有消息循环的线程，有消息循环的线程一般都会有一个Looper。主线程（UI线程）就是一个消息循环的线程。

* 2、获取looper：                
Looper.myLooper();      //获得当前的Looper     
Looper.getMainLooper () //获得UI线程的Lopper    

* 3、Handler的初始化函数（构造函数），如果没有参数，那么他就默认使用的是当前的Looper，如果有Looper参数，就是用对应的线程的Looper。

* 4、如果一个线程中调用Looper.prepare()，那么系统就会自动的为该线程建立一个消息队列，然后调用 Looper.loop();之后就进入了消息循环，这个之后就可以发消息、取消息、和处理消息。

![](http://ou21vt4uz.bkt.clouddn.com/handler1.png)

![](http://ou21vt4uz.bkt.clouddn.com/handler2.png)

**一、大致流程**       

* 1.在创建Activity之前，当系统启动的时候，先加载ActivityThread这个类，在这个类中的main函数，调用了Looper.prepareMainLooper()方法进行初始化Looper对象；

* 2.然后创建了主线程的handler对象（Tips：加载ActivityThread的时候，其内部的Handler对象[静态的]还未创建）；
 
* 3.随后才创建了ActivityThread对象；

* 4.最后调用了Looper.loop();方法，不断的进行轮询消息队列的消息。

* 也就是说，在ActivityThread和Activity创建之前（同样也是Handler创建之前，当然handler由于这两者初始化），就已经开启了Looper的loop()方法，不断的进行轮询消息。需要注意的是，这个轮询的方法是阻塞式的，没有消息就一直等待（实际是等着MessageQueue的next()方法返回消息）。在应用一执行的时候，就已经开启了Looper，并初始化了Handler对象。此时，系统的某些组件或者其他的一些活动等发送了系统级别的消息，这个时候主线程中的Looper就可以进行轮询消息，并调用msg.target.dispatchMessage(msg)（msg.target即为handler）进行分发消息，并通过handler的handleMessage方法进行处理；所以会优于我们自己创建的handler中的消息而处理系统消息。

**准备数据和对象：**

* ①、如果在主线程中处理message（即创建handler对象），那么如上所述，系统的Looper已经准备好了（当然，MessageQueue也初始化了），且其轮询方法loop已经开启。【系统的Handler准备好了，是用于处理系统的消息】。【Tips：如果是子线程中创建handler，就需要显式的调用Looper的方法prepare()和loop()，初始化Looper和开启轮询器】    

* ②、通过Message.obtain()准备消息数据（实际是从消息池中取出的消息）

* ③、创建Handler对象，在其构造函数中，获取到Looper对象、MessageQueue对象（从Looper中获取的），并将handler作为message的标签设置到msg.target上

* 1、发送消息：sendMessage()：通过Handler将消息发送给消息队列

* 2、给Message贴上handler的标签：在发送消息的时候，为handler发送的message贴上当前handler的标签

* 3、开启HandlerThread线程，执行run方法。

* 4、在HandlerThread类的run方法中开启轮询器进行轮询：调用Looper.loop()方法进行轮询消息队列的消息
>【Tips：这两步需要再斟酌，个人认为这个类是自己手动创建的一个线程类，Looper的开启在上面已经详细说明了，这里是说自己手动创建线程（HandlerThread）的时候，才会在这个线程中进行Looper的轮询的】

* 5、在消息队列MessageQueue中enqueueMessage(Message msg, long when)方法里，对消息进行入列，即依据传入的时间进行消息入列（排队）

* 6、轮询消息：与此同时，Looper在不断的轮询消息队列

* 7、在Looper.loop()方法中，获取到MessageQueue对象后，从中取出消息（Message msg = queue.next()）

* 8、分发消息：从消息队列中取出消息后，调用msg.target.dispatchMessage(msg);进行分发消息

* 9、将处理好的消息分发给指定的handler处理，即调用了handler的dispatchMessage(msg)方法进行分发消息。

* 10、在创建handler时，复写的handleMessage方法中进行消息的处理

* 11、回收消息：在消息使用完毕后，在Looper.loop()方法中调用msg.recycle()，将消息进行回收，即将消息的所有字段恢复为初始状态

测试代码：

```
/**
 * Handler 构造函数测试
 * @author zhaoyu 2013-10-5 上午9:56:38
 */
public class HandlerConstructorTest extends Activity {
	private Handler handler1 = new Handler(new Callback() {
		@Override
		public boolean handleMessage(Message msg) {
			System.out.println("使用了Handler1中的接口Callback");
			return false;		// 此处，如果返回 false，下面的 handlerMessage方法会执行，true ，下面的不执行
		}
	});
	
	private Handler handler2 = new Handler() {
		public void handleMessage(Message msg) {
			System.out.println("Handler2");
		}
	};

	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		//消息1
Message obtain1 = Message.obtain();
		obtain1.obj = "sendMessage";
		obtain1.what = 1;
		handler1.sendMessage(obtain1);
		//消息2
		Message obtain2 = handler2.obtainMessage();
		handler2.sendMessage(obtain2);	//①
//		handler2.dispatchMessage(obtain2);	//②
	}
}
```

**二、详细解释**      

* **1、准备Looper对象**

**两种情况初始化Looper对象：**

* 1）在主线程中不需要显式的创建Looper对象，直接创建Handler对象即可；因为在主线程ActivityThread的main函数中已经自动调用了创建Looper的方法：Looper.prepareMainLooper();，并在最后调用了Looper.loop()方法进行轮询。

* 2）如果在子线程中创建Handler对象，需要创建Looper对象，即调用显式的调用Looper.prepare()

**初始化Looper的工作：**

* 1）初始化Looper对象：通过调用Looper.prepare()初始化Looper对象，在这个方法中，新创建了Looper对象

* 2）将Looper绑定到当前线程：在初始化中，调用sThreadLocal.set(new Looper(quitAllowed))方法，将其和ThreadLocal进行绑定
在ThreadLocal对象中的set方法，是将当前线程和Looper绑定到一起：首先获取到当前的线程，并获取线程内部类Values，通过Thread.Values的put方法，将当前线程和Looper对象进行绑定到一起。即将传入的Looper对象挂载到当前线程上。
>Tips：在Looper对象中，可以通过getThread()方法，获取到当前线程，即此Looper绑定的线程对象。

源代码：

Looper中：

```
public static void prepare() {
        prepare(true);
    }
    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
ThreadLocal中：
public void set(T value) {
        Thread currentThread = Thread.currentThread();
        Values values = values(currentThread);
        if (values == null) {
            values = initializeValues(currentThread);
        }
        values.put(this, value);
    }

```
    
* **2、创建消息Message：**

**消息的创建可以通过两种方式：**

* 1）new Message()

* 2）Message.obtain()：【当存在多个handler的时候，可以通过Message.obtain(Handler handler)创建消息，指定处理的handler对象】
>Tips：建议使用第二种方式更好一些。原因：
	因为通过第一种方式，每有一个新消息，都要进行new一个Message对象，这会创建出多个Message，很占内存。
	而如果通过obtain的方法，是从消息池sPool中取出消息。每次调用obtain()方法的时候，先判断消息池是否有消息（if (sPool != null)），没有则创建新消息对象，有则从消息池中取出消息，并将取出的消息从池中移除【具体看obtain()方法】
	

```
public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                sPoolSize--;
                return m;
            }
        }
        return new Message();
    }

public Message() {
    }
    
```
 
* **3、创建Handler对象**   

两种形式创建Handler对象：

 1）创建无参构造函数的Handler对象：

 2）创建指定Looper对象的Handler对象

最终都会调用相应的含有Callback和boolean类型的参数的构造函数
>【这里的Callback是控制是否分发消息的，其中含有一个返回值为boolean的handleMessage(Message msg)方法进行判断的；
  boolean类型的是参数是判断是否进行异步处理，这个参数默认是系统处理的，我们无需关心】

在这个构造函数中，进行了一系列的初始化工作：

* ①、获取到当前线程中的Looper对象

* ②、通过Looper对象，获取到消息队列MessageQueue对象

* ③、获取Callback回调对象

* ④、获取异步处理的标记

源代码：

①、创建无参构造函数的Handler对象：

```
public Handler() {
        this(null, false);
    }
public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) && (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +klass.getCanonicalName());
            }
        }
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException("Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```

②、创建指定Looper对象的Handler对象

```
public Handler(Looper looper) {
        this(looper, null, false);
    }
public Handler(Looper looper, Callback callback, boolean async) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```


* **4、Handler对象发送消息：**

* 1）Handler发送消息给消息队列：
Handler对象通过调用sendMessage(Message msg)方法，最终将消息发送给消息队列进行处理

这个方法（所有重载的sendMessage）最终调用的是enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis)

* （1）先拿到消息队列：在调用到sendMessageAtTime(Message msg, long uptimeMillis)方法的时候，获取到消息队列（在创建Handler对象时获取到的）

* （2）当消息队列不为null的时候（为空直接返回false，告知调用者处理消息失败），再调用处理消息入列的方法：

`
enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis)
`

这个方法，做了三件事：

* ①、为消息打上标签：msg.target = this;：将当前的handler对象这个标签贴到传入的message对象上，为Message指定处理者
* ②、异步处理消息：msg.setAsynchronous(true);，在asyn为true的时候设置
* ③、将消息传递给消息队列MessageQueue进行处理：

`
queue.enqueueMessage(msg, uptimeMillis);
`

```
public final boolean sendMessage(Message msg){
        return sendMessageDelayed(msg, 0);
    }
public final boolean sendMessageDelayed(Message msg, long delayMillis){
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
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
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

* 2）MessageQueue消息队列处理消息：

在其中的enqueueMessage(Message msg, long when)方法中，工作如下：
在消息未被处理且handler对象不为null的时候，进行如下操作（同步代码块中执行）

* ①、将传入的处理消息的时间when（即为上面的uptimeMillis）赋值为当前消息的when属性。
* ②、将next()方法中处理好的消息赋值给新的消息引用：Message p = mMessages;
	在next()方法中：不断的从消息池中取出消息，赋值给mMessage，当没有消息发来的时候，Looper的loop()方法由于是阻塞式的，就一直等消息传进来
* ③、当传入的时间为0，且next()方法中取出的消息为null的时候，将传入的消息msg入列，排列在消息队列上，此时为消息是先进先出的，否则，进入到死循环中，不断的将消息入列，根据消息的时刻（when）来排列发送过来的消息，此时消息是按时间的先后进行排列在消息队列上的
	
```
final boolean enqueueMessage(Message msg, long when) {
        if (msg.isInUse()) {
            throw new AndroidRuntimeException(msg + " This message is already in use.");
        }
        if (msg.target == null) {
            throw new AndroidRuntimeException("Message must have a target.");
        }
        boolean needWake;
        synchronized (this) {
            if (mQuiting) {
                RuntimeException e = new RuntimeException(msg.target + " sending message to a Handler on a dead thread");
                Log.w("MessageQueue", e.getMessage(), e);
                return false;
            }

            msg.when = when;
            Message p = mMessages;
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }
        }
        if (needWake) {
            nativeWake(mPtr);
        }
        return true;
    }
```

* **5、轮询Message**

* 1）开启loop轮询消息
当开启线程的时候，执行run方法，在HandlerThread类中，调用的run方法中将开启loop进行轮询消息队列：

在loop方法中，先拿到MessageQueue对象，然后死循环不断从队列中取出消息，当消息不为null的时候，通过handler分发消息：msg.target.dispatchMessage(msg)。消息分发完之后，调用msg.recycle()回收消息，

* 2）回收消息：
在Message的回收消息recycle()这个方法中：首先调用clearForRecycle()方法，将消息的所有字段都恢复到原始状态【如flags=0，what=0，obj=null，when=0等等】
然后在同步代码块中将消息放回到消息池sPool中，重新利用Message对象

源代码：
Looper.loop()

```
public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();
        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
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
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, “……”);
            }
            msg.recycle();
        }
    }

msg.recycle();：
public void recycle() {
        clearForRecycle();
        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }

/*package*/ void clearForRecycle() {
        flags = 0;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        when = 0;
        target = null;
        callback = null;
        data = null;
    }
```

* **6、处理Message**

在Looper.loop()方法中调用了msg.target.dispatchMessage(msg);的方法，就是调用了Handler中的dispatchMessage(Message msg)方法：

* 1）依据Callback中的handleMessage(msg)的真假判断是否要处理消息，如果是真则不进行消息分发，则不处理消息，否则进行处理消息
* 2）当Callback为null或其handleMessage(msg)的返回值为false的时候，进行分发消息，即调用handleMessage(msg)处理消息【这个方法需要自己复写】

```
/**
     * Subclasses must implement this to receive messages.
     */
    public void handleMessage(Message msg) {
    }
    
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

==========
**场景一：**

在主线程中创建Handler，其中复写了handleMessage方法（处理message，更新界面）
然后创建子线程，其中创建Message对象，并设置消息，通过handler发送消息

* **示例代码：**

```
public class MainActivity2 extends Activity implements OnClickListener{
    private Button bt_send;
	private TextView tv_recieve;
	private Handler handler = new Handler(){
		@Override
		public void handleMessage(Message msg) {
			super.handleMessage(msg);
			tv_recieve.setText((String) msg.obj);
		}
	};

	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        bt_send = (Button) findViewById(R.id.bt_send);
        tv_recieve = (TextView) findViewById(R.id.tv_recieve);
        bt_send.setOnClickListener(this);
        tv_recieve.setOnClickListener(this);
	}
	@Override
	public void onClick(View v) {
		switch (v.getId()) {
		case R.id.bt_send:
			new Thread(){
				public void run() {
					Message msg = new Message();
					msg.obj = "消息来了"+ System.currentTimeMillis();
					handler.sendMessage(msg);
				}
			}.start();
			break;
		}
	}
}

```

* **执行过程：**

**1、Looper.prepare()**

在当前线程(主线程)中准备一个Looper对象，即轮询消息队列MessageQueue的对象；此方法会创建一个Looper，在Looper的构造函数中，初始化的创建了一个MessageQueue对象(用于存放消息)，并准备好了一个线程供调用

**2、new Handler():**

在当前线程中创建出Handler，需要复写其中的handleMessage(Message msg)，对消息进行处理（更新UI）。在创建Handler中，会将Looper设置给handler，并随带着MessageQueue对象；其中Looper是通过调用其静态方法myLooper()，返回的是ThreadLocal中的currentThread，并准备好了MessageQueue【mQueue】

**3、Looper.loop():**

无限循环，对消息队列进行不断的轮询，如果没有获取到消息，就会阻塞线程；如果有消息，直接从消息队列中取出消息，并通过调用msg.target.dispatchMessage(msg)进行分发消息给各个控件进行处理。
[其中的msg.target实际就是handler]。

**4、创建子线程，handler.sendMessage(msg)**

在handler.sendMessage(msg)方法中，实际上最终调用sendMessageAtTime(Message msg，long uptimeMillis)方法[sendMessageXXX方法都是最终调用的sendMessageAtTime方法]；此方法返回的enqueueMessage(queue，msg，uptimeMillis)，实际上返回的是MessageQueue中的enqueueMessage(msg，uptimeMillis)，其中进行的操作时对存入的消息进行列队，即根据接收到的消息的时间先后进行排列[使用的单链形式]；然后将消息就都存入到了消息队列中，等待着handler进行处理。

![](http://ou21vt4uz.bkt.clouddn.com/handler3.png)

![](http://ou21vt4uz.bkt.clouddn.com/handler4.png)

**场景二：**

创建两个子线程，一个线程中创建Handler并进行处理消息，另一个线程使用handler发送消息。

示例代码：

```
public class MainActivity extends Activity implements OnClickListener{

   private Button bt_send;
	private TextView tv_recieve;
	private Handler handler;

	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        bt_send = (Button) findViewById(R.id.bt_send);
        tv_recieve = (TextView) findViewById(R.id.tv_recieve);
        bt_send.setOnClickListener(this);
        tv_recieve.setOnClickListener(this);
        new Thread(){
        	public void run() {
        		//Looper.prepare();
        		handler = new Handler(Looper.getMainLooper()){
        			@Override
        			public void handleMessage(Message msg) {
        				super.handleMessage(msg);
        				tv_recieve.setText((String) msg.obj);
        				
        			}
        		};
        		//Looper.loop();
        	}
        }.start();
    }

	@Override
	public void onClick(View v) {
		switch (v.getId()) {
		case R.id.bt_send:
			new Thread(){
				public void run() {
					Message msg = new Message();
					msg.obj = "消息来了"+ System.currentTimeMillis();
					handler.sendMessage(msg);
				}
			}.start();
			break;
		}
	}
}

```

简单说明执行过程:

说明：在子线程中是不能更新界面的操作的，只能放在主线程中进行更新。所以必须将处理的消息放到主线程中，才能进行更新界面，否则会报错

* 1、子线程中创建Handler，并处理消息
1）创建Handler：
源码如下：

```
public Handler(Looper looper, Callback callback, boolean async) {
  mLooper = looper;
  mQueue = looper.mQueue;
  mCallback = callback;
  mAsynchronous = async;
}
```
这个构造函数做了一下几步工作：

**①、创建轮询器：**

由于新创建的子线程中没有轮询器，就需要创建一个轮询器，才能进行消息的轮询处理。传入的是主线程的轮询器，就已经将这个looper绑定到主线程上了【传入哪个线程的Looper，就绑定在哪个线程上】

**②、将消息队列加入到轮询器上。**

消息队列MessageQueue是存放handler发来的消息的，等着Looper进行轮询获取；在一个线程中的MessageQueue需要一个Looper进行管理，所以两者需要同在一个线程中。

**③、回调和异步加载。（此处不做分析[其实我还没分析好]）**
需要注意的是界面更新：
上面说到了，在子线程中是不可以进行更新界面的操作的，这就需要使用带有轮询器参数的handler构造函数进行创建，传入主线程的轮询器：Looper.getMainLooper()，从而将消息加入到主线程的消息队列之中。因此就可进行在handleMessage方法中进行处理消息更新界面了。

* 2）、消息处理：

复写其中的handleMessage(Message msg)，对消息进行处理（更新UI）。
在创建Handler中，会将Looper设置给handler，并随带着MessageQueue对象；其中Looper是通过调用其静态方法myLooper()，返回的是ThreadLocal中的currentThread，并准备好了MessageQueue【mQueue】

虽然是在子线程中编写的代码，但是由于传入的是主线程的looper，所以，Looper从MessageQueue队列中轮询获取消息、再进行更新界面的操作都是在主线程中执行的。

* 3）、Looper.loop():

说明：由于传入的是主线程的Looper，而在主线程中已经有这一步操作了，所以这里就不需要进行显示的调用了。但是主线程在这个时候是做了这个轮询的操作的。

无限循环，对消息队列进行不断的轮询，如果没有获取到消息，就会结束循环；如果有消息，直接从消息队列中取出消息，并通过调用msg.target.dispatchMessage(msg)进行分发消息给各个控件进行处理。

[其中的msg.target实际就是handler]。

* 2、创建子线程，发送消息handler.sendMessage(msg)

新开一个子线程，发送消息给另一个子线程
  
在handler.sendMessage(msg)方法中，实际上最终调用sendMessageAtTime(Message msg，long uptimeMillis)方法[sendMessageXXX方法都是最终调用的sendMessageAtTime方法] 
  
此方法返回的enqueueMessage(queue，msg，uptimeMillis)，实际上返回的是MessageQueue中的enqueueMessage(msg，uptimeMillis)，其中进行的操作时对存入的消息进行列队，即根据接收到的消息的时间先后进行排列[使用的单链形式]；

然后将消息就都存入到了消息队列中，等待着handler进行处理。

![](http://ou21vt4uz.bkt.clouddn.com/handler5.png)