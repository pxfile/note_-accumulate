#第二章：IPC机制（中）
---
>好的，我们继续来了解IPC机制，在上篇我们可能就是把理论的知识写完了，然后现在基本上是可以实战了。

##一.Android中的IPC方式
>本节我们开始详细的分析各中跨进程的方式，具体方式有很多，比如可以通过在Intent中附加extras来传递消息，或者通过共享文件的方式来共享数据，还可以采用Binder方式来跨进程通信，另外，ContentProvider天生就是支持扩进程访问的，所以通过Socket也可以实现IPC，上述的各种方法都能实现IPC，他们的使用方式和侧重点上有很大的区别，我们在这里都会一一说的；

###1.使用Bundler
>Bundler，用Intent传值的引用对象，我们都知道A,S,B三大组件都是可以通过intent的Bundler传值的，那是因为Bundler实现了Parcelable接口，所以他可以在不同的进程间传输，基于这一点,当我们在一个进程中使用另外一个进程的A,S,B，我们就可以在Bunlder中附加我们需要传输给远程进程的信息，然后用intent发送过去，当然，我们传输的数据必须能够序列化，比如基本数据类型，实现了Parcelable接口的对象，实现了Serializable接口的对象以及一些Android支持的特殊对象，具体内容可以看下Bundler这个类，就可以看出他所支持的类型了，Bundler不支持的类型我们无法通过他在进程间传递数据，这个很简单，就不再详细介绍了，这是一种很简单的进程间通信方式

>除了直接传递数据这种典型的使用场景，他还有一种特殊的使用场景，比如A进程正在进行一个计算，计算完成之后他要启动B进程一个组件并把计算结果传递给B进程，可是遗憾的是这个数据不支持放在Bundler里，因此无法使用intent来传递，这个时候如果我们用其他IPC，那就稍微有些麻烦了，可以考虑如下形式：通过intent启动进程B的一个Service组件，让Service去完成计算，这样就不再需要什么跨进程了，这也只是一种小的技巧

##2.使用文件共享
>文件共享是一种不错的进程间通讯的方式，两个进程通过读/写同一个文件来交换数据，比如A进程把数据写入文件，B再去读取，我们知道，在Winddows上，一个文件如果被加了排斥锁将会导致其他线程无法对其访问，甚至两个线程同时进度写的操作也是不允许的，尽管这可能出问题，通过文件交换数据很好使用，除了可以交换一些文本信息外，我们还可以序列化UI和对象到文件里，引用的时候再恢复，下面就来展示下这个功能

>还是本章的例子，在onResume序列化一个对象到sd卡，第二个Activity去恢复，，关键代码如下：

###MainActivity

```
private void persistToFile() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                User user = new User("1", "jek", false);
                File dir = new File(PATH);
                if (!dir.exists()) {
                    dir.mkdir();
                }
                File cachedFile = new File(PATH);
                ObjectOutputStream objectOutputStream = null;
                try {
                    objectOutputStream = new ObjectOutputStream(new FileOutputStream(cachedFile));
                    objectOutputStream.writeObject(user);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

```

###SecondActivity


```
 private void recoverFromFile() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                User user = null;
                File cachedFile = new File(MainActivity.PATH);
                if(cachedFile.exists()){
                    ObjectInputStream objectInputStream = null;
                    try {
                        objectInputStream = new ObjectInputStream(new FileInputStream(cachedFile));
                        user = (User) objectInputStream.readObject();
                    } catch (IOException e) {
                        e.printStackTrace();
                    } catch (ClassNotFoundException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
    }

```

>如果我们查看log的话会发现，，是成功的恢复了User对象的数据的

>通过文件共享这种方式来共享数据对文件格式是没有具体要求的，比如可以是文本文件，也可以是XML文件，只要读/写双方约定数据格式即可。通过文件共享的方式也是有局限性的，比如并发读/写的问题，像上面的那个例子，如果并发读/写，那么我们读出的内容就有可能不是最新的，如果是并发写的话那就更严重了。因此我们要尽量避免并发写这种情况的发生或者考虑使用线程同步来限制多个线程的写操作。通过上面的分析，我们可以知道，文件共享方式适合在对数据同步要求不高的进程之间进行通信，并且要妥善处理并发读/写的问题。

>当然，SharedPreferences是个特例，众所周知，SharedPreferences是Android中提供的轻量级存储方案，它通过键值对的方式来存储数据，在底层实现上它采用XML文件来存储键值对，每个应用的SharedPreferences文件都可以在当前包所在的data目录下查看到，一般来说，它的目录位于/data/data/package name/shared_prefs目录下，其中package name表示的是当前应用的包名。从本质上来说，SharedPreferences也属于文件的一种，但是由于系统对它的读/写有一定的缓存策略，即在内存中会有一份SharedPreferences文件的缓存，因此在多进程模式下，系统对它的读/写就变得不可靠，当面对高并发的读/写访问Sharedpreferences有很大几率会丢失数据，因此，不建议在进程间通信中使SharedPreferences。


###3.使用Messenger
>Messenger可以翻译为信使，顾名思义，通过它可以在不同进程中传递Message对象，在Message中放入我们需要传递的数据，就可以轻松地实现数据的进程间传递了。Messenger是一种轻量级的IPC方案，它的底层实现是AIDL，为什么这么说呢，我们大致看一下Messenger这个类的构造方法就明白了。下面是Messenger的两个构造方法，从构造方法的实现上我们可以明显看出AIDL的痕迹，不管是IMessenger还是Stub.asInterface，这种使用方法都表明它的底层是AIDL。

```
	public Messenger(Handler target) {
        mTarget = target.getIMessenger();
    }
    
    public Messenger(IBinder target){
    	mTarget = IMessenger.Stud.asInterface(target);
    }
```

>Messenger的使用方法很简单，它对AIDL做了封装，使得我们可以更简便地进行进程通信。同时，由于它一次处理一个请求，因此在服务端我们不用考虑线程同步的问题，这是因为服务端中不存在并发执行的情形。实现一个Messenger有如下几个步骤，分为服
务端和客户端。

- 1.服务端进程

>首先，我们需要在服务端创建一个Service来处理客户端的连接请求，同时创建一个Handler并通过它来创建一个Messenger对象，然后在Service的onBind中返回这个Messenger对象底层的Binder即可。

- 2.客户端进程

>客户端进程中，首先要绑定服务端的Service，绑定成功后用服务端返回的IBinder对象创建一个Messenger，通过这个Messenger就可以向服务端发送消息了，发消息类型为
Message对象。如果需要服务端能够回应客户端，就和服务端一样，我们还需要创建一个Handdler并创建一个新的Messenger，并把这个Messenger对象通过Message的replyTo参数传递给服务端，服务端通过这个replyTo参数就可以回应客户端。这听起来可能还是有点抽
象，不过看了下面的两个例子，读者肯定就都明白了。首先，我们来看一个简单点的例子，
这个例子中服务端无法回应客户端。。

>首先看服务端的代码，这是服务端的典型代码，可以看到MessengerHandler用来处理客户端发送的消息，并从消息中取出客户端发来的文本信息，而mMessenger是一个Mwsswnger对象，他和MessengerHandler相关联，并在onBind方法中返回他里面的IBind对象

```
public class MessengerService extends Service {

    public static final String TAG = "MessengerService";

    private static class MessengerHandler extends Handler{
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 100:
                    Log.i(TAG,"数据:" + msg.getData());
                    break;
                default:
                    
                super.handleMessage(msg);
            }
        }
    }

    private final Messenger mMessenger = new Messenger(new MessengerHandler());

    @Override
    public IBinder onBind(Intent intent) {
        
        return mMessenger.getBinder();
    }
}

```

>然后注册下这个Service，让其在独立进程中运行

```
 <service android:name=".MessengerService"
            android:process=":remote"/>

```

>接下来我们看看服务端，服务端比较简单

```
public class MessengerActivity extends AppCompatActivity {

    public static final String TAG = "MessengerActivity";

    private Messenger mService;
    
    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mService =  new Messenger(service);
            Message msg = Message.obtain(null,10);
            Bundle data = new Bundle();
            data.putString("msg","hell this is client");
            msg.setData(data);
            try {
                mService.send(msg);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

    @Override
    protected void onCreate( Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_messenger);

        Intent intent = new Intent(this,MessengerService.class);
        bindService(intent,mConnection, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(mConnection);
    }
}


```

>这样运行之后就能收到发送的消息了

>通过上面的例子可以看出，在Messenger中进行数据传递必须将数据放入Messsage中，而Messenger和Message都实现了Parcelable接口，因此可以跨进程传输，简单来说，Messebger所支持的数据类型就是Message所支持的传输类型。实际上，通过 Messenger来传递Message，Message中能使用的载体就只有what、arg1、arg2、Bundle以及replyTo。Message的另一个字段object在同一个进程中是很实用的，但是在进程间通信的时候，在Android2.2以前object字段不支持跨进程传输，即便是2.2以后，也仅仅是系统提供的实现了Parcelable接口的对象才能通过它来传输。这就意味着我们自定义的Parcelable对象是无法通过object字段来传输的，读者可以试一下。非系统的Parcelable对象的确无法通过object字段来传输，这也导致了object字段的实用性大大降低，所幸我们还有Bundle，Bundle中
可以支持大量的数据类型。

>上面的例子演示了如何在服务端接收客户端中发送的消息，但是有时候我们还需要能回应客户端，下面就介绍如何实现这种效果。还是采用上面的例子，但是稍微做一下修改，每当客户端发来一条消息，服务端就会自动回复一条“嗯，你的消息我已经收到，稍后
回复你。”，这很类似邮箱的自动回复功能。

>首先看服务端的修改，服务端只需要修改MessengerHandler，当收到消息后，会立即回复一条消息给客户端


```
private static class MessengerHandler extends Handler{
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 100:
                    Log.i(TAG,"数据:" + msg.getData());
                    Messenger messenger = msg.replyTo;
                    Message reply = Message.obtain(null,200);
                    Bundle bundle = new Bundle();
                    bundle.putString("reply","收到你的消息,等下回复");
                    try {
                        messenger.send(reply);
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }

```

>接着我们来看看客户端的修改，为了接受服务端的恢复，客户端也需要准备一个接收消息的Messenger和handler，如下：


```
 private Messenger mGetReplyMessenger = new Messenger(new MessengerHandler());

    private static class MessengerHandler extends Handler{
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what){
                case 200:
                    Log.i(TAG,"Service:" + msg.getData().getString("reply"));
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }

```

>除了上述的修改，还有关键的一点，当客户端发送消息的时候，需要把接收服务端回复的Messenger通过Message的replyTo参数传递给服务端,如下：


```
private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mService =  new android.os.Messenger(service);
            Message msg = Message.obtain(null,100);
            Bundle data = new Bundle();
            data.putString("msg","hell this is client");
            //注意这句话
            msg.replyTo  = mGetReplyMessenger;
            msg.setData(data);
            try {
                mService.send(msg);
            } catch (RemoteException e) {
                e.printStackTrace();
            }

        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

```

>通过上述的修改，我们再次运行，就达到了自动回复的效果了；

>到这里，我们采用Messenger进程通讯的例子就说完了，我们画一张工作原理图，这样更加便于理解：

![这里写图片描述](http://img.blog.csdn.net/20161005161726760)

>关于进程间通信，这个只能针对于同一个应用的通讯，那有没有针对不同应用的？是这样的，之所以选择在同一个应用内进行进程间通信，是因为操作起来比较方便，但是效果和在两个应用间进行进程间通信是一样的。在本章刚开始就说过，同一个应用的不同组件，如果它们运行在不同进程中，那么和它们分别属于两个应用没有本质区别，关于这点需要深刻理解，因为这是理解进程间通信的基础。

###四.使用AIDL

>上一节我们介绍了使用Messenger来进行进程间通信的方法，可以发现，Messenger是以串行的方式处理客户端发来的消息，如果大量的消息同时发送到服务端，服务端仍然只能一个个处理，如果有大量的并发请求，那么用Messenger就不太合适了。同时，Messenger的作用主要是为了传递消息，很多时候我们可能需要跨进程调用服务端的方法，这种情形用Messenger就无法做到了，但是我们可以使用AIDL来实现跨进程的方法调用。AIDL是Messenger的底层实现，因此Messenger本质上也是AIDL，只不过系统为我们做了封装，从而方便上层的调用而已。在上一节中，我们介绍了Binder的概念，大家对Binder也有了一定的了解，在Binder的基础上我们可以更加容易地理解AIDL。这里先介绍使用AIDL
来进行进程间通信的流程，分为服务端和客户端两个方面。

- 1.服务端

>服务端首先要创建一个Service用来监听客户端的连接请求，然后创建一个AIDL文件，将暴露给客户端的接口在这个AIDL文件中声明，最后在Service中实现这个AIDL接口即可

- 2.客户端

>客户端所要做事情就稍微简单一些，首先需要绑定服务端的Service，绑定成功后，将服务端返回的Binder对象转成AIDL接口所属的类型，接着就可以调用AIDL中的方法了

>上面描述的只是一个感性的过程，AIDL的实现过程远不止这么简单，接下来会对其中的细节和难点进行详细介绍，并完善我们在Binder那一节所提供的的实例。

- 3.AIDL接口的创建

>首先看AIDL接口的创建，如下所示，我们创建了一个后缀为AIDL.的文件，在里面明了一个接口和两个接口方法。

```
// IBookManager.aidl
package com.liuguilin.ipcsample;

import com.liuguilin.ipcsample.Book;

interface IBookManager {
    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    List<Book>getBookList();
    void addBook(in Book book);
}


```

>在AlDL文件中，并不是所有的数据类型都是可以使用的，那么到底AIDL文件哪些数据类型呢？如下所示

- 基本数据类型(int、long、char、boolean、double等）；
- Sting和CharSequence:
- List:只支持ArrayList，里面每个元素都必须能够被AIDL支持；
- Map：只支持HashMap，里面的每个元素都必须被AIDL支持，包括key和value:
- Parcelable：所有实现了Parcelable接口的对象；
- AIDL.所有的AIDL接口本身也可以在AIDL.文件中使用

>以上6种数据类型就是AIDL所支持的所有类型，其中自定义的Parcelable对象和AIDL对家比须要显式import进来，不管它们是否和当前的AIDL文件位于同一个包内。比如IBookManager.AIDL这个文件，里面用到了Book这个类，这个类实现了Parcelable接口并且和IBookManager.AIDL位于同一个包中，但是遵守AIDL的规范，我们仍然需要显式的

```
import com.liuguilin.ipcsample.Book;
```

>AIDL中会大量使用到Parcelable，至于如何使用,在本章的前面已经介绍过，这里就不再整述。

>另外一个需要注意的地方就是，如果AIDL文件中用到了自定义的Parcelable对象，那么必须新增一个和他同名的AIDL文件，并在其中声明它为Parcelable 类型。在上面的IBookManager.AIDL这个类中我们用到Book，所以，我们必须要创建Book.aidl，然后
在里面添加如下内容:

```
// Book.aidl.aidl
package com.liuguilin.ipcsample;

parcelable Book;

```

>我们需要注意，AIDL中每个实现了Parcelable接口的类都需要按照上面那种方式去创建相应的AIDL文件并声明那个类为parcelable。除此之外，AIDL中除了基本数据类型，其他类型的参数必须标上方向：in、out或者inout,in表示输入型参数，out表示输出型参数，inout表示输入输出型参数，至于它们具体的区别，这个就不说了。我们要根据实际需要去指定参数类型，不能一概使用out或者inout，因为这在底层实现是有开销的。最后，，AIDL接口中只支持方法，不支持声明静态常量，这一点区别于传统的接口。

>为了方便AIDL的开发，建议把所有和AIDL相关的类和文件全部放入同一个包中，这样做的好处是，当客户端是另外一个应用时，我们可以直接把整个包复制到客户端工程,对于本例来说，就是要把aidl这个包和包中的文件原封不动地复制到客户端中。如果AIDL相关的文件位于不同的包中时，那么就需要把这些包一一复制到服务端和客户端要保持一致，否则运行会出错，这是因为客户端需要反序列化服务端中和AIDL接口相关的所有类，如果类的完整路径不一样的话，就无法成功反序列化，程序也就无法正常运行。为了方便演示，本章的所有示例都是在同一个工程中进行的，但是读者要理解，一个工程和两个工程的多进程本质是一样的，两个工程的情况，除了需要复制AIDL接口所相关的包到客户端，其他完全一样，读者可以自行试验。

- 4.远程服务端Service的实现

>上述我们讲了如何定义一个AIDL的接口，接下来我们实现以下这个接口：


```
public class BookManagerService extends Service {

    private static final String TAG = "BookManagerService";

    private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<>();

    private Binder mBinder = new IBookManager.Stub() {

        @Override
        public List<Book> getBookList() throws RemoteException {
            return mBookList;
        }

        @Override
        public void addBook(Book book) throws RemoteException {
            mBookList.add(book);
        }
    };

    @Override
    public void onCreate() {
        super.onCreate();

        mBookList.add(new Book(1,"Android"));
        mBookList.add(new Book(1,"IOS"));
    }

    @Override
    public IBinder onBind(Intent intent) {

        return mBinder;

    }
}

```

>上面是一个服务端Service的典型实现，首先在onCreate中初始化添加了两本图书的信息，然后创建了一个Binder对象并在onBind中返回它，这个对象继承自IBookManager.Stub并且实现了它内部的AIDL方法，这个过程在Binder那一节已经介绍过了，这里就不多说了。这里主要看getBookList和addBook这两个AIDL方法的实现，实现过程也比较简单，注意这里采用了CopyOnWriteArrayList，这个CopyOnWriteArrayList支持并发读/写。在前面我们提到，AIDL方法是在服务端的Binder线程池中执行的，因此当多个客户端同时连接的
时候，会存在多个线程同时访问的情形，所以我们要在AIDL方法中处理线程同步，而我们直接使用CopyonWriteArayList来进行自动的线程同步。

>前面我们提到，AIDL中能够使用的List只有ArrayList，但是我们这里却使用了CopyOnWriteArrayList，注意它不是继承ArrayList，为什么能够正常工作呢?这是因为AIDL中所支持的是抽象的List，而List只是一个接口，因此虽然服务端返回的是CopyOnWriteArrayList，但是在Binder中会按照List的规范去访问数据并最终形成一个新的ArrayList传递给客户端。所以，我们在服务端采用CopyOnWriteArrayList是完全可以的。和此类似的还有ConcurrentHashMap，你可以体会一下这种转换情形，然后我们需要在清单文件中注册一下

```
<service
     android:name=".BookManagerService"
     android:process=":remote" />
```

###五.客户端的实现
>客户端的实现就比较简单了，我们绑定远程服务，绑定成功之后返回的Binder对象转换成AIDL接口，让后可以通过这个接口去调用服务端的远程方法，代码如下：

```

public class BookManagerActivity extends AppCompatActivity {

    private static final String TAG = "BookManagerActivity";

    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            IBookManager bookManager = IBookManager.Stub.asInterface(service);
            try {
                List<Book> list = bookManager.getBookList();
                Log.i(TAG, "list Type :" + list.getClass());
                getCanonicalName();
                Log.i(TAG, "list string : " + list.toString());
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

    private void getCanonicalName() {
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_book_manager);

        Intent intent = new Intent(this, BookManagerService.class);
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);

    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        unbindService(mConnection);
    }
}


```

>绑定成功之后，会通过bookManager去掉用getBookList方法，然后打印出所获取的图书信息，需要注意的是，服务端的方法可能需要很久才会执行完毕，这个时候下面的代码就会导致ANR，这一点需要注意，后面我们再解释这种情况，之所以要这样学是为了让你更加熟悉AIDL的步骤

>我们执行一下，log也就打印了，可以发现，虽然我们服务端返回的CopyOnWriteArrayList类型，但是客户端收到的仍然是ArrayList，这也证实了我们前面所做的分析，第二个log得到信息；

>这就是一个完整的AIDL的运行过程了，但是这些都只是皮毛而已，真正的AIDL应用汇复杂许多，我们再来聊一下AIDL的难点：

>我们将诶下来再启动另一个接口addBook，我们在客户端给服务端添加一本书在获取一次看看现在是什么样的；


```
private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            IBookManager bookManager = IBookManager.Stub.asInterface(service);
            try {
                List<Book> list = bookManager.getBookList();
                Log.i(TAG, "list Type :" + list.getClass());
                Book newBook = new Book(3, "降龙十八掌");
                bookManager.addBook(newBook);
                Log.i(TAG, "add Book: " + newBook);
                List<Book> newList = bookManager.getBookList();
                Log.i(TAG, "query list : " + newList.toString());
                getCanonicalName();
                Log.i(TAG, "list string : " + list.toString());
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

```

>运行的结果想必大家都知道了，我们成功的添加了一本书；

>现在我们考虑一种情况，假设有一种需求：用户不想时不时地去查询图书列表了，太累了，于是，他去问图书馆，“当有新书时能不能把书的信息告诉我呢？”。大家应该明白了，这就是一种典型的观察者模式，每个感兴趣的用户都观察新书，当新书到的时候，图书馆就通知每一个对这本书感兴趣的用户，这种模式在实际开发中用得很多，下面我们就来模拟这种情形。首先，我们需要提供一个AIDL接口，每个用户都需要实现这个接口并且向图书馆申请新书的提醒功能，当然用户也可以随时取消这种提醒。之所以选择AIDL接口而不是普通接口，是因为AIDL中无法使用普通接口。这里我们创建一个IOnNewBookArrivedListener.aidl文件，我们所期望的情况是：当服务端有新书到来时，就会通知每一个己经申请提醒功能的用户。从程序上来说就是调用所有IOnNewBookArivedListener对象中的onNewBookArived方法，并把新书的对象通过参数传递给客户端，内容如下所示。

```
// IOnNewBookArrivedListener.aidl
package com.liuguilin.ipcsample;

import com.liuguilin.ipcsample.Book;

interface IOnNewBookArrivedListener {
   
   void onNewBookArrived(in Book newBook);
}

```

>除了要新增加一个AIDL外，还需要在原有的接口中添加两个新的方法

```
// IBookManager.aidl
package com.liuguilin.ipcsample;

import com.liuguilin.ipcsample.Book;
import com.liuguilin.ipcsample.IOnNewBookArrivedListener.aidl;

interface IBookManager {

    List<Book>getBookList();
    void addBook(in Book book);
    void registerListener(IOnNewBookArrivedListener listener);
    void unregisterListener(IOnNewBookArrivedListener listener);
}
```

>接下来，我们的服务端也是要稍微的修改一下，每隔5s向感兴趣的用户提醒


```
public class BookManagerService extends Service {

    private static final String TAG = "BookManagerService";

    private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<>();

    private AtomicBoolean mIsServiceDestoryed = new AtomicBoolean(false);

    private CopyOnWriteArrayList<IOnNewBookArrivedListener>mListenerList = new CopyOnWriteArrayList<>();

    private Binder mBinder = new IBookManager.Stub() {

        @Override
        public List<Book> getBookList() throws RemoteException {
            return mBookList;
        }

        @Override
        public void addBook(Book book) throws RemoteException {
            mBookList.add(book);
        }

        @Override
        public void registerListener(IOnNewBookArrivedListener listener) throws RemoteException {
            if(!mListenerList.contains(listener)){
                mListenerList.add(listener);
            }else {
                Log.i(TAG,"already exists");
            }
            Log.i(TAG,"registerListener size:" + mListenerList.size());
        }

        @Override
        public void unregisterListener(IOnNewBookArrivedListener listener) throws RemoteException {
                if(mListenerList.contains(listener)){
                    mListenerList.remove(listener);
                    Log.i(TAG,"remove listener");
                }else {
                    Log.i(TAG,"can not remove listener");
                }
            Log.i(TAG,"unregisterListener size:" + mListenerList.size());
        }
    };

    @Override
    public void onCreate() {
        super.onCreate();

        mBookList.add(new Book(1,"Android"));
        mBookList.add(new Book(1,"IOS"));
        new Thread(new ServiceWorker()).start();
    }

    @Override
    public IBinder onBind(Intent intent) {

        return mBinder;

    }

    @Override
    public void onDestroy() {
        mIsServiceDestoryed.set(true);
        super.onDestroy();
    }

    private class ServiceWorker implements Runnable{

        @Override
        public void run() {

            while (!mIsServiceDestoryed.get()){
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                int bookId = mBookList.size()+1;
                Book newBook = new Book(bookId,"new book#" + bookId);
                try {
                    onNewBookArrived(newBook);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private void onNewBookArrived(Book book) throws RemoteException{
        mBookList.add(book);
        Log.i(TAG,"onNewBookArrived size:" + mListenerList.size());
        for (int i = 0; i < mListenerList.size(); i++) {
            IOnNewBookArrivedListener listener = mListenerList.get(i);
            Log.i(TAG,"listener: "+ listener);
            listener.onNewBookArrived(book);
        }
    }

}


```

>最后，我们还需要修改一下客户端的代码，主要是两方面，首先是客户端要注册IOnNewBookArrivedListener到远程的服务器，这样当有新书时服务端才能通知客户端，同时在我们的Activity的onDestory方法里面去取消绑定，，另一方面，当有新书的时候，服务端会会带哦客户端的Binder线程池中执行，因此，为了便于进行UI操作，我们需要有一个Hnadler可以将其切换到客户端的主线程去执行，这个原理在Binder中已经做了分析了，这里不多说，把代码贴上：

```
package com.liuguilin.ipcsample;

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.Handler;
import android.os.IBinder;
import android.os.Message;
import android.os.RemoteException;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;

import java.util.List;

/**
 * Created by lgl on 16/10/5.
 */

public class BookManagerActivity extends AppCompatActivity {

    private static final String TAG = "BookManagerActivity";

    public static final int MESSAGE_NEW_BOOK_ARRIVED = 1;

    private IBookManager mRemoteBookManager;

    private Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MESSAGE_NEW_BOOK_ARRIVED:
                    Log.i(TAG, "Handler");
                    break;
                default:
                    super.handleMessage(msg);
                    break;
            }
        }
    };

    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            IBookManager bookManager = IBookManager.Stub.asInterface(service);
            try {
                List<Book> list = bookManager.getBookList();
                Log.i(TAG, "list Type :" + list.getClass());
                Book newBook = new Book(3, "降龙十八掌");
                bookManager.addBook(newBook);
                Log.i(TAG, "add Book: " + newBook);
                List<Book> newList = bookManager.getBookList();
                Log.i(TAG, "query list : " + newList.toString());
                getCanonicalName();
                Log.i(TAG, "list string : " + list.toString());
                //注册
                bookManager.registerListener(mOnNew);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            mRemoteBookManager = null;
        }
    };

    private static void getCanonicalName() {
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_book_manager);

        Intent intent = new Intent(this, BookManagerService.class);
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);

    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        unbindService(mConnection);
    }

    private void onNewBookArrived
    throws RemoteException

    {
        mBookList.add(book);
        for (int i = 0; i < mListenerList.size(); i++) {
            IOnNewBookArrivedListener listener = mListenerList.get(i);
            listener.onNewBookArrived(book);
        }
    }

    private class ServiceWorker implements Runnable {

        @Override
        public void run

        {
            while (!mIsServiceDestoryed.get()) {
                try {
                    Thread.sleep(5000);
                } catch (InteruptedException e) {
                    e.toString();
                }

                int bookId = mBookList.size() + 1;
                Book newBook = new Book(bookId, "new book#" + bookId):
                try {
                    onNewBookArrived(newBook);
                } catch () {

                }
            }
        }
    }

}

```		

>最后我们如果运行的话，不难发现，我们的每隔5s新书推送是成功的


>如果你以为AIDL就这样结束了，那你就错了，之前就说过，AIDL远不止这么简单，目前我们还有一些难点还没有涉及

>从上面的代码我们可以看出，当BookManagerActivity关闭时，我们会在onDestory中去接触已经注册的服务端的listener，这就相当于我们不想再接收图书馆的新书提醒，所以我们可以随时退出这个activity

>从上log可以看出，程序没有像我们所预期的那样执行。在解注册的过程中，服务端竟然无法找到我们之前注册的那个listener，在客户端我们注册和解注册时明明传的是同一个listener啊！最终，服务端由于无法找到要解除的listener而宣告解注册失败！这当然不是我们想要的结果，但是仔细想想，好像这种方式的确无法完成解注册。其实，这是必然的，这种解注册的处理方式在日常开发过程中时常使用到，但是放到多进程中却无法奏效，因为Binder会把客户端传递过来的对象重新转化并生成一个新的对象。虽然在注册和解注册过程中使用的是同一个客户端对象，但是通过Binder传递到服务端后会产生两个全新的对象。别忘了对象是不能跨进程直接传输的，对象的跨进程传输本质上都是反序列化的过程，这就是为什么AIDL中的自定义对象都必须要实现Parcelable接口的原因。那么我们要怎么做才能实现解注册的功能尼？答案是用RemoteCallbackList,这看起来很抽象，不过没关系，请看接下来的分析；

> RemoteCallbackList是系统专门用来删除listener的接口，RemoteCallbackList是一个泛型，支持管理任意的AIDL接口，这点从他的声明就可以看出，，因为它的工作原理很简单，在它的内部有一个Map结构专门用来保存所有的AIDL回调，这个Map的key是IBinder类型，value是Callback类型，如下所示。

```
ArrayMap<IBinder,Callback> mCallbacks = new ArrayMap<IBinder,Callback>();

```

>其中Callback中封装了真正的远程listener。当客户端注册listener的时候，它会把这个listener的信息存入mCallbacks中，其中key和value分别通过下面的方式获得：


```
IBinder key = listener.asBinder();
Callback value = new Callback(listener,cookie);

```

>到这里，我相信读者应该都明白了，虽然说多次跨进程传输客户端的同一个对象会在服务端生成不同的对象，但是这些新生成的对象都有一个共同点，那就是他们底层的Binder对象是同一个，利用这些特性，就可以实现我们无法实现的功能了，当客户端解注册的时候，我们只要遍历服务端所有的listener，找到那个和解注册listener具有相同Binder对象的服务端listener并把它删掉，这就是RemoteCallbackList为我们做的事情，同时RemoteCallbackList还有一个很有用的功能，就是当客户端终止后，它能够自动移除客户端的listener,另外，RemoteCallbackList内部自动实现了线程同步的功能，所以我们使用他来注册和解注册，不需要做额外的线程工作，由此可见，RemoteCallbackList是一个很有价值的类，下面我们来演示一下他是如何解注册的

> RemoteCallbackList使用起来很很简单，我们要BookManagerService做一些修改，首先我们创建一个RemoteCallbackList对象来替代之前的CopyonWriteArrayList


```
 private RemoteCallbackList<IOnNewBookArrivedListener>mListenerList = new RemoteCallbackList<IOnNewBookArrivedListener>();

```

>然后修改registerListener 和unregisterListener这两个接口的实现


```
  @Override
        public void registerListener(IOnNewBookArrivedListener listener) throws RemoteException {
            mListenerList.registener(listener);
        }

        @Override
        public void unregisterListener(IOnNewBookArrivedListener listener) throws RemoteException {
            mListenerList.unregistener(listener);
        }
```

>怎么样，使用起来是不是很简单，接下来我们修改onNewBookArrived方法，当有新书的时候，我们就要通知所有已注册的listener

```
private void onNewBookArrived(Book book) throws RemoteException{
        mBookList.add(book);
        final int N = mListenerList.beginBroadcast();
        for (int i = 0; i < N; i++) {
            IOnNewBookArrivedListener l = mListenerList.getBroadcast(i);
            if(i != null){
                l.onNewBookArrived(book);
            }
        }
        mListenerList.finishBroadcast();
    }

```

>BookManagerService的修改已经完毕，为了方便我们验证程序的功能，我们还需要添加一些log，这里我们就不演示了


>到这里，AIDL的基本使用方法已经介绍完了，但是有几点还需要再次说明一下。我们知道，客户端调用远程服务的方法，被调用的方法运行在服务端的Binder线程池中，同时客户端线程会被挂起，这个时候如果服务端方法执行比较耗时，就会导致客户端线程长时间地阻塞在这里，而如果这个客户端线程是UI线程的话，就会导致客户端ANR，这当然不是我们想要看到的。因此，如果我们明确知道某个远程方法是耗时的，那么就要避免在客户端的UI线程中去访问远程方法。由于客户端的onServiceConnected和 onServiceDisconnected方法都运行在UI线程中，所以也不可以在它们里面直接调用服务端的耗时方法，这点要尤其注意。另外，由于服务端的方法本身就运行在服务端的Binder线程池中，所以服务端方法本身就可以执行大量耗时操作，这个时候切记不要在服务端方法中开线程执行异步任务，除非你明确知道自己在干什么，否则不建议这么做。下面我们稍微改造一下服务端getBookList,我们让他耗时

```
	@Override
        public List<Book> getBookList() throws RemoteException {
            SystemClock.sleep(5000);
            return mBookList;
        }
```



