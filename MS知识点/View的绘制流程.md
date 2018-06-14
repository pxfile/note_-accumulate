View的绘制流程
===
![](https://img-blog.csdn.net/20170123001028935?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* [Android进阶——Android视图工作机制之measure、layout、draw](http://blog.csdn.net/qq_30379689/article/details/54588736)

* [Android View 测量流程(Measure)完全解析](https://www.jianshu.com/p/3299c3de0b7d)

* [Android View 绘制流程 源码解析](https://www.jianshu.com/p/bb7977990baa)

View的绘制流程：OnMeasure()——>OnLayout()——>OnDraw()

各步骤的主要工作：

**_OnMeasure()：_**

    测量视图大小。从顶层父View到子View递归调用measure方法，measure方法又回调OnMeasure。

**_OnLayout()：_**

    确定View位置，进行页面布局。从顶层父View向子View的递归调用view.layout方法的过程，即父View根据上一步measure子View所得到的布局大小和布局参数，将子View放在合适的位置上。

**_OnDraw()：_**

    绘制视图:ViewRoot创建一个Canvas对象，然后调用OnDraw()。六个步骤：①、绘制视图的背景；②、保存画布的图层（Layer）；③、绘制View的内容；④、绘制View子视图，如果没有就不用；⑤、还原图层（Layer）；⑥、绘制滚动条。

![View绘制流程函数调用链](http://ou21vt4uz.bkt.clouddn.com/interview/custom_view/flow_img/view_measure.png)


**如何自定义控件**：

1.  自定义属性的声明和获取

    *   分析需要的自定义属性
    *   在res/values/attrs.xml定义声明
    *   在layout文件中进行使用
    *   在View的构造方法中进行获取
2.  测量onMeasure

3.  布局onLayout(ViewGroup)

4.  绘制onDraw

5.  onTouchEvent

6.  onInterceptTouchEvent(ViewGroup)

7.  状态的恢复与保存

## canvas的简单使用

此三者一般会在自定义view的onDraw()中用到：

**canvas**：决定view的布局（位置，画布颜色，形状）

**paint**：决定view的属性（颜色，字体大小，风格）

**path**：路径（path的用法深入比较复杂，此处由于是入门，就不多加阐述混淆新手了）

**大致绘制步骤：**

首先定义paint的属性，然后获取到view的宽高（一般绘制会根据view的大小来适配），随后根据获取到的view的大小使用canvas来绘制出形状。

*   **drawXxx方法族**：以一定的坐标值在当前画图区域画图，另外图层会叠加， 即后面绘画的图层会覆盖前面绘画的图层。drawCircle，drawRect,drawColor,drawBitmap,**drawPath**用的很少

*   **clipXXX方法族**：在当前的画图区域裁剪(clip)出一个新的画图区域，这个 画图区域就是canvas对象的当前画图区域了。比如：clipRect(new Rect())， 那么该矩形区域就是canvas的当前画图区域

*   **getXxx方法族**：获得与Canvas相关一些值，比如宽高，屏幕密度等。

*   **save**()，**restore**()，**saveLayer**()，**restoreToCount**()等保存恢复图层的方法

*   **translate**(平移)，**scale**(缩放)，**rotate**(旋转)，**skew**(倾斜)
```
protected void onDraw(Canvas canvas) {
         super.onDraw(canvas);
          //把整张画布绘制成白色
            canvas.drawColor(Color.WHITE);
            Paint paint = new Paint();
            //去锯齿
            paint.setAntiAlias(true);
            paint.setColor(Color.BLUE);
            paint.setStyle(Paint.Style.STROKE);
                //填充
               //paint.setStyle(Paint.Style.FILL);
               //-----------------设置渐变后绘制------------------
            //为Paint设置渐变器
            Shader mShader = new LinearGradient(0,0,40,60,
                    new int[]{
                    Color.RED,Color.GREEN,Color.BLUE,Color.YELLOW},
                    null,Shader.TileMode.REPEAT);
            //为Paint设置渐变器
            paint.setShader(mShader);
            //设置阴影
            paint.setShadowLayer(45, 10, 10, Color.GRAY);
            paint.setStrokeWidth(3);
            //绘制圆形
            canvas.drawCircle(40, 40, 30, paint);
            //绘制正方形
            canvas.drawRect(10,80,70,140, paint);
            //绘制矩形
            canvas.drawRect(10,150,70,190, paint);
            RectF re1 = new RectF(10,200,70,230);
            //绘制圆角矩形
            canvas.drawRoundRect(re1, 15, 15, paint);
            RectF rel1 = new RectF(10,240,70,270);
            //绘制椭圆
            canvas.drawOval(rel1, paint);
            //定义一个Path对象,封闭成一个三角形
            Path path1 = new Path();
            path1.moveTo(10, 340);
            path1.lineTo(70, 340);
            path1.lineTo(40, 290);
            path1.close();
            //根据path进行绘制,绘制三角形
            canvas.drawPath(path1,paint); 
 
            //根据path进行绘制,封闭成一个五角形
            Path path2 = new Path();
            path2.moveTo(26, 360);
            path2.lineTo(54, 360);
            path2.lineTo(70, 392);
            path2.lineTo(40, 420);
            path2.lineTo(10, 392);
            path2.close();
            //根据path进行绘制,绘制五角形
            canvas.drawPath(path2,paint);
```