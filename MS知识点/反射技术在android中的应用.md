反射技术在android中的应用
===
[java/android中的反射机制](https://www.2cto.com/kf/201709/676771.html)

## 什么是反射机制

个人理解就是通过反编译获取类中所有的信息(包括：变量、方法、接口)，供开发者利用。

### 优缺点

* 优点：增强代码的自适应能力(动态的创建对象)、调用一些类中的私有方法(例如通过反射机制调用android[系统](https://www.2cto.com/os/)挂断电话的方法)。

* 缺点：降低程序性能。牛逼的背后总是苦逼，反射机制说白了就是通过类名去解释类，然后告诉jvm我们需要做什么，肯定比jvm执行固定代码要耗时耗资源。

## **动态语言：**

一般认为在程序运行时，允许改变程序结构或变量类型，这种语言称为动态语言。从这个观点看，Perl，Python，Ruby是动态语言，C++，Java，C#不是动态语言。尽管这样，JAVA有着一个非常突出的动态相关机制：反射（Reflection）。运用反射我们可以于运行时加载、探知、使用编译期间完全未知的classes。换句话说，Java程序可以加载在运行时才得知名称的class，获悉其完整构造方法，并生成其对象实体、或对其属性设值、或唤起其成员方法。

## **反射：**

要让Java程序能够运行，就得让Java类被Java虚拟机加载。Java类如果不被Java虚拟机加载就不能正常运行。正常情况下，我们运行的所有的程序在编译期时候就已经把那个类被加载了。 Java的反射机制是在编译时并不确定是哪个类被加载了，而是在程序运行的时候才加载。使用的是在编译期并不知道的类。这样的编译特点就是java反射。

## **反射的作用:**

如果有AB两个程序员合作，A在写程序的时需要使用B所写的类，但B并没完成他所写的类。那么A的代码是不能通过编译的。此时，利用Java反射的机制，就可以让A在没有得到B所写的类的时候，来使自身的代码通过编译。

## **反射的实质:**

反射就是把Java类中的各种存在给解析成相应的Java类。要正确使用Java反射机制就得使用Class（C大写） 这个类。它是Java反射机制的起源。当一个类被加载以后，Java虚拟机就会自动产生一个Class对象。通过这个Class对象我们就能获得加载到虚拟机当中这个Class对象对应的方法、成员以及构造方法的声明和定义等信息。

## **反射机制的优点与缺点:**

为什么要用反射机制？直接创建对象不就可以了吗，这就涉及到了动态与静态的概念: 
* 静态编译：在编译时确定类型，绑定对象,即通过。 
* 动态编译：运行时确定类型，绑定对象。动态编译最大限度发挥了java的灵活性，体现了多态的应用，降低类之间的藕合性。 

### 优点
一句话，反射机制的优点就是可以实现动态创建对象和编译，体现出很大的灵活性，特别是在J2EE的开发中它的灵活性就表现的十分明显。比如，一个大型的软件，不可能一次就把把它设计的很完美，当这个程序编译后，发布了，当发现需要更新某些功能时，我们不可能要用户把以前的卸载，再重新安装新的版本，假如这样的话，这个软件肯定是没有多少人用的。采用静态的话，需要把整个程序重新编译一次才可以实现功能的更新，而采用反射机制的话，它就可以不用安装，只需要在运行时才动态的创建和编译，就可以实现该功能。

### 缺点
它的缺点是对性能有影响。使用反射基本上是一种解释操作，我们可以告诉JVM，我们希望做什么并且它满足我们的要求。这类操作总是慢于只直接执行相同的操作。

* * *

## **Android FrameWork中的反射:**

一个类中的每个成员都可以用相应的反射API的一个实例对象来表示——反射机制。 
了解这些，那我们就知道了，我们可以利用反射机制在Java程序中，动态的去调用一些protected甚至是private的方法或类，这样可以很大程度上满足我们的一些比较特殊需求。例如Activity的启动过程中Activity的对象的创建。

以下代码位于ActivityThread中：

```
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {

        。。。。。。
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            StrictMode.incrementExpectedActivityCount(activity.getClass());
            r.intent.setExtrasClassLoader(cl);
            r.intent.prepareToEnterProcess();
            if (r.state != null) {
                r.state.setClassLoader(cl);
            }
        } 
        。。。。。。
```

上面代码可知Activity在创建对象的时候调用了mInstrumentation.newActivity(); 
以下代码位于Instrumentation中：

```
    public Activity newActivity(ClassLoader cl, String className,
            Intent intent)
            throws InstantiationException, IllegalAccessException,
            ClassNotFoundException {
            //这里的className就是在manifest中注册的Activity name.
        return (Activity)cl.loadClass(className).newInstance();
    }
```

最终在newActivity()里返回的是利用cl.loadClass返回的Activity对象。可知，Activity对象的创建是通过反射完成的。java程序可以动态加载类定义，而这个动态加载的机制就是通过ClassLoader来实现的，所以可想而知ClassLoader的重要性如何。 

## **ClassLoader和DexClassLoader**

上面说到JAVA的动态加载的机制就是通过ClassLoader来实现的，ClassLoader也是实现反射的基石。ClassLoader是JAVA提供的一个类，顾名思义，它就是用来加载Class文件到JVM，以供程序使用的。

但是问题来了，ClassLoader加载文件到JVM，但是Android是基于DVM的，用ClassLoader加载文件进DVM肯定是不行的。于是Android提供了另外一套加载机制，分别为 dalvik.system.DexClassLoader 和 dalvik.system.PathClassLoader，区别在于 PathClassLoader 不能直接从 zip 包中得到 dex，因此只支持直接操作 dex 文件或者已经安装过的 apk（因为安装过的 apk 在 cache 中存在缓存的 dex 文件）。而 DexClassLoader 可以加载外部的 apk、jar 或 dex文件，并且会在指定的 outpath 路径存放其 dex 文件。

## **ClassLoader在JAVA中的应用**

下面利用反射来调用另一个类中的方法

```
//定义一个测试类，用来被反射调用
package com.izzy;
public class Test {
    private String s;
    //构造方法
    public Test(String s) {
        this.s = s;
    }
    //定义一个方法，用来输出，构造方法中传递进来的参数
    public void display() {
        System.out.println(s);
    }

}

```

```
package com.izzy;
import java.lang.reflect.Constructor;

public class Client {

    public static void main(String[] s) {
        try {
        //首先拿到系统ClassLoader，并加载Class，返回的是一个Class对象clazz 
            Class clazz = ClassLoader.getSystemClassLoader().loadClass(
                    "com.izzy.Test");
        //通过clazz 拿到构造方法并转换成对象
            Constructor constructor = clazz.getConstructor(String.class);
            Object obj = constructor.newInstance("I AM IZZY");
        //通过clazz 拿到成员方法
            Method method = clazz.getMethod("display", null);
            method.invoke(obj, null);
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

}

```

![这里写图片描述](http://img.blog.csdn.net/20160617160312603)

## **DexClassLoader在Android中的应用**

以一个例子来说明DexClassLoader用法（本例采用两个已安装的Apk），现在有两个Apk：Share和Test，利用Test来调用Share 里面的方法。

首先在Share　apk中定义Share类，其中有一个display()方法提供给远程调用

```
public class Share {
    public void display(String s) {
        Log.e("IZZY", s);
    }

}
```

接着在manifest文件中配置action和category，方便调用这找到

```
  <activity
            android:name=".MainActivity"
            android:theme="@android:style/Theme.Light.NoTitleBar">
            <intent-filter>
                <action android:name="com.IZZY"/>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
```

接下来就是在Test Apk中编写调用代码了

```
 public void getFromRemote() {
        Intent intent = new Intent("com.IZZY");
        PackageManager pm = getPackageManager();
        List<ResolveInfo> resolveInfos = pm.queryIntentActivities(intent, 0);

        ResolveInfo resolveInfo = resolveInfos.get(0);
        ActivityInfo activityInfo = resolveInfo.activityInfo;
        //拿到目标类的包名
        String packageName = activityInfo.packageName;

        //拿到目标类所在的apk或者jar存放的路径
        String dexPath = activityInfo.applicationInfo.sourceDir;
        //该路径为拿到目标类dex文件存放在调用者里的路径
        String dexOutputDir = getApplicationInfo().dataDir;
        //拿到目标类所使用的C/C++库存放路径
        String nativeLibraryDir = activityInfo.applicationInfo.nativeLibraryDir;
        //拿到类装载器
        ClassLoader classLoader = getClassLoader();

        //DexClassLoader参数分别对应以上四个参数
        DexClassLoader dcl = new DexClassLoader(dexPath,dexOutputDir,nativeLibraryDir,classLoader);
        try {
            //装载目标类
            Class<?> clazz = dcl.loadClass(packageName + ".Share");
            //拿到构造器并实例化对象
            Constructor<?> constructor = clazz.getConstructor();
            Object o = constructor.newInstance();
            //拿到成员方法
            Method display = clazz.getMethod("display", String.class);
            display.invoke(o, "I AM IZZY");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

在该调用方法中首先利用Intent查询到目标activityInfo，然后利用查询到的activityInfo得到目标Apk的包名，目标Apk所在的apk或者jar存放的路径dexPath，目标Apk所使用的C/C++库存放路径nativeLibraryDir。然后利用这些参数实例化DexClassLoader加载器。之后反射调用目标类中的方法。
（ps：此处使用的目标Apk是已经安装过的，因此采用Intent查询来拿到dexPath和nativeLibraryDir，如果是未安装过的jar包或Apk，则直接传入该jar包活Apk的文件存放路径和C/C++库存放路径）。 

运行结果： 
![这里写图片描述](http://img.blog.csdn.net/20160630133339862) 
从结果看，调用者Apk拿到了目标Apk的方法并成功执行。

## **DexClassLoaderde 在Android中的使用场景**

上面是是使用的已经安装过的Apk，如果采用未安装过的jar包或者Apk，则实例化DexClassLoader的时候把相应路径改为需要加载的jar包或者Apk路径亦可拿到结果。这就使得DexClassLoaderde可以应用在HotFix（热修复），动态加载框架等等 一些基于插件化的架构中。