App启动优化
===

## 应用的启动方式

通常来说，启动方式分为两种：冷启动和热启动。

1、冷启动：当启动应用时，后台没有该应用的进程，这时系统会重新创建一个新的进程分配给该应用，这个启动方式就是冷启动。

2、热启动：当启动应用时，后台已有该应用的进程（例：按back键、home键，应用虽然会退出，但是该应用的进程是依然会保留在后台，可进入任务列表查看），所以在已有进程的情况下，这种启动会从已有的进程中来启动应用，这个方式叫热启动。

## App的启动过程

## 本文所指的优化针对冷启动。简单解释一下App的启动过程：

*   1.点击Launcher，启动程序,通知ActivityManagerService

*   2.ActivityManagerService通知zygote进程孵化出应用进程，分配内存空间等

*   3.执行该应用ActivityThread的main()方法

*   4.应用程序通知ActivityManagerService它已经启动，ActivityManagerService保存一个该应用的代理对象,ActivityManagerService通过它可以控制应用进程

*   5.ActivityManagerService通知应用进程创建入口的Activity实例，执行它的生命周期

启动过程中Application和入口Activity的生命周期方法按如下顺序调用：

*   1.Application 构造方法

*   2.attachBaseContext()

*   3.onCreate()

*   4.入口Activity的对象构造

*   5.setTheme() 设置主题等信息

*   6.入口Activity的onCreate()

*   7.入口Activity的onStart()

*   8.入口Activity的onResume()

*   9.入口Activity的onAttachToWindow()

*   10.入口Activity的onWindowFocusChanged()

## 启动时间统计

* [统计启动时长-标准](https://pxfile.github.io/2018/03/04/android-%E7%BB%9F%E8%AE%A1%E5%90%AF%E5%8A%A8%E6%97%B6%E9%95%BF-%E6%A0%87%E5%87%86/)

## 启动页优化

### Static Block

很多代码中的Static Block，都是做一些初始化工作，特别是ContentProvider中在Static Block中初始化一些UriMatcher，这些东西可以做成懒加载模式。

### Application

Application是程序的主入口，特别是很多第三方SDK都会需要在Application的onCreate里面做很多初始化操作，不得不说，各种第三方SDK，都特别喜欢这个『兵家必争之地』，再加上自己的一些库的初始化，会让整个Application不堪重负。

优化的方法，无非是通过以下几个方面：

*   延迟初始化
*   后台任务
*   界面预加载

### 阻塞

阻塞有很多种情况，例如磁盘IO阻塞（读写文件、SharedPerfences）、网络阻塞（现在应该不会了）以及高CPU占用的代码（加解密、渲染、解析等等）。

### 耗时方法

通过使用TraceView && Systrace && Method Tracing工具来进行排查，见《Android群英传:神兵利器》

## App启动优化的一般过程

1.  通过TraceView、Systrace来分析耗时的方法与组件。
2.  梳理启动加载的每一个库、组件。
3.  将梳理出来的库，按功能和需求进行划分，设计该库的启动时机。
4.  与交互沟通，设计启动画面，按前文方法进行优化。

## 解决方案

### Theme

当系统加载一个Activity的时候，onCreate()是一个耗时过程，那么在这个过程中，系统为了让用户能有一个比较好的体验，实际上会先绘制一些初始界面，类似于PlaceHolder。

系统首先会读取当前Activity的Theme，然后根据Theme中的配置来绘制，当Activity加载完毕后，才会替换为真正的界面。所以，Google官方提供的解决方案，**就是通过android:windowBackground属性，来进行加载前的配置，同时，这里不仅可以配置颜色，还能配置图片**，例如，我们可以使用一个layer-list来作为android:windowBackground要显示的图：

start_window.xml

```
<layer-list xmlns:android="http://schemas.android.com/apk/res/android"
            android:opacity="opaque">
    <item android:drawable="@android:color/darker_gray"/>
    <item>
        <bitmap
            android:gravity="center"
            android:src="@mipmap/ic_launcher"/>
    </item>
</layer-list>
```

可以看见，这里通过layer-list来实现图片的叠加，让开发者可以自由组合。

> 配置中的android:opacity=”opaque”参数是为了防止在启动的时候出现背景的闪烁。

接下来可以设置一个新的Style，这个Style就是Activity预加载的Style。

```
<resources>

    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

    <style name="StartStyle" parent="AppTheme">
        <item name="android:windowBackground">@drawable/start_window</item>
    </style>
</resources>
```

OK，下面在Mainifest中给Activity指定需要预加载的Style：

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.xys.startperformancedemo">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity
            android:name=".MainActivity"
            android:theme="@style/StartStyle">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>

                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>

</manifest>
```

这里需要注意下，一定是Activity的Theme，而不是Application的Theme。

最后，我们在Activity加载真正的界面之前，将Theme设置回正常的Theme就好了：

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        setTheme(R.style.AppTheme);
        super.onCreate(savedInstanceState);
        SystemClock.sleep(2000);
        setContentView(R.layout.activity_main);
    }
}

```

在这个Activity中，我使用SystemClock.sleep(2000)，模拟了一个Activity加载的耗时过程，在super.onCreate(savedInstanceState)调用前，将主题重新设置为原来的主题。

通过这种方式设置的效果如下：

![这里写图片描述](http://img.blog.csdn.net/20161105135857103)

启动的时候，会先展示一个画面，这个画面就是系统解析到的Style，等Activity加载完全完毕后，才会加载Activity的界面，而在Activity的界面中，我们将主题重新设置为正常的主题，从而达到一个友好的启动体验，这种方式其实并没有真正的加速启动过程，而是通过交互体验来优化了展示的效果。

### 异步初始化

这个很简单，就是让App在onCreate里面尽可能的少做事情，而利用手机的多核特性，尽可能的利用多线程，例如一些第三方框架的初始化，如果能放线程，就尽量的放入线程中，最简单的，你可以直接new Thread()，当然，你也可以通过公共的线程池来进行异步的初始化工作，这个是最能够压缩启动时间的方式

### 延迟初始化

延迟初始化并不是减少了启动时间，而是让耗时操作让位、让资源给UI绘制，将耗时的操作延迟到UI加载完毕后，所以，这里建议通过mDecoView.post方法，来进行延迟加载，代码如下：

```
getWindow().getDecorView().post(new Runnable() {

  @Override public void run() {
    ……
  }
});
```

我们的ContentView就是通过mDecoView.addView加入到根布局的，所以，通过这种方式，可以让延迟加载的内容，在ContentView初始化完毕后，再进行执行，保证了UI绘制的流畅性。

### IntentService

IntentService是继承于Service并处理异步请求的一个类，在IntentService的内部，有一个工作线程来处理耗时操作，启动IntentService的方式和启动传统Service一样，同时，当任务执行完后，IntentService会自动停止，而不需要去手动控制。

```
public class InitIntentService extends IntentService {

    private static final String ACTION = "com.xys.startperformancedemo.action";

    public InitIntentService() {
        super("InitIntentService");
    }

    public static void start(Context context) {
        Intent intent = new Intent(context, InitIntentService.class);
        intent.setAction(ACTION);
        context.startService(intent);
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        SystemClock.sleep(2000);
        Log.d(TAG, "onHandleIntent: ");
    }
}
```

我们将耗时任务丢到IntentService中去处理，系统会自动开启线程去处理，同时，在任务结束后，还能自己结束Service，多么的人性化！OK，只需要在Application或者Activity的onCreate中去启动这个IntentService即可：

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    InitIntentService.start(this);
}
```

最后不要忘记在Mainifest注册Service。

### 使用ActivityLifecycleCallbacks

Framework提供的这个方法可以监控到所有Activity的生命周期，在这里，我们就可以通过onActivityCreated这样一个回调，来将一些UI相关的初始化操作放到这里，同时，通过unregisterActivityLifecycleCallbacks来避免重复的初始化。同时，这里onActivityCreated回调的参数Bundle，可以用来区别是否是被系统所回收的Activity。

```
public class MainApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        // 初始化基本内容
        // ……
        registerActivityLifecycleCallbacks(new ActivityLifecycleCallbacks() {
            @Override
            public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
                unregisterActivityLifecycleCallbacks(this);
                // 初始化UI相关的内容
                // ……
            }

            @Override
            public void onActivityStarted(Activity activity) {
            }

            @Override
            public void onActivityResumed(Activity activity) {
            }

            @Override
            public void onActivityPaused(Activity activity) {
            }

            @Override
            public void onActivityStopped(Activity activity) {
            }

            @Override
            public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
            }

            @Override
            public void onActivityDestroyed(Activity activity) {
            }
        });
    }
}
```

### 资源优化

有几个方面，一个自然是优化布局、布局层级，一个是优化资源，尽可能的精简资源、避免垃圾资源，这些可以通过混淆和tinyPNG这些工具来实现。

## 定位问题
可以通过**Method Tracing**或者**DDMS**来获得更全面详细的信息。 启动应用，点击 Start Method Tracing，应用启动后再次点击，会自动打开刚才操作所记录下的.trace文件，建议使用DDMS来查看，功能更加方便全面。

通过对traceview的详细跟踪以及代码的详细比对，我发现**卡顿发生在**：

*   **部分数据库及IO的操作发生在首屏Activity主线程；**
*   **Application中创建了线程池；**
*   **首屏Activity网络请求密集；**
*   **工作线程使用未设置优先级；**
*   **信息未缓存，重复获取同样信息；**
*   **流程问题：例如闪屏图每次下载，当次使用；**

以及其它细节问题：

*   执行无用老代码；
*   执行开发阶段使用的代码；
*   执行重复逻辑；
*   调用三方SDK里或者Demo里的多余代码；

**项目修改：** **1\. 数据库及IO操作都移到工作线程，并且设置线程优先级为THREAD_PRIORITY_BACKGROUND，这样工作线程最多能获取到10%的时间片，优先保证主线程执行。**

**2\. 流程梳理，延后执行； 实际上，这一步对项目启动加速最有效果。通过流程梳理发现部分流程调用时机偏早、失误等，例如：**

*   **更新等操作无需在首屏尚未展示就调用，造成资源竞争；**
*   **自有统计在Application的调用里创建数量固定为5的线程池，造成资源竞争，在上图traceview功能说明图中最后一行可以看到编号12执行5次，耗时排名前列；此处线程池的创建是必要但可以延后的。**
*   **修改广告闪屏逻辑为下次生效。**

**3.其它优化；**

*   **去掉无用但被执行的老代码；**
*   **去掉开发阶段使用但线上被执行的代码；**
*   **去掉重复逻辑执行代码；**
*   **去掉调用三方SDK里或者Demo里的多余代码；**
*   **信息缓存，常用信息只在第一次获取，之后从缓存中取；**
*   **项目是多进程架构，只在主进程执行Application的onCreate()；**

**通过ADB命令统计应用的启动时间：adb shell am start -W 首屏Activity**

### 七、问题：

1、还可以继续优化的方向？

*   **项目里使用Retrofit网络请求库，FastConverterFactory做Json解析器，TraceView中看到FastConverterFactory在创建过程中也比较耗时，考虑将其换为GsonConverterFactory。但是因为类的继承关系短时间内无法直接替换，作为优化点暂时遗留；**
*   **可以考虑根据实际情况将启动时部分接口合并为一，减少网络请求次数，降低频率；**
*   **相同功能的组件只保留一个，例如：友盟、GrowingIO、自有统计等功能重复；**
*   **使用[ReDex](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Ffacebook%2Fredex)进行优化；实验Redex发现Apk体积确实是小了一点，但是启动速度没有变化，或许需要继续研究。**

2、异步、延迟初始化及操作的依据？ **注意一点：并不是每一个组件的初始化以及操作都可以异步或延迟；是否可以取决组件的调用关系以及自己项目具体业务的需要。保证一个准则：可以异步的都异步，不可以异步的尽量延迟。让应用先启动，再操作。**

3、通用应用启动加速套路？

*   **利用主题快速显示界面**
*   **异步初始化组件**
*   **梳理业务逻辑，延迟初始化组件、操作**
*   **正确使用线程**
*   **去掉无用代码、重复逻辑等**

4、其它

*   **将启动速度加快了35%不代表之前的代码都是问题，从业务角度上将，代码并没有错误，实现了业务需求。但是在启动时这个注重速度的阶段，忽略的细节就会导致性能的瓶颈。**
*   **开发过程中，对核心模块与应用阶段如启动时，使用TraceView进行分析，尽早发现瓶颈。**


## 甩锅方案

下面是两种不同的方案，都是在Style中进行配置：

```
<item name="android:windowDisablePreview">true</item>
```

与

```
<item name="android:windowIsTranslucent">true</item>
<item name="android:windowNoTitle">true</item>
```
设置效果类似，即通过取消、透明化系统的统一的加载页面来达到启动的『加速』，实际上，是一个『甩锅』的过程。强烈建议开发者不要通过这种方式去做『所谓的启动加速』,这种方式虽然看上去自己的App启动非常快，瞬间就完成了，但实际上，是将真正的启动界面给隐藏了。

> 系统说：这锅，我们不背！

## 无解

对应5.0以下的65535问题，目前只能通过Multidex来进行处理，而在5.0以下的机器上，系统在加载前的合并Dex的过程，有可能非常长，这也是暂时无解的问题，只能希望后面Multidex进行优化。
Multidex的使用，是拖慢启动速度的元凶，必须要做优化。

OK，App的启动优化基本如上，其重点过程，依然是分析耗时的操作，以及如何设计合理的启动顺序

## 参考
[一触即发 App启动优化最佳实践](http://blog.csdn.net/eclipsexys/article/details/53044990)