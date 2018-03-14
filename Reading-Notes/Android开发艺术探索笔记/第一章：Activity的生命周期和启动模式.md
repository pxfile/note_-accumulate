# 第一章：Activity的生命周期和启动模式
---
>怀着无比崇敬的心情翻开了这本书，路漫漫其修远兮，程序人生，为自己加油！

## 一.序
>作为这本书的第一章，主席还是把Activity搬上来了，也确实，和Activity打交道的次数基本上是最多的，而且他的内容和知识点也是很多的，非常值得我们优先把他掌握，Activity中文翻译过来就是"活动"的意思，但是主席觉得这样翻译有些生硬，直接翻译成“界面”可能更好，的确，Activity主要也是用于UI效果的呈现，不过两者翻译都不为过，我们知其意就行了，正常情况下，我们除了Window,Dialog,Toast，我们还能见到的就只有Activity了，他需要setContentView()去绑定一个视图View作为效果的呈现，然而，作为一本高质量的进阶书。肯定不会去围绕着入门知识讲解，本章的侧重点在于对Activity使用过程中搞不清楚的概念，生命周期和启动模式已经IntentFilter的匹配规则分析，毕竟Activity在异常状态下的生命周期是多样化的，至于Activity的启动模式和各种各样的Flags,更是让很多人摸不着头脑，还有隐式启动Activity中也有着复杂的Intent匹配过程，所以我们还是一步步的去学习下去，真正的了解Activity这个小家伙！

## 二.Activity的生命周期全面分析
>Activity的生命周期，本章主要讲解两个方面

- 典型情况下的生命周期
- 异常情况下的生命周期

>典型情况是指用户参与的情况下，Activity所经过的生命周期的变化，异常情况下的话，就有多种可能了，比如系统回收或者由于当前设备的Configuration发生改变从而导致Activity被销毁重建，异常情况下的生命周期的关注点和典型情况下有些不同，所以要分开来描述才能描述的清楚些

### 1.典型情况下的生命周期分析
>在正常的情况下，生命周期会经历以下的生命周期

- onCreate：表示Activity正在被创建，这是生命周期的第一个方法，在这个方法中，我们可以做一些初始化的工作，比如调用onContentView去加载界面布局资源，初始化Activity所需数据等

- onRestart:表示Activity正在重新启动，一般情况下，当当前Activity从不可见重新变为可见时，onRestart就会被调用，这总情况一般是用户行为所导致的，比如用户按home键切换到桌面或者用户打开了一个新的Activity，这时当前的Activity就会被暂停，也就是onPause和onStop方法被执行了，接着用户又回到了这个Activity，就会出现这种情况

- onStart:表示Activity正在被启动，即将开始，这个时候Activity已经可见了，但是还没有出现在前台，还无法和用户交互，这个时候我们可以理解为Activity已经启动了，但是我们还没有看见

- onResume:表示Activity已经可见了，并且出现在前台，并开始活动了，要注意这个和onStart的对比，这两个都表示Activity已经可见了，但是onStart的时候Activity还处于后台，onResume的时候Activity才显示到前台

- onPause:表示Activity正在停止，正常情况下，紧接着onStop就会被调用，在特殊情况下，如果这个时候再快速的回到当前Activity，那么onResume就会被调用，主席的理解是这个情况比较极端，用户操作很难重现这个场景，此时可以做一些数据存储，停止动画等工作，但是注意不要太耗时了，因为这样会影响到新的Activity的显示，onPause必须先执行完，新Activity的onResume才会执行

- onStop:表示Activity即将停止，同样可以做一些轻量级的资源回收，但是不要太耗时了

- onDestroy:表示Activity即将被销毁，这是Activity生命周期的最后一个回调，在这里我们可以做一些最后的回收工作和资源释放

>正常情况下，Activity的常用生命周期用官网的一张图就足够表示了

![这里写图片描述](http://img.blog.csdn.net/20160915144447393)

>这里附加几个说明

- 1.针对一个特定的Activity，第一次启动，回调如下：onCreate ——> onStart ——> onResume

- 2.当用户打开新的Activity或者切换到桌面的时候，回调如下：onPause ——> onStop ——> 这里有一种特殊的情况就是，如果新的Activity采取了透明的主题的话，那么当前Activity不会回调onStop

- 3.当用户再次回到原来的Activity，回调如下：onRestart ——> onStart ——> onResume

- 4.当用户按back键的时候回调如下：onPause ———> onStop ——> onDestroy

- 5.当Activity被系统回收的时候再次打开，生命周期回调方法和1是一样的，但是你要注意一下就是只是生命周期一样，不代表所有的进程都是一样的，这个问题等下回详细分析

- 6.从整个生命周期来说，onCreate和onDestroy是配套的,分别标示着Activity的创建和销毁，并且只可能有一次调用，从Activity是否可见来说，onStart和onStop是配套的，随着用户的操作和设备屏幕的点亮和熄灭，这两个方法可能被调用多次，从Activity是否在前台来说，onResume和onPause是配套的，随着用户操作或者设备的点亮和熄灭，这两个方法可能被多次调用


>这里提出两个问题

- 1.onStart和onResume，onPause和onStop从描述上都差不多，对我们来说有什么实质性的不同呢？
- 2.假设当前Activity为A，如果用户打开了一个新的Activity为B，那么B的onResume和A的onPause谁先执行尼？

>我们先来回答第一个问题，从实际使用过程来说， onStart和onResume，onPause和onStop看起来的确差不多，甚至我们可以只保留其中的一对，比如只保留onStart和onStop，既然如此，那为什么Android系统还会提供看起来重复的接口呢？根据上面的分析，我们知道，这两个配对的回调分别代表不同的意义，onStart和onStop是从Activity是否可见这个角度来回调的，除了这种区别，在实际的使用中，没有其他明显的区别
>
>第二个问题，我们就要从源码的角度来分析以及得到解释了，关于Activity的工作原理会在本书后续章节进行讲解，这里我们大致的了解即可，从Activity的启动过程来看，我们来看一下系统的源码，Activity启动过程的源码相当复杂，设计到了Instrumentation,Activit和ActivityManagerService（AMS），这里不详细分析这一过程，简单理解，启动Activity的请求会由 Instrumentation 来处理，然后他通过Binder向AMS发请求，AMS内部维护着一个ActivityStack，并负责栈内的Activity的状态同步，AMS通过ActivityThread去同步Activity的状态从而完成生命周期方法的调用，在ActivityStack中的resumeTopActivityLnnerLocked方法中，有这么这段代码

```

        //we need to start pausing the current activity so the top one can be resumed
        boolean dontWaitForPause = (next.info.flags& ActivityInfo.FLAG_RESUME_WHILE_PAUSING)!=0;
        boolean pausing = mStackSupervisor.pauseBackStacks(userLeaving, KeyStore.TrustedCertificateEntry,dontWaitForPause);
        if(mResumedActivity != null){
            pausing != startPaUSINGlOCAKED(userLeaving,false,true,dontWaitForPause);
            if(DEBUG_STATES){
                Slog.d(TAG,"resumeTopActivityLocked:pausing" + mResumedActivity);
            }
        }
```

>从上述的代码中我们可以看出，在新Activity启动之前，栈顶的Activity需要先onPause后，新的Activity才能启动，最终，在ActvityStackSupervisor中的realStartActivityLocked方法中，会调用如下代码

```
app.thread.scheduleLaunchActivity(new Intent(r.intent),r.appToken,System.identityHashCode(r),r.info,new Configuration(mService.mConfiguration)
        ,r.compat,r.task.voiceInteractor,app.repProcState,r.icicle,r.persistentState,results,new Intents,!andResume,mService.isNextTransitionForward()
        ,profilerInfo);
```
>我们都知道，在这个app.thread的类型是IApplicationThread的具体实现实在ActivityTread中,所以，这段代码实际上遇到了ActivityThread当中，，即ApplicationThread的scheduleLaunchActivity方法，而scheduleLaunchActivity方法最终会完成生命周期的调用过程，因此可以得出结论，是旧Activity县onPause，然后新的Activityy再启动

>至于ApplicationThread的scheduleLaunchActivity方法为什么会完成新Activity的生命周期，请看接下来的代码，scheduleLaunchActivty为什么会完成新的Activty

```
private void handlerLaunchActivity(ActivityClientRecord r, Intent customIntent){

        //if we are getting ready to gc after going to the background,well we are back active so skip it
        unscheduleGcIdler();
        mSomeActivitiesChanged =true;
        if(r.profilerInfo != null){
            mProfiler.setProfiler(r.profilerInfo);
            mProfiler.startProfiling;
        }
        //Make sure we are running with the most recent config
        handlerConfigurationChanged(null,null);

        if(localLOGV)Slog.v
                    TAG,"Handling launch of"+r);

        //在这里新Activity被创建出来，其onCreate和onStart被调用
        Activity a = PerformLaunchActivity(r,customIntent);

        if(a != null){
            r.createdConfig = new Configuration(mConfiguration);
            Bundle oldState = r.start;
            handlerResumeActivity(r.token,false,r.isForward,
                    !r.activity.mFinished && r.startsNotResumed);
        }
        //省略...
    }

```

>c从上面的分析可以看出，当新的Activity启动的时候，旧的Activity的onPause方法会先执行，然后才启动新的Activity，到底是不是这样尼？我们可以写一个小栗子来验证一下，如下是两个Activity的代码，在MainActivity中点击按钮可以跳转到SecondActivity，同时为了分析生命周期，我们把log日志也打出来

#### MainActivity

```
package com.liuguilin.activitysample;

import android.content.Intent;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.view.View;

public class MainActivity extends AppCompatActivity {

    public static final String TAG = "MainActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        findViewById(R.id.btnTo).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                startActivity(new Intent(MainActivity.this, SecondActivity.class));
            }
        });

    }

    @Override
    protected void onPause() {
        super.onPause();
        Log.i(TAG, "onPause");
    }

    @Override
    protected void onStop() {
        super.onStop();
        Log.i(TAG, "onStop");
    }
}

```

#### SecondActivity

```
package com.liuguilin.activitysample;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;

/**
 * Created by lgl on 16/8/24.
 */
public class SecondActivity extends AppCompatActivity {

    private static final String TAG = "SecondActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        Log.i(TAG, "onCreate");
    }

    @Override
    protected void onStart() {
        super.onStart();
        Log.i(TAG, "onStart");
    }

    @Override
    protected void onResume() {
        super.onResume();
        Log.i(TAG, "onResume");
    }
}


```
>这样我们可以观察到他的生命周期

![这里写图片描述](http://img.blog.csdn.net/20160910130112326)

>通过这个生命周期我们可以观察到，旧的Activity的onPause先调用，然后新的Activity才启动，这也证实了我们上面的分析原理，也许有人问，你只是分析了Andorid5.0的源码，你怎么所有的版本源码逻辑都相同，的确，我们不能把所有的版本都概括，但是作为Android的一个运行过程的基本逻辑，随着版本的更新并不会很大的改变，因为Android也需要兼容性，，不能说在同一个版本上运行有两种不同的逻辑，那根本不可能，关于这一点，我们要把握一个度，就是对于Android的基本运行机制，的不同，Android不能在onPause中做重量级的操作，因为必须在onPause执行完成以后新的Activity才能Resume，从这一点我们也间接性的证明了我们的结论，通过分析这个问题，我们知道onPause和onStop都不能做耗时的操作，尤其是onPause，这也意味着，我们应当尽量的在onStop中做操作，从而使新的Activity尽快显示出来并且换到前后台


## 三.异常情况下的生命周期分析

>上一节我们分析的是正常事情下的生命周期，但是我们写程序也不要理想化，居多的问题就是出在异常情况下，我们知道,Activity除了受用户操作导致的正常生命周期的调度，同时还会存在一些异常的情况，比如当资源相关的系统配置发生改变以及系统内存不足的时候，Activity就有可能被杀死，下面我们具体来分析下这几种情况

### 1.资源相关的系统配置发生改变导致Activity被杀死并重新创建

>理解这个问题，首先要对系统的资源加载有一定的了解，这里就不详细分析系统资源加载的机制了，但是我们简单说明一下，拿最简单的图片来说，当我们把一张图片挡在drawable中的时候，就可以通过Resources去获取这张图片了，同时为了兼容不同的设备，我们可能还需要在其他一些目录下放置不同的图片，比如drawable-xhdpi之类的，当应用程序启动时，系统会根据当前设备的情况去加载合适的Resources资源，比如说横屏手机和竖屏手机会拿着两张不同的图片（设定了landscape或者portrait状态下的图片），比如之前Activity处于竖屏，我们突然旋转屏幕，由于系统配置发生了改变，在默认情况下，Activity会被销毁并且重新创建，当然我们也可以阻止系统重新创建我们的Activity

>默认情况下，如果我们的Activity不做特殊处理，那么当系统配置发生改变之后，Activity就会销毁并且重新创建，可以看图

![这里写图片描述](http://img.blog.csdn.net/20160910132814613)

>当系统配置发生改变的时候，Activity会被销毁，其onPause，onStop,onDestroy均会被调用，同时由于Activity是异常情况下终止的，系统会调用onSaveInstanceState来保存当前Activity的状态，这个方法调用的时机是在onStop之前，他和onPause没有既定的时序关系，他即可能在onPause之前调用，也有可能在之后调用，需要强调的是，这个方法只出现在Activity被异常终止的情况下，正常情况下是不会走这个方法的，当我们onSaveInstanceState保存到Bundler对象作为参数传递给onRestoreInstanceState和onCreate方法，因此我们可以通过onRestoreInstanceState和onCreate方法来判断Activity是否被重建。如果被重建了，我们就取出之前的数据恢复，从时序上来说，onRestoreInstanceState的调用时机应该在onStart之后


>同时我们要知道，在onSaveInstanceState和onRestoreInstanceState方法中，系统自动为我们做了一些恢复工作，当Activity在异常情况下需要重新创建时，系统会默认我们保存当前的Activity视图架构，并且为我们恢复这些数据，比如文本框中用户输入的数据，ListView滚动的位置，这些View相关的状态系统都会默认恢复，具体针对某一个特定的View系统能为们恢复那些数据？我们可以查看View的源码，和Activity一样，每一个View都有onSaveInstanceState和onRestoreInstanceState这两个方法，看一下他们的实现，就能知道系统能够为每一个View恢复数据


>关于保存和恢复View的层次结构，系统的工作流程是这样的：首先Activity被意外终止时，Activity会调用onSaveInstanceState去保存数据，然后Activity会委托Window去保存数据，接着Window再委托上面的顶级容器去保存数据，顶级容器是一个ViewGroup，一般来说他可能是一个DecorView,最后顶层容器再去一一通知他的子元素来保存数据，这样整个数据保存过程就完成了，可以发现，这是一种典型的委托思想，上层委托下层，父容器委托子容器，去处理一件事件，这种思想在Android 中有很多的应用，这里就不再重复介绍了，接下来举个例子，那TextView来说，我们分析一下他到底保存了那些数据


```
 @Override
    public Parcelable onSaveInstanceState() {
        Parcelable superState = super.onSaveInstanceState();

        // Save state if we are forced to
        boolean save = mFreezesText;
        int start = 0;
        int end = 0;

        if (mText != null) {
            start = getSelectionStart();
            end = getSelectionEnd();
            if (start >= 0 || end >= 0) {
                // Or save state if there is a selection
                save = true;
            }
        }

        if (save) {
            SavedState ss = new SavedState(superState);
            // XXX Should also save the current scroll position!
            ss.selStart = start;
            ss.selEnd = end;

            if (mText instanceof Spanned) {
                Spannable sp = new SpannableStringBuilder(mText);

                if (mEditor != null) {
                    removeMisspelledSpans(sp);
                    sp.removeSpan(mEditor.mSuggestionRangeSpan);
                }

                ss.text = sp;
            } else {
                ss.text = mText.toString();
            }

            if (isFocused() && start >= 0 && end >= 0) {
                ss.frozenWithFocus = true;
            }

            ss.error = getError();

            return ss;
        }

        return superState;
    }


```

>从上述源码中我们可以看到，TextView为了保存自己的文本选中和文本结构内容，并且通过查看onRestoreInstanceState方法的源码，可以发现它的确恢复了这些数据，具体源码就不在贴出，读者可以自己去看下源码，下面我们看来看下实际的例子，对比一下Activity正常终止和异常终止的不同，同时验证一下系统的数据恢复能力，为了方便测试，我们采用了旋转屏幕来终止Activity，在我们旋转屏幕以后，Activity被销毁重建，我们输入的文本被正确还原了，说明我们的系统能够正确的做一些View层的分析，我们看下代码

```
package com.liuguilin.activitysample;

import android.os.Bundle;
import android.os.PersistableBundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;

public class MainActivity extends AppCompatActivity {

    public static final String TAG = "MainActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if (savedInstanceState != null) {
            String test = savedInstanceState.getString("extra_test");
            Log.i(TAG, test);
        }
    }

    @Override
    public void onSaveInstanceState(Bundle outState, PersistableBundle outPersistentState) {
        super.onSaveInstanceState(outState, outPersistentState);
        Log.i(TAG, "onSaveInstanceState");
        outState.putString("extra_test", "test");
    }

    @Override
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);
        String test = savedInstanceState.getString("extra_test");
        Log.i(TAG, test);
    }
}

```

>上面的代码很简单，首先我们在onSaveInstanceState中保存一个字符串，然后当我们的Activity被销毁并且重新创建之后，我们再去获取之前存储的字符串，接收的位置可以选择onRestoreInstanceState或者onCreate，两者的区别是：onRestoreInstanceState一旦被调用，其参数Bundler savedInstanceState一定有值，我们不用额外的判断是否为空但是onCreate不行，onCreate如果正常启动的话，其参数Bundler onSaveInstanceState为null,所以需要一些额外的判断，这两个方法我们选择任意一个都是可以进行数据恢复的，但是关键建议我们使用onRestoreInstanceState去恢复数据


>Activity销毁后调用了onSaveInstanceState来保存数据，重新创建以后再onCreate和onRestoreInstanceState中能恢复数据，这个正好证明了我们刚才的分析，针对onSaveInstanceState我们有一点要说明，那就是系统只会在即将被销毁并且有机会重新显示的情况下才会去调用它，考虑到这一种情况，当Activity正常销毁的时候，系统不会调用onSaveInstanceState，因为被销毁的Activity不可能再次被显示出来，这句话不好理解，但是我们可以对比一下旋转屏幕所造成的Activity异常销毁，这个过程和正常停止的Activity是不一样的，因为旋转屏幕之后，Activity被销毁的同时会立即创建新的Activity实例，这个时候Activcity有机会再次立刻显示，所以系统进行了数据存储，这里可以简单的这么理解，系统只在Activity异常终止的情况下才会调用onSaveInstanceState和onRestoreInstanceState来存储和恢复数据，其他情况不会触发

### 2.资源内存不足导致低优先级的Activity被杀死

>这个情况我们不好模拟，但是其数据的存储和恢复过程和情况一是一致的，这里我们描述一下Activity的优先级情况，Activity按照优先级的从高往低，可以分为三种：

- 1.前台Activity:正在和用户交互的Activity，优先级最高
- 2.可见但非前台Activity:比如对话框，导致Activity可见但是位于后台无法和用户直接交互
- 3.后台Activity:已经被暂停的Activity，比如执行了onStop，优先级最低

>当系统内存不足的时候，系统就会按照上述优先级去杀死目标Activity所在的进程，并且在后续通过onSaveInstanceState和onRestoreInstanceState来存储和恢复数据，如果一个进程中没有四大组件在执行，那么这个进程将很快被系统杀死，因此，一些后台工作不适合脱离四大组建而独立运行在后台中，这样进程很容易就被杀死了，比较好的方法就是将后台工作放在Service中从而保证了进程有一定的哟徐爱你集，这样就不会轻易的被杀死

>上面分析了系统的数据存储和恢复机制，我们知道，当系统配置发生改变后，Activity会被重新创建，那我们有没有什么办法不重新创建尼？答案是有的，接下来我们来分析一下这个问题，系统配置中有很多内容，如果当某项内容发生改变后，我们不想系统重新创建，就可以给configChangs属性加上orientation这个值

```
android:configChanges="orientation"

```
>如果想指定多个值的话可以用“|”连接起来

**mcc:The IMSI mobile country code (MCC) has changed — a SIM has been detected and updated the MCC.
IMSI(国际移动用户识别码)发生改变，检测到SIM卡，或者更新MCC**

**mnc:The IMSI mobile network code (MNC) has changed — a SIM has been detected and updated the MNC.
IMSI网络发生改变,检测到SIM卡，或者更新MCC其中mcc和mnc理论上不可能发生变化**

**locale:The locale has changed — the user has selected a new language that text should be displayed in.
语言发生改变，用户选择了一个新的语言，文字应该重新显示**

**touchscreen:The touchscreen has changed. (This should never normally happen.)
触摸屏发生改变，这通常是不应该发生的**

**keyboard:The keyboard type has changed — for example, the user has plugged in an external keyboard.
键盘类型发生改变，例如，用户使用了外部键盘**

**keyboardHidden:The keyboard accessibility has changed — for example, the user has revealed the hardware keyboard.
键盘发生改变，例如，用户使用了硬件键盘**

**navigation:The navigation type (trackball/dpad) has changed. (This should never normally happen.)
导航发生改变，（这通常不应该发生） 举例：连接蓝牙键盘，连接后确实导致了navigation的类型发生变化。因为连接蓝牙键盘后，我可以使用方向键来navigate了**

**screenLayout：The screen layout has changed — this might be caused by a different display being activated.
屏幕的布局发生改变，这可能导致激活不同的显示**

**ontScale：The font scaling factor has changed — the user has selected a new global font size.
全局字体大小缩放发生改变**

**orientation：The screen orientation has changed — that is, the user has rotated the device.设备旋转，横向显示和竖向显示模式切换。**

**screenSize: 屏幕大小改变了**

**smallestScreenSize: 屏幕的物理大小改变了，如：连接到一个外部的屏幕上**

**4.2增加了一个layoutDirection属性，当改变语言设置后，该属性也会成newConfig中的一个mask位。所以ActivityManagerService(实际在ActivityStack)在决定是否重启Activity的时候总是判断为重启。
需要在android:configChanges 中同时添加locale和layoutDirection。
在不退出应用的情况下切换到Settings里切换语言，发现该Activity还是重启了。**


>从上面的属性中我们可以知道，如果我们没有在Activity的configChanges中设备属性的话，当系统发生改变后就会导致Activity重新被创建，上面表格中的项目很多，但是我们常用的只有locale,orientation，keyboardHidden这三个选项，其他用的还是比较少的，这里设置之后显示的效果我就不演示了


## 四.Activity的启动模式
>Android的启动模式，是很有用的，对于Activity的栈的处理，也是极其讲究的，所以你一定要清除他的标志位和启动模式

### 一.Activity的LaunchMode
>首先说一下Activity为什么需要启动模式，我们知道，在默认的情况下，当我们多次启动同一个Activity的时候，系统会创建多个实例并把他们一一放入任务栈中，当我们点击back键的时候会发现这些Activity会一一回退，任务栈是一种先进先出的栈结构，这个好理解， 每按一次back键就有一个activity退出栈，知道栈空为止，当这个栈为空的时候，系统就会回收这个任务栈，关于任务栈的系统工作原理，这里我们暂且不说，在后续章节也会介绍任务栈，知道了Activity的启动模式，我们可发现一个问题，：多次启动同一个Activity会创建多个实例，这样是不是很逗，Activity在设计的时候不可能不考虑到这个问题，所以他提供了启动模式来修改系统的默认行为，目前有四种启动模式

- standard
- singleTop
- singleTask
- singleInstance

>我们先来把这几种启动模式都给介绍完

- standard:标准模式，这也是系统的默认模式，每次启动一个Activity都会重新创建一个实例，是否这个实例已经存在，被创建的实例的生命周期符合典型情况下Activity的生命周期，如上述：onCreate(),onStart();onResume()都会被调用，这是一种典型的多实例实现，一个任务栈都可以有多个实例，每个实例都可以属于不同的任务栈，在这种模式下，谁启动了这个Activity，那么这个Activity就运行在启动它的Activity所在的栈内，比如Activity A启动了Activity B（B是标准模式），那么B就会进入到A所在的栈内，不知道读者有没有注意到，当我们用ApplicationContext去启动standard模式的Activity的时候就会报错：


```
E/AndroidRuntime(674): android.util.androidruntiomException: Calling startActivity from outside of an Activity context requires the FLAG_ACTIVITY_TASK flag . Is this really what are want?
```
>相信读者对这句话不会陌生，这是因为我们的standard模式的Activity默认会进入启动它的Activity所属的任务栈中，但是由于非Activity类型的Context（如ApplicationContext）并没有所谓的任务栈，所以这就有问题了，解决这个问题，就是待启动Activity指定FLAG_ACTIVITY_TASK标记位，这样启动的时候就会为他创建一个新的任务栈，这个时候待启动Activity实际上是以singleTask模式启动的，读者可以仔细体会


- singleTop:栈顶复用模式，在这个模式下，如果新的Activity已经位于任务栈的栈顶，那么此Activity不会被重新创建，同时他的onNewIntent方法会被调用，通过此方法的参数我们可以取出当前请求的信息，需要注意的是，这个Activity的onCreate,onStart不会被系统调用，因为他并没有发生改变，如果新Activity已存在但不是在栈顶，那么新Activity则会重新创建，举个例子，假设现在栈内的情况为ABCD，其中ABCD为四个Activity，A位于栈底，D位于栈顶，这个时候假设要再启动D，如果D的启动模式为singleTop，那么站栈内的情况仍然是ABCD，如果D的启动模式是standard，那么由于D会被重新创建，导致情况就是ABCDD


- singTask：栈内复用模式，这是一种单实例模式，在这种模式下，只要Activity在一个栈内存在，那么多次启动此Activity都不会创建实例，和singTop一样，系统也会回调其onNewIntent方法，具体一点，当一个具有singleTask模式的Activity请求启动后，比如Activity A，系统首先会去寻找是否存在A想要的任务栈，如果不存在，就小红心创建一个任务栈，然后创建A的实例把A放进栈中，如果存在A所需要的栈，这个时候就要看A是否在栈中有实例存在，如果实例存在，那么系统就会把A调到栈顶并调用它的onNewIntent方法，如果实例不存在，就创建A的实例并且把A压入栈中，举几个例子

	- 比如目前任务栈S1中的情况为ABC，这个时候Activity D以singleTask模式请求启动，其所需的任务栈为S2，由于S2和D的实例都不存在，所以系统会先创建任务栈S2，然后创建D的实例将其入栈到S2
	- 另外一种情况，假设D所需的任务栈为S1，其他情况如如上面的一样，那么由于S1已经存在，所以系统会直接创建D的实例并将其引入到S1中
	- 如果D所需要的任务栈为S1，并且当前任务栈S1的情况为ABCD，根据栈内复用的原则，此时D不会被重新创建，系统会把D切换到栈顶并且调用其oNnNewIntent方法，同时由于singleTask默认具有clearTop的效果，会导致栈内所有在D上面的Activity全部出栈，于是最终S1中的情况为AD，这一点比较特殊，在后面还会对此情况详细的分析


>通过上述的三个例子，读者应该还是比较清晰的理解singTask的含义吧


- singleInstance:单实例模式，这是一种加强的singleTask的模式，他除了具有singleTask的所有属性之外，还加强了一点，那就是具有此模式下的Activity只能单独的处于一个任务栈中，换句话说，比如Activity A是singleInstance模式，当A启动的时候，系统会为创建创建一个新的任务栈，然后A独立在这个任务栈中，由于栈内复用的特性，后续的请求均不会创建新的Activity，除非这个独特的任务栈被系统销毁了



>上面介绍了几种启动模式，这里需要指出一种情况，我们假设目前有两个任务栈，前台任务栈的情况为AB，而后台任务栈的情况是CD，这里假设CD的启动模式都是singleTask，现在请求启动D，那么整个后台任务站都会被切换到前台，这个时候整个后退列表变成了ABCD，当用户按back键的时候，列表中的Activity会一一出栈，如图

![这里写图片描述](http://img.blog.csdn.net/20160915150453950)

>如果不是请求D，而是请求C，那么情况就不一样了，如图，具体原因我们在后续章节中分析

![这里写图片描述](http://img.blog.csdn.net/20160915150502981)


>另外一个问题，在singkleTask启动模式中，多次提到了某个Activity所需的任务栈，什么是Activity所需的任务栈尼？这要从一个参数说起：TaskAffinity,可以翻译成任务相关性，这个参数标示了一个Activity所需要的任务栈的名字默认情况下，所有的Activity所需要的任务栈的名字为应用的包名，当然，我们可以为每个Activity都单独指定TaskAffinity，这个属性值必须必须不能和包名相同，否则就相当于没有指定，TaskAffinity属性主要和singleTask启动模式或者allowTaskReparenting属性配合使用，在其他状况下没有意义，另外，任务栈分为前台任务栈和后台任务栈，后台任务栈中的Activity位于暂停状态，用户可以通过切换将后台任务栈再次调为前台

>当TaskAffinity和singleTask启动模式配对使用的时候，他是具有该模式Activity目前任务栈的名字，待启动的Activity会运行在名字和TaskAffinity相同的任务栈中

>当TaskAffinity和allowTaskReparentiing结合的时候，这种情况比较复杂，会产生特殊的效果，当一个应用A启动了应用B的某一个Activity后，如果这个Activity会直接从应用A的任务栈转移到应用B的任务栈中，这还是很抽象的，再具体点，比如现在有2个应用A和B，A启动了B的一个Activity C ，然后按Home键回到桌面，然后再单击B的桌面图标，这个时候并不是启动； B的主Activity，而是重新显示了已经被应用A启动的Activity C,或者说，C从A的任务栈转移到了B的任务栈中，可以这么理解，由于A启动了C，这个时候C只能运行在A的任务栈中，但是C属于B应用，正常情况下，他的TaskAffinity值肯定不可能和A的任务栈相同（因为包名不同），所以，当B启动后，B会创建自己的任务栈，这个时候系统发现C原本所想要的任务栈已经被创建出来了，所以就把C从A的任务栈中转移过来，这种情况读者可以写一个例子测试一下，这里就不做演示了

>如何给Activity指定启动模式？有两种方法，第一种是通过清单文件为Activity指定
	
```
<activity android:name=".SecondActivity"
            android:launchMode="singleTask"/>

```

>另一种启情况就是通过intent的标志位为Activity指定启动模式


```
Intent intent = new Intent();
intent.setClass(this,SecondActivity.class);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);

```

>这两种方式都可以为Activity指定启动模式，但是二者还是有一些区别的，首先，优先级上，第二种比第一种高，当两种同时存在的时候，以第二种为准，其次，上述两种方式在限定范围内有所不同，比如，第一种方式无法直接为Activity设置FLAG_ACTIVITY_CLEAR_TOP标识，而第二种方式无法指定singleInstance模式

>关于Intent中为Activity指定的各种标记位，在下面的小节中会继续说道，下面通过一个实例来体验启动模式的使用效果，还是前面的例子，我们把MainActivity的启动模式设置成singleTask，然后重复启动它，看看他是否会重复创建

```
	 <activity
            android:name=".MainActivity"
            android:configChanges="orientation|screenSize"
            android:launchMode="singleTask">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

```



```
	
        //点击事件
        findViewById(R.id.btnTo).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent = new Intent();
                intent.setClass(MainActivity.this,MainActivity.class);
                intent.putExtra("time", System.currentTimeMillis());
                startActivity(intent);
            }
        });


```


```

    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        Log.d(TAG, "onNewIntent , time =" + intent.getLongExtra("time", 0));
    }

```

>上述修改，我们做了如下操作，连续点击三次按钮启动三次，算上原本的MainActivity实例，正常情况下，任务栈中应该有四个MainActiivty实例，但是我们为其指定了singTask模式，我们一起来看下有何不同

>执行命令


```
adb shell dumpsys activity

```

>ok，输入日志

![这里写图片描述](http://img.blog.csdn.net/20160915164312051)

>从上面的信息中不难看出，尽管启动了四次MainActivity，但是他始终只有一个实例在任务栈中，从log中我们也可以看到

![这里写图片描述](http://img.blog.csdn.net/20160915164658026)

>Activity的确没有被创建，只是暂停了一下，然后调用了onNewIntent，接着又onResume又继续了，现在我们去掉singTask，再来比对一下我们的操作，同样是点击三次按钮，执行adb命令

![这里写图片描述](http://img.blog.csdn.net/20160915182729817)

>我们能够得到目前总共有2个任务栈，前台任务栈taskAffinity值为包名。里面有四个Activity，后台是一个com.android.launcher，他里面就一个桌面


>从上面的导出信息可以看到，在任务栈中有四个MainActivity，这也就验证了Activity的启动模式的工作方式

上述四种启动模式，standard和singleTop都比较好理解，singleInstance由于其特殊性也比较好理解，但是关于singleTask还有一种情况需要理解，比如我们刚才那张启动D的图，在Activity B中请求的不是D而是C，那情况该如何尼？这里可以告诉读者的是，任务列表变成了ABC，是不是很奇怪，Activity为什么直接出栈了，我们用一个实例来说明情况


```
	   <activity
            android:name=".MainActivity"
            android:configChanges="orientation|screenSize"
            android:launchMode="standard">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity
            android:name=".SecondActivity"
            android:configChanges="screenLayout"
            android:launchMode="singleTask"
            android:taskAffinity="com.liuguilin.activitysample1"  />

        <activity
            android:name=".CodeActivity"
            android:configChanges="screenLayout"
            android:launchMode="singleTask" 
            android:taskAffinity="com.liuguilin.activitysample1" />

```

>我们把SecondActivity和CodeActivity的启动模式都改成了singleTask，并且把android:taskAffinity设置为com.liuguilin.activitysample，注意这个taskAffinity的值属于字符串，切中间必须有包名分割符‘.’，然后我们做如下的动作，在MainActivity中单击按钮启动SecondActivity，在SecondActivity中单击按钮启动CodeActivity,在CodeActivity启动MainActivity,最后在MainActivity中启动SecondActivity，，现在按Back，你知道会回那个Activity吗？答案是桌面，是不是有点摸不着头脑，我们看图理解一下

![这里写图片描述](http://img.blog.csdn.net/20160915191032092)


>首先，我们来分析一下这个问题，我们知道，A的启动模式是standard,按照规定，A的taskAffinity继承的是application的taskAffinity，而application默认taskAffinity是包名，所以A的taskAffinity是包名，由于我们在XML中为B和C指定了taskAffinity和启动模式，且有相同的taskAffinity，所以B和C是singleTask模式且有相同的taskAffinity，所以A启动B的时候，按照singleTask的规则，这个时候需要为B重新创建一个任务栈了，B再启动C，按照singleTask的规则，由于C所需要的任务栈已经被B给创建了，所以无需再创建新的任务栈，这个时候系统只是创建C的实例放进任务栈，接着C再启动A，A是标准模式，所以系统会为他创建一个新的实例并将他加入到启动的任务栈中，由于是C启动了A，所以A会进入C的栈内并处于栈顶，这个时候已经有两个任务栈了，接着A再启动B，由于B是singleTask,B需要回到任务栈的栈顶，由于栈的模式为‘先进先出’，B想要回到栈顶，就只能是CA出栈，所以到这里就很好理解，按back键，B就出栈了，然后这个任务栈就是空的，被系统回收了，这个时候就只能是回到后台任务栈把A显示出来，注意这个A是后台任务栈的A，不是BC栈的A，接着再按back就回到了桌面，分析到这里，我们就得到了一条结论，singtleTask模式的Activity切换到栈顶会到导致在他之上的栈内activity出栈，我们可以看下运行的结果

![这里写图片描述](http://img.blog.csdn.net/20160916115608557)


>接着我们再实验中再次验证这个问题，我们采用dumpasys命令，看看输出的是什么？

![这里写图片描述](http://img.blog.csdn.net/20160916123614903)

>可以看到在B任务栈中只剩下B了，其他的都出栈了，这个时候按back肯定就回收了，分析到这里，我相信读者对Activity的启动模式有了很深入的了解吧，下面我们再来说下Activity的标志位

###二.Activity 的 Flags
>Activity的Flags有很多，这里主要是分析一些常用的标记位，标记位的作用很多，有些标志位可以设置Activity的启动模式，比如FLAG_ ACTIVITY _ NEW _ TASK，还有一些直接影响Activity的运行状态，比如FLAG_ ACTIVITY_ CLEAR_ TOP，下面我们来说下一些常用的标记位，剩下的读者可以去看下官方文档，大部分的情况下，Activity不需要设置标记位，因此对于标记位理解即可，在使用标记位的时候，要注意有些标记位是系统内部使用的，应用不需要去设置这些以防出问题。

- FLAG_ ACTIVITY_ NEW _ TASK
>这个标志位的作用是为Activity指向‘singleTask’启动模式，其效果和XML中指定该模式相同

- FLAG_ ACTIVITY_ SINGLE _ TOP
>这个标志位的作用是为Activity指向‘singleTop’启动模式，其效果和XML中指定该模式相同

- FLAG_ ACTIVITY_ CLEAR _ TOP
>具有此标记位的Activity,当他启动时，在同一个任务栈中所有位于他上面的Activity都要出栈，这个模式一般需要和FLAG_ ACTIVITY_ NEW _ TASK配合使用，在这种情况下，被启动的Activity的实例如果已经存在，那么系统就会调用它的onNewIntent,如果被启动的Activity采用标准模式，那么他连同他之上的Activity都要出栈，系统会创建新的Activity实例并放入栈顶

- FLAG_ ACTIVITY_ EXCLUDE_ FROM _ RECENTS
>具有此标记位的Activity，不会出现在历史Activity的列表当中，当某种情况下我们不希望用户通过历史列表回到我们的Activity的时候就使用这个标记位了，他等同于在XML中指定Activity的属性：

```
	android:excludeFromRecents="true"

```

###三.intentFilter的匹配规则
>我们知道，启动Activity分为两种，显示调用和隐式调用，二者的区别这里就不多讲了，显示调用需要明确的指定被启动对象的组件信息，包括包名和类名，而隐式意图则不需要明确指定调用信息，原则上一个intent不应该即是显式调用又是隐式调用，如果二者共存的话以显式调用为主，显式调用很简单，这里主要介绍隐式调用，隐式调用需要intent能够匹配目标组件的IntentFilter中所设置的过滤信息，如果不匹配将无法启动目标Activity，IntentFilter中的过滤信息有action,category,data,下面是一个过滤规则的实例：


```
		<activity
            android:name=".CodeActivity"
            android:configChanges="screenLayout"
            android:launchMode="singleTask"
            android:taskAffinity="com.liuguilin.activitysample1">
            <intent-filter>
                <action android:name="com.liuguilin.activitysample.c" />
                <action android:name="com.liuguilin.activitysample.d" />

                <category android:name="com.liuguilin.category.c" />
                <category android:name="com.liuguilin.category.d" />

                <data android:mimeType="text/plain" />
            </intent-filter>
        </activity>

```

>为了匹配过滤列表，需要同时匹配过滤列表中的action,category,data信息，否则匹配失败，一个过滤列表中的action,category,data可以有多个，所有的action,category,data分别构成不同类别，同一类型的信息共同约束当前类别的匹配过程，只有一个intent同时匹配action类别,category类别,data类别才算是匹配完成，只有完全匹配才能成功启动目标Activity，另外一点，一个Activity钟可以有多个intent-filter,一个intent只要能匹配一组intent-filter即可成功启动Activity

```
		 <activity android:name=".ShareActivity">
            <!-- This activity handlers "SEND" actions with text data-->
            <intent-filter>
                <action android:name="android.intent.action.SEND" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:mimeType="text/plain" />
            </intent-filter>
            <!--This activity also handlers "SEND" and "SEND_MULTIPLE" with media data-->
            <intent-filter>
                <action android:name="android.intent.action.SEND" />
                <action android:name="android.intent.action.SEND_MULTIPLE" />

                <category android:name="android.intent.category.DEFAULT" />

					<data android:mimeType="application/vnd.google.panorama360+jpg"/>
                <data android:mimeType="image/*" />
                <data android:mimeType="text/plain" />
            </intent-filter>
        </activity>
```

>下面详细分析各种属性的匹配规则

- 1.action的匹配规则
>action是一个字符串，系统预定了一些action,同时我们也可以在应用中定义自己的action,action的匹配规则是intent中的action必须能够和过滤规则中的action匹配，这里说的匹配是指action的字符串值完全一样，一个过滤规则中的可以有多个action,那么只要intent中的action能够和过滤规则匹配成功，针对上面的过滤规则，需要注意的是，intent如果没有指定action，那么匹配失败，总结一下，action的匹配需求就是intent中的action存在且必和过滤规则一样的action，这里需要注意的是他和category匹配规则的不同，另外，action区分大小写，大小写不同的字符串匹配也会失败

- 2.category的匹配规则
> category是一个字符串，系统预定义了一些category，同时我们也可以在应用中定义自己的category。category的匹配规则和action不同，它要求Intent中如果含有category，那么所有的category都必须和过滤规则中的其中一个category相同。换句话说，Intent如果出现了category，不管有几个category，对于每个category来说，它必须是过滤规则中已经定义的category。当然，Intent中可以没有category，如果没有category的话，按照上面的描述，这个Intent仍然可以匹配成功。这里要注意下它和action匹配过程的不同，action
是要求Intent中必须有一个action且必须能够和过滤规则中的某个action相同，而category要求Intent可以没有category，但是如果你一旦有category，不管有几个，每个都要能和过滤规则中的任何一个category相同。为了匹配前面的过滤规则中的category，我们可出下面的Intent,intent.addcategory ("com.ryg.category.c”)或者Intent.addcategory ("com rcategory.d)亦或者不设category。为什么不设置category也可以匹配呢？原因是系统在调用startActivity或者startActivityForResult的时候会默认为Intent加上“android.intent.category.DEFAULT”这个category，所以这个category就可以匹配前面的过滤规则中的第三个category。同时，为了我们的activity能够接收隐式调用，就必须在intent-filter中指定“android intent categor.DEFAULT”这个category，原因刚才已经说明了。

- 3.data匹配规则
>data的匹配规则和action有点类似，如果过滤规则中定义了data,那么intent中必须也要定义可匹配的data,在介绍data的匹配规则之前，我们需要来了解一下data的结构，因为data稍微有点复杂

```
 	<data
       android:host="string"
       android:mimeType="string"
       android:path="string"
       android:pathPattern="string"
       android:pathPrefix="string"
       android:port="string"
       android:scheme="sstring" />

```	

>data由两部分组成，mimeType和URI，前者是媒体类型，比如image/jpeg等，可以表示图片等，而URI包含的数据可就多了，下面的URI的结构：

```
	<scheme>://<host>"<port>/[<path>|<pathPrefix>|<pathPattern>]

```	

>这里再给几个实际的例子就好理解了

```
content://com.liuguilin.project:200/folder/subfolder/etc
http://www.baidu.com:80/search/info
```	

>看了上面的两个例子你是不是瞬间就明白了，没错，就是这么简单，不过下面还是要一一介绍含义的：

- Scheme:URI的模式，比如http.file.content等，如果URI中没有指定的scheme,那么整个URI的其他参数无效，这也意味着URI无效
- Host:URI的主机，比如www.google.com,如果host未指定，那么整个URI中的其他参数无效，这也意味着URI无效
- Port:URI中的端口号，比如80，不过需要指定上面两个才有意义
- Path、pathPattem 和 pathPrefix：这三个参数表述路径信息，其中path表示完整的路径信息：pathPattern也表示完整的路径信息，但是它里面可以包含通配符“ * ”，“ * ” 表示0个或多个任意字符，需要注意的是，由于正则表达式的规范，如果想表示真实的字符串，那么“* ” 要写成 “ \\*”，“ \ ”要写成“ \\\ ”,pathPrefix表示路径的前缀信息。


>介绍完data的数据格式后，我们要说一下data的匹配规则了。前面说到，data的匹配规则和action类似，它也要求Intent中必须含有data数据，并且data数据能够完全匹配过滤规则中的某一个datn.这里的完全匹配是指过滤规则中出现的data部分也出现在了 Intent
中的data中。下面分情况说明。

- (1) 如下过来规则

```	
	   <intent-filter>
          <data android:mimeType="image/*"/>
      </intent-filter>

```

>这种规则指定了所有类型为图片，那么intent中的mineType属性必须为“image/*”才能匹配，这种情况下虽然过来规则没有指定URI，但是却有默认值，URI的默认值为content何file,也就是说，虽然没有指定URI，但是Intent中的URI部分的scheme必须为content或者file才能匹配，这点事需要注意的，为了匹配一种的规则我们可以这样写：

```	
	intent.setDataAndType(Uri.parse("file://abc"),"image/png");
```

>另外，如果要为Intent指定完整的data，必须调用setDataAndType方法，不能县调用setData在调用setType,因为这两个方法彼此会清除对方的值，这个看源码就比较好理解了，比如setData:

```	
	public Intent setData(Uri data) {
        mData = data;
        mType = null;
        return this;
    }
```

>可以发现，setData会把类型设置为null，同样的，对方也是


- （2）如下规律规则

```	
 	  <intent-filter>
                <data android:mimeType="video/mpeg" android:scheme="http" .../>
                <data android:mimeType="audio/mpeg" android:scheme="http" .../>
      </intent-filter>
```

>这种规则指定了两组data规则，且每个data都指出了完整的属性值，既有URI又有类型，为了匹配类型二，我们这样写“


```	
	intent.setDataAndType(Uri.parse("http://abc"),"video/png");
```

>或者


```
	intent.setDataAndType(Uri.parse("http://abc"),"audio/png");	
```

>通过上面的实例，我们应该知道了data的匹配规则，关于data还有一些特殊的情况需要说明一下，这也是他和action不同的地方


```	
  <intent-filter >
     <data android:scheme="file" android:host="www.baidu.com"/>
         ...
 </intent-filter>
 
 <intent-filter >
       <data android:scheme="file" />
       <data android:host="www.baidu.com"/>
         ...
  </intent-filter>
```

>到这里我们已经把IntentFilter的过滤规则都讲了一遍了，还记得本书前面给出的一个实例吗？现在我们给出完整的intent匹配规则

```	
	Intent intent = new Intent();
    intent.addCategory("com.liuguilin.category.c");
    intent.setDataAndType(Uri.parse("file//abc"),"text/plain");
    startActivity(intent);
```

>还记得URI中的scheme中的默认值吗？如果把上面的intent.setDataAndType(Uri.parse("file//abc"),"text/plain");这句改成intent.setDataAndType(Uri.parse("http//abc"),"text/plain");打开的actiivty就会报错，提示无法找到Activity，另外一点，intent-filter的匹配规则对于服务和广播也是同样的道理，不过系统对于Service的建议是尽量使用显式意图来启动服务。

>最后，当我们通过隐式方式启动一个Activity的时候，可以做一下判断，看是否Activity能够匹配我们的隐式Intent，如果不做判断就有可能出现上述的错误了。判断方法有两种：采用PackageManager的resolveActivity方法或者Intent 的resolveActivity方法，
果它们找不到匹配的Activity就会返回null，我们通过判断返回值就可以规避上述错误了，另外，PackageManager还提供了queryIntentActivities方法，这个方法和resolveActivity方法法不同的是：它不是返回最佳匹配的Activity信息而是返回所有成功匹配的Activity信息，我们看一下queryIntentActivities和resolveActivity的方法原型：

```
	public abstract List<ResolveInfo>queryIntentActivities(Intent intent,int fladgs);
	public abstract ResolveInfo resolveActivity(Intent intent,int flags);
```

>上述两个方法的第一个参数比较好理解，第二个参数需要注意，我们要使用MATCH_ DEFAULT _ ONLY这个标记位，这个标记位的含义是仅仅匹配那些在intentfilter中声明了 < category android-name="android.intent.category DEFAULT">这个category的 Activity。使用这个标记位的意义在于，只要上述两个方法不返回null，那么startActivity一定可以成功，如果不用这个标记位，就可以把intent-filter 中 category不含DEFAULT的那些Activity给匹配出来，从而导致startActivity可能失败。因为不含有DEFAULT这个category的Activity是无法接收隐式Intent的。在action和 category中，有一类action和category比较重要，他们是：


```	
	<action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
```

>这二者共同作用是用来标明这是一个入口Activity并且会出现在系统的应用列表中,少了任何一个都没有实际意义，也无法出现在系统的应用列表中，也就是二者缺一不可，另外，针对 Service和BroadcastReceiver,PackageManager同样提供了类似的方法去获取成功匹配的组件信息。

>好的，我们的第一章就写到这里了，不得不说这是一本好书，非常的详细，也希望大家仔细的阅读


