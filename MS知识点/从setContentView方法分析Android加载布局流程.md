从setContentView方法分析Android加载布局流程
===


[从setContentView方法分析Android加载布局流程](http://blog.csdn.net/feidu804677682/article/details/46711921)

流程： 
Activity setContentView—>Window setContentView—>PhoneWindow setContentView—->PhoneWindow installDecor—–>PhoneWindow generateLayout——>PhoneWindow mLayoutInflater.inflate(layoutResID, mContentParent);

Activity 类中有一个Window抽象类的实现PhoneWindow类，该类中有个内部类DecorView，继承自FrameLayout，在DecorView容器中添加了根布局，根布局中包含了一个id为 contnet的FrameLayout 内容布局，我们的Activity加载的布局xml最后添加到 id为content的FrameLayout布局当中了。用一个图来描述，如下：

![这里写图片描述](http://img.blog.csdn.net/20150702153425952)

总结：

1.关于requestWindowFeature(Window.FEATURE_NO_TITLE); 去除标题栏的疑问，如果你自己的xxxActivity是继承自Activity，那么恭喜你使用以上方法可以去除标题栏，如果你自己的xxxActivity是继承自AppCompatActivity或者ActionBarActivity，那么很遗憾告诉你，此次系统默认的标题栏已经在主题中去除，此时显示的标题栏是ActionBar导航栏，如果需要去除导航栏，你可以通过如下代码：getSupportActionBar().hide();来隐藏导航栏。

2.requestWindowFeature(Window.FEATURE_NO_TITLE);方法需要在 setContentView方法之前使用，由上面 Step5分析可得，设置Activity Window 特征是在setContentView方法中设置的，因此，如果需要改变Activity Window窗口特征，需要在setContentView方法之前。其实这里有疑问？？？为什么设置全屏的方法

```
getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);
```

可以在setContentView之后呢？？？求解。
