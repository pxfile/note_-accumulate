# TaskAffinity

正常情况下，如果应用已经启动，并将应用切到后台，在通知栏中调起页面时，该应用的Task首先会被调起，然后会将我们的Activity显示在这个Task的顶端。手机百度的通知栏里面有一个快速搜索栏，无论什么情况下，点击之后都会直接弹出搜索页面，透明背景后显示的是桌面。怎么来实现这个功能呢？这就要提到我们的主角TaskAffinity了。

####什么是affinity？
　　affinity是指Activity的归属，Activity与Task的吸附关系，也就是该Activity属于哪个Task。一般情况下在同一个应用中，启动的Activity都在同一个Task中，它们在该Task中度过自己的生命。每个Activity都有taskAffinity属性，这个属性指出了它希望进入的Task。如果一个Activity没有显式的指明taskAffinity，那么它的这个属性就等于Application指明的taskAffinity，如果Application也没有指明，那么该taskAffinity的值就等于应用的包名。我们可以通过在元素中增加taskAffinity属性来为某一个Activity指定单独的affinity。这个属性的值是一个字符串，可以指定为任意字符串，但是必须至少包含一个”.”，否则会报错。

####affinity在什么场合应用呢？
* 1.根据affinity重新为Activity选择宿主task（与allowTaskReparenting属性配合使用）

allowTaskReparenting用来标记Activity能否从启动的Task移动到taskAffinity指定的Task，当把Activity的allowTaskReparenting属性设置成true时，Activity就拥有了一个转移所在Task的能力。具体点来说，就是一个Activity现在是处于某个Task当中的，但是它与另外一个Task具有相同的affinity值，那么当另外这个任务切换到前台的时候，该Activity就可以转移到现在的这个任务当中。allowTaskReparenting默认是继承至application中的allowTaskReparenting=false，如果为true，则表示可以更换；false表示不可以。 
　　举一个形象点的例子，比如有一个天气预报程序，它有一个用于显示天气信息的Activity，allowTaskReparenting属性设置成true，这个Activity和天气预报程序的所有其它Activity具体相同的affinity值。这个时候，你自己的应用程序通过Intent去启动了这个用于显示天气信息的Activity，那么此时这个Activity应该是和你的应用程序是在同一个任务当中的。但是当把天气预报程序切换到前台的时候，这个Activity会被转移到天气预报程序的任务当中，并显示出来。如果将你自己的应用切换到前台，发现你自己应用Task里的那个Activity消失了。

* 2.启动一个Activity过程中Intent使用了FLAG_ACTIVITY_NEW_TASK标记，根据affinity查找或创建一个新的具有对应affinity的task。

　　当调用startActivity()方法来启动一个Activity时，默认是将它放入到当前的任务当中。但是，如果在Intent中加入了FLAG_ACTIVITY_NEW_TASK flag的话，情况就会变的复杂起来。首先，系统会去检查这个Activity的affinity是否与当前Task的affinity相同。如果相同的话就会把它放入到当前Task当中，如果不同则会先去检查是否已经有一个名字与该Activity的affinity相同的Task,如果有，这个Task将被调到前台，同时这个Activity将显示在这个Task的顶端；如果没有的话，系统将会尝试为这个Activity创建一个新的Task。需要注意的是，如果一个Activity在manifest文件中声明的启动模式是”singleTask”，那么他被启动的时候，行为模式会和前面提到的指定FLAG_ACTIVITY_NEW_TASK一样。 
　　那么，有了上面的知识，我们应该可以实现开头提到的功能了。

####功能的实现
首先,在mainifest中配置我们的Activity，

```
<activity
        android:name="com.test.TestActivity"
        android:configChanges="orientation|keyboard|keyboardHidden"
        android:exported="true"
        android:taskAffinity="com.test.TestActivity"
        android:screenOrientation="portrait"/>
        
NotificationManager mNotifManager = (NotificationManager) mContext.getSystemService(Context.NOTIFICATION_SERVICE);
  Notification notification = new Notification();
  notification.icon = R.drawable.icon;
  notification.flags = Notification.FLAG_ONGOING_EVENT;
  notification.flags = Notification.FLAG_AUTO_CANCEL;
  notification.flags = Notification.FLAG_NO_CLEAR;
  RemoteViews mContentView = new RemoteViews(mContext.getPackageName(), 
  R.layout.notification_test);
  notification.contentView = mContentView;

  Intent intent = new Intent();
  intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
  intent.setClass(mContext, TestActivity.class);
  PendingIntent pendingIntent =PendingIntent.getActivity(mContext, 0, intent, 
  PendingIntent.FLAG_UPDATE_CURRENT);                     
     notification.contentView.setOnClickPendingIntent(R.id.rl_notification, 
  pendingIntent);

  mNotifManager.notify(NOTIFYID, notification); 

```

现在我们可以实现开头提到的那种效果了。但是，我发现最近任务中会有两个我们应用的图标，看起来像是启动了两个我们的应用，非常奇怪，而实际上是因为我们的应用启动了两个Task。我们可以通过在manifest此Activity的属性中增加Android:excludeFromRecents=”true”。这属性用于设置由该Activity所启动的任务是否应该被排除在最近使用的应用程序列表之外。也就是说，当这个Activity是一个新任务的根节点时，这个属性决定了这个任务是否会显示在用户最近使用的应用程序列表中。如果设置为true，则这个任务会被排除在列表之外，为false，则表示会包含在最近使用的应用列表中。默认值是false。

####场景验证
前提：将页面背景设置为半透明。
 
* 1、未使用affinity 
如果应用没有启动，点击通知栏，当前Activity被调起，透明背景后显示为桌面。 
如果应用已经启动，点击通知栏，应用的Task调到前台，当前Activy显示在该Task的顶端。 

* 2、使用affinity 
无论应用是否已经启动，点击通知栏，当前Activity都会被调起，透明背景后显示为桌面。 
两种情况下，看最近任务栏，都只显示一个应用图标。
 
这样就实现了我们想要的效果。

####TaskAffinity属性小结

最近在项目中用到了TaskAffinity属性，发现这个还是挺有意思，可以用来控制activity所属的任务栈。但同时只设置这一个属性又是不能完成功能的，需要与其它属性相配合。

* 一.通过配置方式来实现TaskAffinity来实现

上边说到要想使TaskAffinity属性生效，要与其它属性相配合。在配置文件中，需要设置activity的启动模式为singleTask或singleInstance才能生效(其实singleInstance本来就会在新Task中)

```
<activity android:name=".bActivity"
            android:launchMode="singleTask"
            android:taskAffinity="taskName"/>
```

* 二.通过动态的方式实现TaskAffinity属性

通过上述的配置，可以实现TaskAffinity属性。但是这样每次启动该Activity都会在TaskAffinity指定的栈中启动。有时候可能会希望该activity在特殊情况下才在TaskAffinity指定的栈中启动，大部分时候还是在原有的任务栈中启动，这个时候就需要动态方式来实现TaskAffinity属性。 
在配置文件中，只制定TaskAffinity属性，而不制定launchMode的属性为singleTask。

```
<activity android:name=".bActivity"
android:taskAffinity="taskName"/>
```
这样通过正常方式启动该Activity时，该Activity就会在原有任务栈中启动（启动该Activity的任务栈中）。若想在taskAffinity属性生效，需要在启动该Activity时设置Flag为FLAG_ACTIVITY_NEW_TASK。

```
Intent intent = new Intent(aAvtivity.this, bActivity.class);
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);
```
当TaskAffinity生效时，如已经存在相应名称的任务栈，则不会新建栈，而是在该栈的栈顶建立相应activity；如果没有相应名称的任务栈，就会建立对应名称的新的任务栈。

* 另外说明一点，setFlags和addFlags的区别是：setFlags会直接将原来的Flag直接替换掉；而addFlags是将参数添加上去。

