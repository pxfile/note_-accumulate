ButterKnife
===
## [ButterKnife框架原理](http://bxbxbai.github.io/2016/03/12/how-butterknife-works/)

有个大神叫 Jake Wharton，开源了一个神奇的框架叫做 [ButterKnife](http://link.zhihu.com/?target=https%3A//github.com/JakeWharton/butterknife)，这个框架虽然也采用了注解进行注入，不过人家可是编译期生成代码的方式，对运行时没有任何副作用，果真见效快，疗效好，只是编译期有一点点时间成本而已。

它用了Java Annotation Processing技术，就是在Java代码编译成Java字节码的时候就已经处理了@Bind、@OnClick（ButterKnife还支持很多其他的注解）这些注解。

## Java Annotation Processing

Annotation processing 是javac中用于编译时扫描和解析Java注解的工具。Annotation processing是在编译阶段执行的，它的原理就是读入Java源代码，解析注解，然后生成新的Java代码。新生成的Java代码最后被编译成Java字节码，注解解析器（Annotation Processor）不能改变读入的Java 类，比如不能加入或删除Java方法 
下图是Java 编译代码的整个过程，可以帮助我们很好理解注解解析的过程： 
![这里写图片描述](http://img.blog.csdn.net/20160515135054791)

## ButterKnife 工作流程

当你编译你的Android工程时，ButterKnife工程中ButterKnifeProcessor类的process()方法会执行以下操作： 
开始它会扫描Java代码中所有的ButterKnife注解@Bind、@OnClick、@OnItemClicked等 
当它发现一个类中含有任何一个注解时，ButterKnifeProcessor会帮你生成一个Java类，名字类似$$ViewBinder，这个新生成的类实现了ViewBinder接口。这个ViewBinder类中包含了所有对应的代码，比如@Bind注解对应findViewById(), @OnClick对应了view.setOnClickListener()等等。最后当Activity启动ButterKnife.bind(this)执行时，ButterKnife会去加载对应的ViewBinder类调用它们的bind()方法。

比如：

```
class ExampleActivity extends Activity {
     @Bind(R.id.user) EditText username;
     @Bind(R.id.pass) EditText password;

    @Override public void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
         setContentView(R.layout.simple_activity);
         ButterKnife.bind(this);
         // TODO Use fields…
     }

     @OnClick(R.id.submit) void submit() {
     // TODO call server…
     }
}
```

编译成功后，下面的代码生成了：

```
public class ExampleActivity$$ViewBinder<T extends 
        io.bxbxbai.samples.ui.ExampleActivity> implements ViewBinder<T> {

     @Override public void bind(final Finder finder, final T target, Object source) {
          View view;
          view = finder.findRequiredView(source, 21313618, “field ‘user’”);
          target.username = finder.castView(view, 21313618, “field ‘user’”);
          view = finder.findRequiredView(source, 21313618, “field ‘pass’”);
          target.password = finder.castView(view, 21313618, “field ‘pass’”);
          view = finder.findRequiredView(source, 21313618, “field ‘submit’ and method ‘submit’”);
          view.setOnClickListener(
            new butterknife.internal.DebouncingOnClickListener() {
               @Override public void doClick(android.view.View p0) {
      target.submit();
           }
        });
      }

     @Override public void reset(T target) {
           target.username = null;
           target.password = null;
     }
}
```

用一张图来说明一下： 
![这里写图片描述](http://img.blog.csdn.net/20160515140100065)

## ButterKnife.bind 执行阶段

最后，执行bind方法时，我们会调用ButterKnife.bind(this)： 
ButterKnife会调用findViewBinderForClass(targetClass)加载ExampleActivity$$ViewBinder.java类。然后调用ViewBinder的bind方法，动态注入ExampleActivity类中所有的View属性。 
如果Activity中有@OnClick注解的方法，ButterKnife会在ViewBinder类中给View设置onClickListener，并且将@OnClick注解的方法传入其中。

在上面的过程中可以看到，为什么你用@Bind、@OnClick等注解标注的属性或方法必须是public或protected的，因为ButterKnife是通过ExampleActivity.this.editText来注入View的。因为如果你把View设置成private，那么框架必须通过反射来注入View，不管现在手机的CPU处理器变得多快，如果有些操作会影响性能，那么是肯定要避免的，这就是ButterKnife与其他注入框架的不同。

## ButterKnife的特点

方便的处理View的绑定和点击事件 
方便的处理ListView/RecycleView中ViewHolder的绑定事件 
增强代码可读性

eg： 
public class FancyFragment extends Fragment { 
@Bind(R.id.button1) Button button1; 
@Bind(R.id.button2) Button button2;

```
    @Override 
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState){
      View view = inflater.inflate(R.layout.fancy_fragment, container, false);
      ButterKnife.bind(this, view);
      // TODO Use fields...
      return view;
    }

```
}

## 有一点需要注意

通过ButterKnife来注入View时，ButterKnife有`bind(Object, View)` 和 `bind(View)`两个方法，有什么区别呢？

如果你自定义了一个View，比如`public class BadgeLayout extends Fragment`，那么你可以可以通过`ButterKnife.bind(BadgeLayout)`来注入View的

如果你在一个ViewHolder中inflate了一个xml布局文件，得到一个`View`对象，并且这个View是`LinearLayout`或`FrameLayout`等系统自带View，那么不是不能用`ButterKnife.bind(View)`来注入View的，因为ButterKnife认为这些类的包名以`com.android`开头的类是没有注解功能的（-。- 这不是废话吗？），所以这种情况你需要使用`ButterKnife.bind(ViewHolder，View)`来注入View。

这表示**你是把`@Bind`、`@OnClick`等注解写到了这个ViewHolder类中，ViewHolder中的View呢需要从后面那个`View`中去找**， 大概就是这么个意思
