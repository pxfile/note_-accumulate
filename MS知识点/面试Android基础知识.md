面试Android基础知识
===


# Activity之间的通信方式
*   Intent
*   借助类的静态变量
*   借助全局变量/Application
*   借助外部工具 
    – 借助SharedPreference 
    – 使用Android数据库SQLite 
    – 赤裸裸的使用File 
    – Android剪切板
*   借助Service

# Fragment的生命周期，和Activity如何交互
[Android Fragment 真正的完全解析（下）](https://blog.csdn.net/lmj623565791/article/details/37992017)

 [Android Fragment 你应该知道的一切](https://blog.csdn.net/lmj623565791/article/details/42628537)

# service和activity怎么进行数据交互？
* 1.  Activity调用bindService (Intent service, ServiceConnection conn, int flags)方法，得到Service对象的一个引用，这样Activity可以直接调用到Service中的方法，如果要主动通知Activity，我们可以利用回调方法 

* 2.  Service向Activity发送消息，可以使用广播，当然Activity要注册相应的接收器。比如Service要向多个Activity发送同样的消息的话，用这种方法就更好

# [ContentProvider、ContentResolver、ContentObserver之间的关系](http://www.cnblogs.com/anni-qianqian/p/8391887.html)

**ContentProvider、ContentResolver、ContentObserver之间的关系**

ContentPRrovider：

* 四大组件的内容提供者，主要用于对外提供数据

* 实现各个应用程序之间的（跨应用）数据共享，比如联系人应用中就使用了ContentProvider，你在自己的应用中可以读取和修改联系人的数据，不过需要获得相应的权限。其实它也只是一个中间人，真正的数据源是文件或者SQLite等

* 一个应用实现ContentProvider来提供内容给别的应用来操作，通过ContentResolver来操作别的应用数据，当然在自己的应用中也可以

ContentResolver：

* 内容解析者，用于获取内容提供者提供的数据

* ContentResolver.notifyChange（uri）发出消息

ContentObserver：

* 内容监听器，可以监听数据的改变状态

* 目的是观察（捕捉）特定Uri引起的数据库的变化，继而做一些相应的处理，它类似于数据库技术中的触发器（Trigger），当ContentObserver所观察的Uri发生变化时，便会触发它。触发器分为表触发器、行触发器，相应地ContentObsever也分为表ContentObserver、行ContentObserver，当然这是与它所监听的Uri MIME Type有关的

* ContentResolver.registerContentObserver()监听消息

联系：

简单一句话来描述就是：**使用ContentResolver来获取ContentProvider提供的数据，同时注册ContentObserver监听Uri数据的变化**

# AlertDialog,popupWindow,Activity区别

**AlertDialog**：

用来提示用户一些信息,用起来也比较简单,设置标题类容 和按钮即可,如果是加载的自定义的view，调用 dialog.setView(layout);加载布局即可(其他的设置标题 类容 这些就不需要了)

**popupWindow**：

就是一个悬浮在Activity之上的窗口，可以用展示任意布局文件

**Activity**

Activity是Android系统中的四大组件之一，可以用于显示View。
是一个与用户交互的系统模块，几乎所有的Activity都是和用户进行交互的

**区别**：AlertDialog是非阻塞式对话框：AlertDialog弹出时，后台还可以做事情；而PopupWindow是阻塞式对话框：PopupWindow弹出时，程序会等待，在PopupWindow退出前，程序一直等待，只有当我们调用了dismiss方法的后，PopupWindow退出，程序才会向下执行。

这两种区别的表现是：AlertDialog弹出时，背景是黑色的，但是当我们点击背景，AlertDialog会消失，证明程序不仅响应AlertDialog的操作，还响应其他操作，其他程序没有被阻塞，这说明了AlertDialog是非阻塞式对话框；PopupWindow弹出时，背景没有什么变化，但是当我们点击背景的时候，程序没有响应，只允许我们操作PopupWindow，其他操作被阻塞。

# 如何导入外部数据库

把原数据库包括在项目源码的 res/raw

android系统下数据库应该存放在 /data/data/com._._（package name）/ 目录下，所以我们需要做的是把已有的数据库传入那个目录下.操作方法是用FileInputStream读取原数据库，再用FileOutputStream把读取到的东西写入到那个目录.
```
package com.android.ImportDatabase;
 
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
 
import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.os.Environment;
import android.util.Log;
 
public class DBManager {
    private final int BUFFER_SIZE = 400000;
    public static final String DB_NAME = "countries.db"; //保存的数据库文件名
    public static final String PACKAGE_NAME = "com.android.ImportDatabase";
    public static final String DB_PATH = "/data"
            + Environment.getDataDirectory().getAbsolutePath() + "/"
            + PACKAGE_NAME;  //在手机里存放数据库的位置
 
    private SQLiteDatabase database;
    private Context context;
 
    DBManager(Context context) {
        this.context = context;
    }
 
    public void openDatabase() {
        this.database = this.openDatabase(DB_PATH + "/" + DB_NAME);
    }
 
    private SQLiteDatabase openDatabase(String dbfile) {
        try {
            if (!(new File(dbfile).exists())) {　　//判断数据库文件是否存在，若不存在则执行导入，否则直接打开数据库
                InputStream is = this.context.getResources().openRawResource(
                        R.raw.countries); //欲导入的数据库
                FileOutputStream fos = new FileOutputStream(dbfile);
                byte[] buffer = new byte[BUFFER_SIZE];
                int count = 0;
                while ((count = is.read(buffer)) > 0) {
                    fos.write(buffer, 0, count);
                }
                fos.close();
                is.close();
            }
            SQLiteDatabase db = SQLiteDatabase.openOrCreateDatabase(dbfile,
                    null);
            return db;
        } catch (FileNotFoundException e) {
            Log.e("Database", "File not found");
            e.printStackTrace();
        } catch (IOException e) {
            Log.e("Database", "IO exception");
            e.printStackTrace();
        }
        return null;
    }
//do something else here
    public void closeDatabase() {
        this.database.close();
    }
}
```
# 谈谈对接口与回调的理解
回调是基于接口实现的

**回调其实是一种双向调用模式，也就说调用方在接口被调用时也会调用对方的接口，实现方法交还给提供接口的父类处理！**

**为什么要用回调**

* 我们都知道Java是一门面向对象的语言，有一句很著名的话就是”万事万物皆为对象”，我们把普通事物的共性抽取出来，而这些共性之中又充斥着特性，每个不同的特性就需要交给特定的情况处理，通过暴露接口方法可以减少很多重复，代码更加优雅。

打个比方，Button、ImageButton等都具有可被点击的共性，但是被点击之后相关事件的处理是不同的，比如说我想我要点击的这个Button弹出一个消息提示，然而我希望我的ImageButton点击之后可以弹出一个Notifaction通知，这个时候回调方法的好处就体现出来了，因为android对外暴露的OnClickListener()
接口中含有一个OnClick()
方法，你需要怎样的具体实现都由你自己定义，而这个回调方法的所在类View
不会管你怎么实现的，它只负责调用这个回调方法，这就是使用回调的好处。

# Android动画框架实现原理

**Animation**

实现原理是每次绘制视图时View所在的ViewGroup中的drawChild函数获取该View的Animation的Transformation值，然后调用canvas.concat(transformToApply.getMatrix())，通过矩阵运算完成动画帧，如果动画没有完成，继续调用invalidate()函数，启动下次绘制来驱动动画，动画过程中的帧之间间隙时间是绘制函数所消耗的时间，可能会导致动画消耗比较多的CPU资源，最重要的是，动画改变的只是显示，并不能相应事件。

**Animator**

属性动画框架操作的是真实的属性值，直接变化了对象的属性

# LaunchMode应用场景
singleTop适合接收通知启动的内容显示页面。

例如，某个新闻客户端的新闻内容页面，如果收到10个新闻推送，每次都打开一个新闻内容页面是很烦人的。

singleTask适合作为程序入口点。

例如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走onNewIntent，并且会清空主界面上面的其他页面。

singleInstance应用场景：

闹铃的响铃界面。 你以前设置了一个闹铃：上午6点。在上午5点58分，你启动了闹铃设置界面，并按 Home 键回桌面；在上午5点59分时，你在微信和朋友聊天；在6点时，闹铃响了，并且弹出了一个对话框形式的 Activity(名为 AlarmAlertActivity) 提示你到6点了(这个 Activity 就是以 SingleInstance 加载模式打开的)，你按返回键，回到的是微信的聊天界面，这是因为 AlarmAlertActivity 所在的 Task 的栈只有他一个元素， 因此退出之后这个 Task 的栈空了。如果是以 SingleTask 打开 AlarmAlertActivity，那么当闹铃响了的时候，按返回键应该进入闹铃设置界面。

singleInstance不要用于中间页面，如果用于中间页面，[跳转会有问题](http://stackoverflow.com/questions/17337536/android-launch-mode-single-instance)，比如：A -> B (singleInstance) -> C，完全退出后，在此启动，首先打开的是B。

# Android：
---
**五种布局： FrameLayout 、 LinearLayout 、 AbsoluteLayout 、 RelativeLayout 、 TableLayout 全都继承自ViewGroup，各自特点及绘制效率对比。**

* FrameLayout(框架布局)

    此布局是五种布局中最简单的布局，Android中并没有对child view的摆布进行控制，这个布局中所有的控件都会默认出现在视图的左上角，我们可以使用``android:layout_margin``，``android:layout_gravity``等属性去控制子控件相对布局的位置。

* LinearLayout(线性布局)

    一行（或一列）只控制一个控件的线性布局，所以当有很多控件需要在一个界面中列出时，可以用LinearLayout布局。
    此布局有一个需要格外注意的属性:``android:orientation=“horizontal|vertical``。
    
    * 当`android:orientation="horizontal`时，*说明你希望将水平方向的布局交给**LinearLayout** *，其子元素的`android:layout_gravity="right|left"` 等控制水平方向的gravity值都是被忽略的，*此时**LinearLayout**中的子元素都是默认的按照水平从左向右来排*，我们可以用`android:layout_gravity="top|bottom"`等gravity值来控制垂直展示。
    * 反之，可以知道 当`android:orientation="vertical`时，**LinearLayout**对其子元素展示上的的处理方式。

* AbsoluteLayout(绝对布局)

    可以放置多个控件，并且可以自己定义控件的x,y位置

* RelativeLayout(相对布局)

    这个布局也是相对自由的布局，Android 对该布局的child view的 水平layout& 垂直layout做了解析，由此我们可以FrameLayout的基础上使用标签或者Java代码对垂直方向 以及 水平方向 布局中的views进行任意的控制.
    
    * 相关属性：
    
    ```
    
    　　android:layout_centerInParent="true|false"
	　　android:layout_centerHorizontal="true|false"
	　　android:layout_alignParentRight="true|false"
	　　
	```

* TableLayout(表格布局)

    将子元素的位置分配到行或列中，一个TableLayout由许多的TableRow组成
    

---


**Activity生命周期。**

* 启动Activity:
  onCreate()—>onStart()—>onResume()，Activity进入运行状态。
* Activity退居后台:
  当前Activity转到新的Activity界面或按Home键回到主屏：
  onPause()—>onStop()，进入停滞状态。
* Activity返回前台:
  onRestart()—>**onStart()**—>onResume()，再次回到运行状态。
* Activity退居后台，且系统内存不足，
  系统会杀死这个后台状态的Activity（此时这个Activity引用仍然处在任务栈中，只是这个时候引用指向的对象已经为null），若再次回到这个Activity,则会走onCreate()–>onStart()—>onResume()(将重新走一次Activity的初始化生命周期)
* 锁屏：`onPause()->onStop()`
* 解锁：`onStart()->onResume()`
  
* 更多流程分支，请参照以下生命周期流程图
	![](http://img.blog.csdn.net/20130828141902812)


---


**通过Acitivty的xml标签来改变任务栈的默认行为**

* 使用``android:launchMode="standard|singleInstance|singleTask|singleTop"``来控制Acivity任务栈。

    **任务栈**是一种后进先出的结构。位于栈顶的Activity处于焦点状态,当按下back按钮的时候,栈内的Activity会一个一个的出栈,并且调用其``onDestory()``方法。如果栈内没有Activity,那么系统就会回收这个栈,每个APP默认只有一个栈,以APP的包名来命名.

    * standard : 标准模式,每次启动Activity都会创建一个新的Activity实例,并且将其压入任务栈栈顶,而不管这个Activity是否已经存在。Activity的启动三回调(*onCreate()->onStart()->onResume()*)都会执行。
    - singleTop : 栈顶复用模式.这种模式下,如果新Activity已经位于任务栈的栈顶,那么此Activity不会被重新创建,所以它的启动三回调就不会执行,同时Activity的``onNewIntent()``方法会被回调.如果Activity已经存在但是不在栈顶,那么作用与*standard模式*一样.
    - singleTask: 栈内复用模式.创建这样的Activity的时候,系统会先确认它所需任务栈已经创建,否则先创建任务栈.然后放入Activity,如果栈中已经有一个Activity实例,那么这个Activity就会被调到栈顶,``onNewIntent()``,并且singleTask会清理在当前Activity上面的所有Activity.(clear top)
    - singleInstance : 加强版的singleTask模式,这种模式的Activity只能单独位于一个任务栈内,由于栈内复用的特性,后续请求均不会创建新的Activity,除非这个独特的任务栈被系统销毁了

Activity的堆栈管理以ActivityRecord为单位,所有的ActivityRecord都放在一个List里面.可以认为一个ActivityRecord就是一个Activity栈

---

**Activity缓存方法。**

有a、b两个Activity，当从a进入b之后一段时间，可能系统会把a回收，这时候按back，执行的不是a的onRestart而是onCreate方法，a被重新创建一次，这是a中的临时数据和状态可能就丢失了。

可以用Activity中的onSaveInstanceState()回调方法保存临时数据和状态，这个方法一定会在活动被回收之前调用。方法中有一个Bundle参数，putString()、putInt()等方法需要传入两个参数，一个键一个值。数据保存之后会在onCreate中恢复，onCreate也有一个Bundle类型的参数。

示例代码：

```
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //这里，当Acivity第一次被创建的时候为空
        //所以我们需要判断一下
        if( savedInstanceState != null ){
            savedInstanceState.getString("anAnt");
        }
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);

        outState.putString("anAnt","Android");

    }

```

一、onSaveInstanceState (Bundle outState)

当某个activity变得“容易”被系统销毁时，该activity的onSaveInstanceState就会被执行，除非该activity是被用户主动销毁的，例如当用户按BACK键的时候。

注意上面的双引号，何为“容易”？言下之意就是该activity还没有被销毁，而仅仅是一种可能性。这种可能性有哪些？通过重写一个activity的所有生命周期的onXXX方法，包括onSaveInstanceState和onRestoreInstanceState方法，我们可以清楚地知道当某个activity（假定为activity A）显示在当前task的最上层时，其onSaveInstanceState方法会在什么时候被执行，有这么几种情况：

1、当用户按下HOME键时。

这是显而易见的，系统不知道你按下HOME后要运行多少其他的程序，自然也不知道activity A是否会被销毁，故系统会调用onSaveInstanceState，让用户有机会保存某些非永久性的数据。以下几种情况的分析都遵循该原则

2、长按HOME键，选择运行其他的程序时。

3、按下电源按键（关闭屏幕显示）时。

4、从activity A中启动一个新的activity时。

5、屏幕方向切换时，例如从竖屏切换到横屏时。（如果不指定configchange属性）
在屏幕切换之前，系统会销毁activity A，在屏幕切换之后系统又会自动地创建activity A，所以onSaveInstanceState一定会被执行
 
总而言之，onSaveInstanceState的调用遵循一个重要原则，即当系统“未经你许可”时销毁了你的activity，则onSaveInstanceState会被系统调用，这是系统的责任，因为它必须要提供一个机会让你保存你的数据（当然你不保存那就随便你了）。另外，需要注意的几点：
 
1.布局中的每一个View默认实现了onSaveInstanceState()方法，这样的话，这个UI的任何改变都会自动地存储和在activity重新创建的时候自动地恢复。但是这种情况只有在你为这个UI提供了唯一的ID之后才起作用，如果没有提供ID，app将不会存储它的状态。
 
2.由于默认的onSaveInstanceState()方法的实现帮助UI存储它的状态，所以如果你需要覆盖这个方法去存储额外的状态信息，你应该在执行任何代码之前都调用父类的onSaveInstanceState()方法（super.onSaveInstanceState()）。
既然有现成的可用，那么我们到底还要不要自己实现onSaveInstanceState()?这得看情况了，如果你自己的派生类中有变量影响到UI，或你程序的行为，当然就要把这个变量也保存了，那么就需要自己实现，否则就不需要。

3.由于onSaveInstanceState()方法调用的不确定性，你应该只使用这个方法去记录activity的瞬间状态（UI的状态）。不应该用这个方法去存储持久化数据。当用户离开这个activity的时候应该在onPause()方法中存储持久化数据（例如应该被存储到数据库中的数据）。
 
4.onSaveInstanceState()如果被调用，这个方法会在onStop()前被触发，但系统并不保证是否在onPause()之前或者之后触发。
 
 
二、onRestoreInstanceState (Bundle outState)

至于onRestoreInstanceState方法，需要注意的是，onSaveInstanceState方法和onRestoreInstanceState方法“不一定”是成对的被调用的，（本人注：我昨晚调试时就发现原来不一定成对被调用的！）
 
onRestoreInstanceState被调用的前提是，activity A“确实”被系统销毁了，而如果仅仅是停留在有这种可能性的情况下，则该方法不会被调用，例如，当正在显示activity A的时候，用户按下HOME键回到主界面，然后用户紧接着又返回到activity A，这种情况下activity A一般不会因为内存的原因被系统销毁，故activity A的onRestoreInstanceState方法不会被执行

另外，onRestoreInstanceState的bundle参数也会传递到onCreate方法中，你也可以选择在onCreate方法中做数据还原。
还有onRestoreInstanceState在onstart之后执行。
至于这两个函数的使用，给出示范代码（留意自定义代码在调用super的前或后）：

```
@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
        savedInstanceState.putBoolean("MyBoolean", true);
        savedInstanceState.putDouble("myDouble", 1.9);
        savedInstanceState.putInt("MyInt", 1);
        savedInstanceState.putString("MyString", "Welcome back to Android");
        // etc.
        super.onSaveInstanceState(savedInstanceState);
}

@Override
public void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);

        boolean myBoolean = savedInstanceState.getBoolean("MyBoolean");
        double myDouble = savedInstanceState.getDouble("myDouble");
        int myInt = savedInstanceState.getInt("MyInt");
        String myString = savedInstanceState.getString("MyString");
}

```

---



**Fragment的生命周期和activity如何的一个关系**

这我们引用本知识库里的一张图片：
![Mou icon](https://github.com/GeniusVJR/LearningNotes/blob/master/Part1/Android/FlowchartDiagram.jpg?raw=true)


**为什么在Service中创建子线程而不是Activity中**

这是因为Activity很难对Thread进行控制，当Activity被销毁之后，就没有任何其它的办法可以再重新获取到之前创建的子线程的实例。而且在一个Activity中创建的子线程，另一个Activity无法对其进行操作。但是Service就不同了，所有的Activity都可以与Service进行关联，然后可以很方便地操作其中的方法，即使Activity被销毁了，之后只要重新与Service建立关联，就又能够获取到原有的Service中Binder的实例。因此，使用Service来处理后台任务，Activity就可以放心地finish，完全不需要担心无法对后台任务进行控制的情况。


**Intent的使用方法，可以传递哪些数据类型。**

通过查询Intent/Bundle的API文档，我们可以获知，Intent/Bundle支持传递基本类型的数据和基本类型的数组数据，以及String/CharSequence类型的数据和String/CharSequence类型的数组数据。而对于其它类型的数据貌似无能为力，其实不然，我们可以在Intent/Bundle的API中看到Intent/Bundle还可以传递Parcelable（包裹化，邮包）和Serializable（序列化）类型的数据，以及它们的数组/列表数据。

所以要让非基本类型和非String/CharSequence类型的数据通过Intent/Bundle来进行传输，我们就需要在数据类型中实现Parcelable接口或是Serializable接口。

[http://blog.csdn.net/kkk0526/article/details/7214247](http://blog.csdn.net/kkk0526/article/details/7214247)

**Fragment生命周期**

![](http://7xntdm.com1.z0.glb.clouddn.com/fragment_lifecycle.png)

注意和Activity的相比的区别,按照执行顺序

* onAttach(),onDetach()
* onCreateView(),onDestroyView()

---

**Service的两种启动方法，有什么区别**
1.在Context中通过``public boolean bindService(Intent service,ServiceConnection conn,int flags)`` 方法来进行Service与Context的关联并启动，并且Service的生命周期依附于Context(**不求同时同分同秒生！但求同时同分同秒屎！！**)。

2.通过`` public ComponentName startService(Intent service)``方法去启动一个Service，此时Service的生命周期与启动它的Context无关。

3.要注意的是，whatever，**都需要在xml里注册你的Service**，就像这样:
```
<service
        android:name=".packnameName.youServiceName"
        android:enabled="true" />
```

**广播(Broadcast Receiver)的两种动态注册和静态注册有什么区别。**

* 静态注册：在AndroidManifest.xml文件中进行注册，当App退出后，Receiver仍然可以接收到广播并且进行相应的处理
* 动态注册：在代码中动态注册，当App退出后，也就没办法再接受广播了

---


**ContentProvider使用方法**

[http://blog.csdn.net/juetion/article/details/17481039](http://blog.csdn.net/juetion/article/details/17481039)

---

**目前能否保证service不被杀死**

**Service设置成START_STICKY**

* kill 后会被重启（等待5秒左右），重传Intent，保持与重启前一样

**提升service优先级**
 * 在AndroidManifest.xml文件中对于intent-filter可以通过``android:priority = "1000"``这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，**同时适用于广播**。
 * 【结论】目前看来，priority这个属性貌似只适用于broadcast，对于Service来说可能无效

**提升service进程优先级**
 * Android中的进程是托管的，当系统进程空间紧张的时候，会依照优先级自动进行进程的回收
 * 当service运行在低内存的环境时，将会kill掉一些存在的进程。因此进程的优先级将会很重要，可以在startForeground()使用startForeground()将service放到前台状态。这样在低内存时被kill的几率会低一些。
 * 【结论】如果在极度极度低内存的压力下，该service还是会被kill掉，并且不一定会restart()

**onDestroy方法里重启service**
 * service +broadcast  方式，就是当service走onDestory()的时候，发送一个自定义的广播，当收到广播的时候，重新启动service
 * 也可以直接在onDestroy()里startService
 * 【结论】当使用类似口口管家等第三方应用或是在setting里-应用-强制停止时，APP进程可能就直接被干掉了，onDestroy方法都进不来，所以还是无法保证

**监听系统广播判断Service状态**
 * 通过系统的一些广播，比如：手机重启、界面唤醒、应用状态改变等等监听并捕获到，然后判断我们的Service是否还存活，别忘记加权限
 * 【结论】这也能算是一种措施，不过感觉监听多了会导致Service很混乱，带来诸多不便

**在JNI层,用C代码fork一个进程出来**

* 这样产生的进程,会被系统认为是两个不同的进程.但是Android5.0之后可能不行

**root之后放到system/app变成系统级应用**

**大招: 放一个像素在前台(手机QQ)**

---


**动画有哪两类，各有什么特点？三种动画的区别**

* tween 补间动画。通过指定View的初末状态和变化时间、方式，对View的内容完成一系列的图形变换来实现动画效果。
Alpha
Scale
Translate
Rotate。

* frame 帧动画
AnimationDrawable 控制
animation-list xml布局

* PropertyAnimation 属性动画
 
---

**Android的数据存储形式。**



* SQLite：SQLite是一个轻量级的数据库，支持基本的SQL语法，是常被采用的一种数据存储方式。
Android为此数据库提供了一个名为SQLiteDatabase的类，封装了一些操作数据库的api

* SharedPreference： 除SQLite数据库外，另一种常用的数据存储方式，其本质就是一个xml文件，常用于存储较简单的参数设置。

* File： 即常说的文件（I/O）存储方法，常用语存储大数量的数据，但是缺点是更新数据将是一件困难的事情。

* ContentProvider: Android系统中能实现所有应用程序共享的一种数据存储方式，由于数据通常在各应用间的是互相私密的，所以此存储方式较少使用，但是其又是必不可少的一种存储方式。例如音频，视频，图片和通讯录，一般都可以采用此种方式进行存储。每个Content Provider都会对外提供一个公共的URI（包装成Uri对象），如果应用程序有数据需要共享时，就需要使用Content Provider为这些数据定义一个URI，然后其他的应用程序就通过Content Provider传入这个URI来对数据进行操作。

---

**Sqlite的基本操作。**


[http://blog.csdn.net/zgljl2012/article/details/44769043](http://blog.csdn.net/zgljl2012/article/details/44769043)

---

**如何判断应用被强杀**

在Application中定义一个static常量，赋值为－1，在欢迎界面改为0，如果被强杀，application重新初始化，在父类Activity判断该常量的值。

**应用被强杀如何解决**

如果在每一个Activity的onCreate里判断是否被强杀，冗余了，封装到Activity的父类中，如果被强杀，跳转回主界面，如果没有被强杀，执行Activity的初始化操作，给主界面传递intent参数，主界面会调用onNewIntent方法，在onNewIntent跳转到欢迎页面，重新来一遍流程。

**Json有什么优劣势。**

**怎样退出终止App**

**Asset目录与res目录的区别。**
res 目录下面有很多文件，例如 drawable,mipmap,raw 等。res 下面除了 raw 文件不会被压缩外，其余文件都会被压缩。同时 res目录下的文件可以通过R 文件访问。Asset 也是用来存储资源，但是 asset 文件内容只能通过路径或者 AssetManager 读取。 [官方文档](https://developer.android.com/studio/projects/index.html)

**Android怎么加速启动Activity。**
分两种情况，启动应用 和 普通Activity
启动应用 ：Application 的构造方法，onCreate 方法中不要进行耗时操作，数据预读取(例如 init 数据) 放在异步中操作
启动普通的Activity：A 启动B 时不要在 A 的 onPause 中执行耗时操作。因为 B 的 onResume 方法必须等待 A 的 onPause 执行完成后才能运行

**Android内存优化方法：ListView优化，及时关闭资源，图片缓存等等。**

**Android中弱引用与软引用的应用场景。**
软引用：缓存
弱引用：内部类，缓存（glide图片库）

**Bitmap的四种属性，与每种属性队形的大小。**


**View与View Group分类。自定义View过程：onMeasure()、onLayout()、onDraw()。**

如何自定义控件：

1. 自定义属性的声明和获取

    * 分析需要的自定义属性
    * 在res/values/attrs.xml定义声明
    * 在layout文件中进行使用
    * 在View的构造方法中进行获取       
2. 测量onMeasure
3. 布局onLayout(ViewGroup)
4. 绘制onDraw
5. onTouchEvent
6. onInterceptTouchEvent(ViewGroup)
7. 状态的恢复与保存


**Android长连接，怎么处理心跳机制。**

---

**View树绘制流程**

---

**下拉刷新实现原理**

---

**你用过什么框架，是否看过源码，是否知道底层原理。**

Retrofit

EventBus

glide

---

**Android5.0、6.0新特性。**

Android5.0新特性：

* MaterialDesign设计风格
* 支持多种设备
* 支持64位ART虚拟机

Android6.0新特性

* 大量漂亮流畅的动画
* 支持快速充电的切换
* 支持文件夹拖拽应用
* 相机新增专业模式

Android7.0新特性

* 分屏多任务
* 增强的Java8语言模式
* 夜间模式

---


**Context区别**

* Activity和Service以及Application的Context是不一样的,Activity继承自ContextThemeWraper.其他的继承自ContextWrapper
* 每一个Activity和Service以及Application的Context都是一个新的ContextImpl对象
* getApplication()用来获取Application实例的，但是这个方法只有在Activity和Service中才能调用的到。那么也许在绝大多数情况下我们都是在Activity或者Service中使用Application的，但是如果在一些其它的场景，比如BroadcastReceiver中也想获得Application的实例，这时就可以借助getApplicationContext()方法，getApplicationContext()比getApplication()方法的作用域会更广一些，任何一个Context的实例，只要调用getApplicationContext()方法都可以拿到我们的Application对象。
* Activity在创建的时候会new一个ContextImpl对象并在attach方法中关联它，Application和Service也差不多。ContextWrapper的方法内部都是转调ContextImpl的方法
* 创建对话框传入Application的Context是不可以的
* 尽管Application、Activity、Service都有自己的ContextImpl，并且每个ContextImpl都有自己的mResources成员，但是由于它们的mResources成员都来自于唯一的ResourcesManager实例，所以它们看似不同的mResources其实都指向的是同一块内存
* Context的数量等于Activity的个数 + Service的个数 + 1，这个1为Application

---

**IntentService的使用场景与特点。**

>IntentService是Service的子类，是一个异步的，会自动停止的服务，很好解决了传统的Service中处理完耗时操作忘记停止并销毁Service的问题

优点：

* 一方面不需要自己去new Thread
* 另一方面不需要考虑在什么时候关闭该Service

onStartCommand中回调了onStart，onStart中通过mServiceHandler发送消息到该handler的handleMessage中去。最后handleMessage中回调onHandleIntent(intent)。

---


**图片缓存**

查看每个应用程序最高可用内存：

```
    int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);  
    Log.d("TAG", "Max memory is " + maxMemory + "KB");  
```

---

**Gradle**

构建工具、Groovy语法、Java

Jar包里面只有代码，aar里面不光有代码还包括代码还包括资源文件，比如 drawable 文件，xml 资源文件。对于一些不常变动的 Android Library，我们可以直接引用 aar，加快编译速度

---

**你是如何自学Android**

首先是看书和看视频敲代码，然后看大牛的博客，做一些项目，向github提交代码，觉得自己API掌握的不错之后，开始看进阶的书，以及看源码，看完源码学习到一些思想，开始自己造轮子，开始想代码的提升，比如设计模式，架构，重构等。

---




