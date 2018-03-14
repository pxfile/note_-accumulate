###自定义View和ViewGroup相关


**ViewGroup的职能为：**  

给childView计算出建议的宽和高和测量模式 ；决定childView的位置；为什么只是建议的宽和高，而不是直接确定呢，别忘了childView宽和高可以设置为wrap_content，这样只有childView才能计算出自己的宽和高。

**View的职责：**

根据测量模式和ViewGroup给出的建议的宽和高，计算出自己的宽和高；同时还有个更重要的职责是：在ViewGroup为其指定的区域内绘制自己的形态。

在自定义view中：

1、自定义View的属性

　　通过attrs.xml 添加

```
<declare-styleable name="CustomTextView">                                  
  
  <attr name="titleText" /> 
   
  <attr name="titleTextColor" /> 
   
  <attr name="titleTextSize" /> 
   
</declare-styleable> 
```

2、在View的构造方法中获得我们自定义的属性

　　通过Context.obtainStyledAttributes获取xml属性。

3、重写onMesure 

　　设置了WRAP_CONTENT时，我们需要自己进行测量

4、重写onDraw

>*********主要View的执行过程*********:

* （1）构造方法 
* （2）onFinishInflate 
* （3）onSizeChanged
* （4）onDraw

在自定义ViewGroup中：

1. onMeasure中计算childView的测量值以及模式，以及设置自己的宽和高

2. onLayout对其所有childView进行定位（设置childView的绘制区域）

　　通过getChildCount()获取总子view，getChildAt获取childview调用各自的layout(int, int, int, int)方法。

ViewGroup不会执行onDraw说明：

1）ViewGroup默认情况下，会被设置成WILL_NOT_DRAW，这是从性能考虑，这样一来，onDraw就不会被调用了。

2）如果我们要重要一个ViweGroup的onDraw方法，有两种方法：

        1，在构造函数里面，给其设置一个颜色，如#00000000。

        2，在构造函数里面，调用setWillNotDraw(false)，去掉其WILL_NOT_DRAW flag。

**Android中实现view的更新有两组方法**

一组是invalidate，另一组是postInvalidate，其中前者是在UI线程自身中使用，而后者在非UI线程中使用。 

前面要利用Handler结合使用和利用postInvalidate()来实现在线程中刷新界面。 

* 1，利用invalidate()刷新界面 
　　实例化一个Handler对象，并重写handleMessage方法调用invalidate()实现界面刷新;而在线程中通过sendMessage发送界面更新消息。 

* 2，使用postInvalidate()刷新界面 
　　使用postInvalidate则比较简单，不需要handler，直接在线程中调用postInvalidate即可。 (*****源码也是通过handler去执行*****)

