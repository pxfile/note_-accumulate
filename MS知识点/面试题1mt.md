mt面试题1
===

1.打开一个activity 这时来了一个电话 那这个activity都经历了哪些生命周期？
2.平时是怎么使用activity生命周期的
3.Activity的四种LaunchMode?
4.编程中使用过SparseArray么?
5.ListView优化方法?
6.Android如何使用新线程处理UI(Handler消息传送)?
7.Android的编译打包过程?
8.如何使用SharedPreference进行数据储存?
9.众多的drawable文件夹都有什么不同?
10.数据库升级需要注意什么?
11.在gradle的配置文件中 versionCode versionName minSdkVersion targetSdkVersion 各代表什么含义?
12.如何在android中优雅的使用SQLite?
13.如何正确使用LoaderManager ?
14. ImageView 为了适应各种显示场景 如何变换图片的显示方式?
15. 是否使用过Fragment 在Fragment中是否出现过getActivity() 为空的问题 什么原因 如何解决?
16.是否使用过GSON解析Json数据 如何解析服务器返回的下列数据？
17.android的官方建议应用程序的开发采用mvc模式。何谓mvc?
18.android中得ANR是什么  如何避免?
19.

## 打开一个activity 这时来了一个电话 那这个activity都经历了哪些生命周期？或者是按back键的时候呢？或者是按Home键的时候呢？

* activity的生命周期 有以下几个

onCreate() onStart() onResume() onPause() onRestart() onStop() onDestroy()

* 首先说一下一个正常的activity 从打开到按返回键退出经历的生命周期

onCreate() ---> onStart() ---> onResume() --->onPause() --->onStop() ---> onDestroy()

* 创建activity时生命周期

首先会回调onCreate()（一次）---> 启动activity时会回调onStart()（可见不可交互）--->恢复activity时回调onResume() （onStart()之后一定会回调）---> 暂停时回调onPause() ---> 停止时回调onStop() ---> 销毁时调用onDestroy() （back键 或者系统资源不足kill掉低优先级别的活动）

* 然后说一下activity 从打开到按Home键再切换回来经历的生命周期

onCreate() ---> onStart() ---> onResume() ---> onPause() ---> onStop()---> onRestart() ---> onStart()--->onResume() 

* 电话打进来经历的生命周期

onCreate() ---> onStart() ---> onResume() ---> onPause() ---> onStop()

## 平时是怎么使用activity生命周期的

* 在onCreate() 里面加载资源

比如listView=(ListView)findViewById(R.id.listview);（不做耗时任务）

* onResume() 与 onPause() 两个方法中 执行一些对称的操作

比如暂停时保存一些用户数据 恢复时读取出来 恢复时创建资源 暂停时释放一些资源

* 还有onStart() 不要在方法内执行很耗时的操作 这个特别重要 因为这个时期是可见不可交互的 时间越长 体验越差

* 还有activity没有被销毁的时候 重新启动时会回调onRestart() 

* 还有一些情况 比如 使用户activity不可见时 经历onPause() ---> onStop() 电话打(新启动一个其他activity)进来就是这样的情况 遮挡住了当前的activity

* AlertDialog这个要特别说明一下，这个是不影响activity的生命周期的

## Activity的四种LaunchMode?

Activity一共有以下四种launchMode：

* standard 

默认模式 总是创建新的A实例 同一个任务可以有多个A的实例

A 为 standard 模式

堆栈 : A1->A2->A3

再次启动 A 

堆栈 : A1->A2->A3->A4

* singleTop

类似于standard 不过 当堆栈顶部是B的实例时 不会创建新的B实例

B 为 singleTop 

 堆栈 : A1->B1->A2->B2

再次启动 B

堆栈 : A1->B1->A2->B2

* singleTask

B1在新的task创建C的实例 C可以在自己的task中创建B2和A2 A2启动C时 不会创建新的C实例 而是直接转到C的当前实例

并且 C返回时 直接返回启动C的B1 而不是转入C之前的A2

C 为 singleTask

堆栈 : A1->B1

启动 C 

堆栈：A1->B1->C

依次启动  B  A

堆栈：A1->B1
                       \
                       C ->B2->A2

再次启动 C 

堆栈：A1->B1
                       \
                       C 

* singleInstance

类似于singleTask 但新的task只能有D一个实例 D启动的B2会在原来的task创建

B2无法返回D  而是返回到B1  A1退出后 可以看到D还在

D 为 singleInstance

依次启动A B D

堆栈：A1->B1  
          /            
       D

*D并启动后没有在A B 所在栈 而且指向也不在B

依次启动B A 

堆栈：A1->B1 - - - B2->A2
           /  
         D

连续两次返回

堆栈：A1->B1
          /  
        D

再次返回

堆栈：A1
          /  
        D

再次返回

堆栈：D  

再次返回

堆栈：A1  *退回初始栈

singleInstance 应用 share应用 多个程序调用只实例化一次 且不与其他任何任务处于相同栈

## 编程中使用过SparseArray么?

SparseArray是Android专门为HashMap<Integer,Object>类型的数据进行专门写的类 目的是提高效率, 之前使用的是
```
HashMap<Integer, E> hashMap = new HashMap<Integer, E>();
```
现在我们可以使用
```
SparseArray<E> sparseArray = new SparseArray<E>();
```
来获取更高的性能

相应的也有SparseBooleanArray 用来取代HashMap<Integer, Boolean> SparseIntArray用来取代HashMap<Integer, Integer>

**内部原理:稀疏数组**

所谓**稀疏数组**就是数组中大部分的内容值都未被使用（或都为零） 在数组中仅有少部分的空间使用 因此造成内存空间的浪费 为了节省内存空间 并且不影响数组中原有的内容值 我们可以采用一种压缩的方式来表示稀疏数组的内容。

假设有一个9*7的数组 其内容如下：

在此数组中 共有63个空间 但却只使用了5个元素 造成58个元素空间的浪费 以下我们就使用稀疏数组重新来定义这个数组：

其中在稀疏数组中第一部分所记录的是原数组的列数和行数以及元素使用的个数

第二部分所记录的是原数组中元素的位置和内容,经过压缩之后,原来需要声明大小为63的数组而使用压缩后只需要声明大小为6*3的数组 仅需18个存储空间

它有两个方法可以添加键值对：
```
public void put(int key, E value) {} 
public void append(int key, E value){}
```
有四个方法可以执行删除操作:
```
public void delete(int key) {} 
public void remove(int key) {} 
public void removeAt(int index){} 
public void clear(){}
```
查找数据:
```
public E get(int key)
public E get(int key, E valueIfKeyNotFound)
```
查找键值
```
public int keyAt(int index)
```

注意上面的keyAt 因为内部使用了二分查找 找不到时返回小于0的数值，而不是返回-1

## ListView优化方法?
考点: 说出原理 +写出一个简单的getView方法

* 1.分页

* 2.convertView+ViewHolder

**convertView**

Adapter 类中 getView方法是重中之重 因为listView最主要的显示和处理都在这里面 convertView是getView中的参数

这是一个缓存机制 缓存可视范围中的view 上下滚动列表时 只创建之前缓存内没有的view 删除可视范围外的view 并形成新的缓存

缓存效果和不使用缓存相差特别大 

**ViewHolder**

将view视图保存 保存的是指向之前视图的引用 避免重复调用findViewById 影响效率

有效果 但效果不大 总得来说还是convertView的效果极其明显

下面是一个使用了convertView+ViewHolder的代码

    @Override
    public View getView(finalint position, View convertView, ViewGroup parent) {
         ViewHolder holder;
        if (convertView == null) {
                 convertView = mInflater.inflate(R.layout.item,null);
                 holder = new ViewHolder();
                /**得到各个控件的对象*/                   
                holder.title = (TextView) convertView.findViewById(R.id.ItemTitle);
                holder.text = (TextView) convertView.findViewById(R.id.ItemText);
                holder.bt = (Button) convertView.findViewById(R.id.ItemButton);
                convertView.setTag(holder);//绑定ViewHolder对象                  
        } else {
                holder = (ViewHolder)convertView.getTag();//取出ViewHolder对象                 
        }
        /**设置TextView显示的内容，即我们存放在动态数组中的数据*/           
        holder.title.setText(getDate().get(position).get("ItemTitle").toString());
        holder.text.setText(getDate().get(position).get("ItemText").toString());
          
        /**为Button添加点击事件*/            
        holder.bt.setOnClickListener(new OnClickListener() {
            @Override
            publicvoid onClick(View v) {
                Log.v("MyListViewBase", "你点击了按钮" + position);//打印Button的点击信息                   
            }
        });
          
        return convertView;
    }
  
}
  
/**存放控件*/
public final class ViewHolder{
    public TextView title;
    public TextView text;
    public Button   bt;
}
 
## Android如何使用新线程处理UI(Handler消息传送)?

Android里的消息处理 涉及到Handler Looper Message MessageQueue

**Message**：消息 其中包含了消息ID 消息处理对象以及处理的数据等 由MessageQueue统一列队 终由Handler处理 

**Handler**：处理者 负责Message的发送及处理 使用Handler时 需要实现handleMessage(Message msg)方法来对特定的Message进行处理 例如更新UI等 

**MessageQueue**：消息队列 用来存放Handler发送过来的消息 并按照FIFO规则执行 当然 存放Message并非实际意义的保存 而是将Message以链表的方式串联起来的 等待Looper的抽取

**Looper**：一个线程可以产生一个Looper对象 用来管理MessageQueue 它就像一个消息泵 不断地从MessageQueue中抽取Message执行 因此 一个MessageQueue需要一个Looper

使用Handler可以发送和处理消息 或者是发送处理绑定在线程消息队列中的rannable对象 每一个Handler实例都联系到一个单一的线程和那个线程的消息队列中

当创建一个新的Handler 他将绑定到这个线程或者是创建Hanlder的线程的消息队列 从这一点上来看 它会传递信息和消息队列的rannables并在消息弹出消息队列时执行它们

举个例子 当我们启动一个Android应用程序的时候 Android会首先开启一个主线程 这个主线程的工作主要就是管理界面中的UI控件 进行事件分发 处理消息响应函数等

但是如果处理一个比较耗时的操作时 比如读取本地大文件 读取网络数据等等时 如果依然用主线程的话 就会出现问题 Android系统规定默认5S无反应的话 就会提示无响应(ANR) 

在这个时候我们就需要另外开一个线程来处理耗时的工作 但是因为子线程涉及到UI更新 而更新UI只能在主线程中更新（Android主线程不是线程安全的） 子线程中操作是危险的

Handler就是用来解决这个复杂问题而出现的 Handler运行在主线程中(UI线程)中  它与子线程可以通过Message对象来传递数据 

这个时候 Handler就承担着接受子线程使用sendMessage()方法传递的Message对象(里面包含数据) 把这些消息放入主线程队列中 配合主线程进行更新UI 

下面举个使用handler刷新UI的例子:
```
public class HandlerDemo extends Activity {
     
    private int title = 0;
    private Handler mHandler = new Handler(){      
        public void handleMessage(Message msg) {
            switch (msg.what) {
            case 1:
                updateTitle();
                break;
            }
        };
    };
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
         
        Timer timer = new Timer();
        timer.scheduleAtFixedRate(new MyTask(), 1, 5000);
    }
         
    private class MyTask extends TimerTask{
        @Override
        public void run() {
            Message message = new Message();
            message.what = 1;
            mHandler.sendMessage(message);
             
        }  
    }
    public void updateTitle(){     
        setTitle("refresh "+title+" times");
        title ++;
    }
}
```
## Android的编译打包过程?

* 1. 生成R.java类文件： 

解压第三方lib 与源程序 使用aapt生成R.java 

* 2.编译.java类文件生成class文件： 

需要解决依赖冲突 包冲突等问题

* 3.混淆处理 生成单一jar包

proguard混淆  所有依赖的jar放在一起 形成混淆后的单一jar包

* 4.将class文件打包生成classes.dex文件： 

dx命令行脚本生成classes.dex文件 delvik可以识别这种文件

* 5.打包资源文件（包括res、assets、androidmanifest.xml等）： 

aapt生成资源包文件 过程比较复杂 重点是R文件中会储存所有资源的ID resources.arsc会储存资源的索引表

* 6.生成未签名的apk安装文件：

apk builder命令脚本生成未签名的apk安装文件。

* 7.对未签名的apk进行签名生成签名后的android文件： 

jar signer对未签名的包进行apk签名

## 如何使用SharedPreference进行数据储存?
```
SharedPreferences sp =getSharedPreferences("status",Activity.MODE_PRIVATE);
sp.getInt("isLogin", false);直接get就可以得到相应数据 没有该字段 取默认值
SharedPreferences.Editor editor = sp.edit();//使用editor进行储存
editor.putString("param1", "eva");
editor.putInt("param2", 250);
editor.commit();//2.3之后可以使用editor.apply(); 异步操作
```
## 众多的drawable文件夹都有什么不同?

* 对分辨率进行区分 系统会自动使用分辨率下的资源
也可使用一些范围来标识尺寸 

* 如果对尺寸的要求不高 一般都将drawable文件夹下存放一些效果文件 使用单独一个文件夹来放置图片文件

* 但是还是建议开发程序兼容不同平台不同屏幕 将各自文件夹根据需求均存放不同版本图片 

## 数据库升级需要注意什么?
请参考 拍店APP 旧版升级后 SQLite出错分析
[Android版本更新时对SQLite数据库升级或者降级遇到的问题](http://blog.csdn.net/jie1991liu/article/details/50339797)
* 1.数据库的升级是通过修改version值实现的。 
* 2.新用户会通过onCreate()方法产生数据库。对于升级用户而言，如果没有升级数据库，则不会进行数据库创建及升级操作。如果升级了数据库，则会调用onUpgrade( )方法。 
* 3.应用的升级与数据库升级无必然的联系。
* 4.如果没找到数据库，会调用`onCreate`里的方法，如果找到了数据而且版本较低，会调用`onUpgrade`里的方法。像你这样的情况，新表要在`onCreate`和`onUpgrade`都写上创建语句。
* 5.对于可能导致操作异常的sql语句增加try、catch，但是不能对所有sql语句增加try、catch，因为有的数据必须要执行才能保证正常功能。比如新建数据库等。 

## 在gradle的配置文件中 versionCode versionName minSdkVersion targetSdkVersion 各代表什么含义?

* versionCode 版本号 是一个递增的数字 用户不可见

* versionName 版本名 可以任意自定义 用户可见 

* minSdkVersion 在安装程序的时候 如果目标设备的API版本小于
minSdkVersion 或者大于maxSdkVersion 程序将无法安装 一般来说没有必要设置maxSdkVersion

* targetSdkVersion  指的是应用最适的版本 调用接口时只会调用该版本实现的API 而不是早期版本的API

## 如何在android中优雅的使用SQLite?

* a.ContentProvider+LoaderManager

首先Loader提供了一套在UI的主线程中异步加载数据的框架 可以非常简单的在Activity或者Fragment中异步加载数据 配合LoaderManager 是目前在Activity&Fragment中异步读取ContentProvider的非常不错的方案

ContentProvider着实让人很头疼 实际上Android文档中提到 如果没有跨进程的需求 或者向其他应用分享数据的需求就不必使用ContentProvider 但ContentProvider为数据库的管理提供了更清晰的接口 并且为了使用CursorLoader ContentProvider是必须构建的

此方案的使用场景：一般适用于大量的数据查询 或者需要经常修改并及时展示的数据显示到UI上 同时可以避免查询数据的时候 造成UI主线程的卡顿
缺点是此方案虽好 但是使用起来稍微麻烦 对于一般的简单数据处理有点大才小用

* b.ORM

除此之外就是使用ORM框架了 使用ORM框架的目的是使代码看起来更优雅 可读性更好 使用起来更方便 即使对于一些编程新手来说 可能你没有那么熟悉SQL 但是你一样熟悉了这套ORM框架依然可以进行开发工作 对于有哪些流行的ORM框架 比如GreenDao 每个框架各有优劣 但其实也没多大区别 自己用着顺手就好
ORM适应于大部分场景 但是你应该知道它并没有那么神奇 其实底层还是通过拼接SQL来执行的 中间多了一层性能上当然没有直接执行SQL好了 不过这点性能可以忽略 几乎对程序没什么影响 正常使用完全ok 但是复杂点的或者一旦出问题你不熟悉SQL会比较困难

* c.SQL

其次就是使用Android自带的一套框架来写SQL了 但是缺点显而易见 代码可读性差 出错概率更大 自己处理各种增产改查与升级 但是我觉得这应该是一个Android开发必经的一个过程 你只有真正经历过坑 才能真正体会到为什么ORM比较好 从中也可以学习熟悉下基本的SQL操作 

从未经历过SQLite的开发者 学习应该是从下往上的

所以介绍一下最基本的数据库操作

SQLite是一个轻量级的数据库 虽然轻量 但是足够开发者使用的 而且提供的API支持帮助开发者使用函数拼出SQL 

SQLiteOpenHelper  : 维护和管理数据库的基类 两个函数需要重载 onUpgrade() onCreate() 

onUpgrade() 主要是升级数据库时进行的操作 具体可参考10中的wiki

onCreate() 数据库第一次被创建时会执行 (升级数据库版本 这种操作是不执行的 切记)

SQLiteDatabase : 数据库基类 可使用以下方法执行SQL 

此外可以直接使用insert等语句操作
```
SQLiteDatabase db = openOrCreateDatabase("test.db", Context.MODE_PRIVATE, null);(表创建后可使用openHelper.getReadableDatabase();得到数据库)
db.execSQL("DROP TABLE IF EXISTS person"); 
db.execSQL("CREATE TABLE person (_id INTEGER PRIMARY KEY AUTOINCREMENT, name VARCHAR, age SMALLINT)"); 
Person person = new Person(); 
person.name = "john"; 
person.age = 30; 
db.execSQL("INSERT INTO person VALUES (NULL, ?, ?)", new Object[]{person.name, person.age}); 
```
数据库事务
```
public void add(List<Person> persons) { 
        db.beginTransaction();  //开始事务 
        try { 
            for (Person person : persons) { 
                db.execSQL("INSERT INTO person VALUES(null, ?, ?, ?)", new Object[]{person.name, person.age, person.info}); 
            } 
            db.setTransactionSuccessful();  //设置事务成功完成 
        } finally { 
            db.endTransaction();    //结束事务 
        } 
    }
```
## 如何正确使用LoaderManager ?
loaderManager 使用规范

[LoaderManager使用详解](https://segmentfault.com/a/1190000007132972)

LoaderManager用来负责管理与Activity或者Fragment联系起来的一个或多个Loaders对象。每个Activity或者Fragment都有唯一的一个LoaderManager实例，用来启动、停止、保持、重启、关闭它的Loaders。这些事件有时直接在客户端通过调用initLoader()/restartLoader()/destroyLoader()函数来实现。通常这些事件通过主要的Activity/Fragment声明周期事件来触发，而不是手动（当然也可以手动调用）。比如，当一个Activity关闭时（destroyed），改活动将指示它的LoaderManager来销毁并且关闭它的Loaders（当然也会销毁并关闭与这些Loaders关联的资源，比如Cursor）。

LoaderManager并不知道数据如何装载以及何时需要装载。相反地，LoaderManager只需要控制它的Loaders们开始、停止、重置他们的Load行为，在配置变换（比如横竖屏切换）时保持loaders们的状态，并提供一个简单的接口来获取load结果到客户端中。

## ImageView 为了适应各种显示场景 如何变换图片的显示方式?
1. 在layout xml中定义android:scaleType="XX"

2. 或在代码中调用imageView.setScaleType(ImageView.ScaleType.XX)

ImageView.ScaleType.CENTER  按原图大小居中显示  如果超过屏幕 截取图片居中的部分显示（显示不全）

ImageView.ScaleType.CENTER_CROP 按比例扩大使图片居中显示  使得图片的长或宽大于等于view的长或宽（显示不全）

ImageView.ScaleType.CENTER_INSIDE 按比例缩小使图片居中显示  使图片的长宽小于等于view的长宽 不缩小的按原大小显示（完全显示 ）

ImageView.ScaleType.FIT_CENTER 图片按比例缩放或扩大到view的宽度

ImageView.ScaleType.FIT_XY  不按比例  让图片充满整个view

## 是否使用过Fragment 在Fragment中是否出现过getActivity() 为空的问题 什么原因 如何解决?

* 第一种情况 当应用切换到后台 内存不足 activity被回收时 fragment并未回收 而是被暂存了 FragmentManager将其保存为bundle 下面为FragmentActivity源码
```
/**
 * Save all appropriate fragment state.
 */
@Override
protected void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    Parcelable p = mFragments.saveAllState();
    if (p != null) {
        outState.putParcelable(FRAGMENTS_TAG, p);
    }
}
```
所以 当再次回到应用时  activity为空  但是fragment 并不是空

* 第二种情况 在fragment中发送网络请求 当请求发出去 立刻退出activity 此时fragment并未回收 请求回来处理数据 当使用到getActivity 会得到空

**解决方法**:  最优方法为在使用getActivity的时候判空  如果传递进fragment一个activity的引用 也可以解决

16.是否使用过GSON解析Json数据 如何解析服务器返回的下列数据？
 
```
{
    "versioninfo": {
        "isUpdated": false,
        "currentVersion": 175,
        "changeLog": "4",
        "versionname": "4.7.5",
        "appurl": "/appupdate/download/group-175_0-meituan.apk?channel=meituan",
        "forceupdate": 0
    }
}
```
Java Bean要写成UpdateInfo{ VersionInfo versionInfo;} 即多嵌套一层  GSON才会识别
```
Gson gson = new Gson();
UpdateInfo info = gson.fromJson(response, UpdateInfo.class);
```
## android的官方建议应用程序的开发采用mvc模式。何谓mvc?

mvc是model view controller 包含三个部分
模型（model）对象：是应用程序的主体部分 所有的业务逻辑都应该写在该层
视图（view）对象：是应用程序中负责生成用户界面的部分 也是在整个mvc架构中用户唯一可以看到的一层 接收用户的输入 显示处理结果
控制器（control）对象：是根据用户的输入 控制用户界面数据显示及更新model对象状态的部分 控制器更重要的一种导航功能 想用用户出发的相关事件 交给model处理。

android鼓励弱耦合和组件的重用 在android中mvc的具体体现如下：

1)视图层（view）：一般采用xml文件进行界面的描述，使用的时候可以非常方便的引入，当然，如何你对android了解的比较的多了话，就一定 可以想到在android中也可以使用javascript+html等的方式作为view层，当然这里需要进行java和javascript之间的通 信，幸运的是，android提供了它们之间非常方便的通信实现。

2)控制层（controller）：android的控制层的重 任通常落在了众多的acitvity的肩上，这句话也就暗含了不要在acitivity中写代码，要通过activity交割model业务逻辑层处理， 这样做的另外一个原因是android中的acitivity的响应时间是5s，如果耗时的操作放在这里，程序就很容易被回收掉。

3)模型层（model）：对数据库的操作、对网络等的操作都应该在model里面处理，当然对业务计算等操作也是必须放在的该层的。

## android中得ANR是什么  如何避免?

ANR即Application Not Responding 程序无响应 

由系统的Activity Manager和Window Manager系统服务监视

1.在5秒内没有响应输入的事件（例如 按键按下 屏幕触摸）

2.BroadcastReceiver在10秒内没有执行完毕

这两种情况下会提示ANR 提示用户关闭应用或等待

如何避免？

在UI线程不做耗时操作  使用loader AsyncTask handler 