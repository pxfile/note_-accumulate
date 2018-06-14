View事件传递机制
===
[事件分发机制](https://www.jianshu.com/p/38015afcdb58)

1).Android事件分发机制的本质是要解决：点击事件由哪个对象发出，经过哪些对象，最终达到哪个对象并最终得到处理。这里的对象是指Activity、ViewGroup、View.

2).Android中事件分发顺序：Activity（Window） -> ViewGroup -> View.

3).事件分发过程由dispatchTouchEvent() 、onInterceptTouchEvent()和onTouchEvent()三个方法协助完成

**设置Button按钮来响应点击事件事件传递情况：（如下图）**

布局如下:

![onInterceptTouchEvent](http://upload-images.jianshu.io/upload_images/4642697-01f23a099103eb82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/210)

最外层：Activiy A，包含两个子View：ViewGroup B、View C

中间层：ViewGroup B，包含一个子View：View C

最内层：View C

假设用户首先触摸到屏幕上View C上的某个点（如图中黄色区域），那么Action_DOWN事件就在该点产生，然后用户移动手指并最后离开屏幕。

**按钮点击事件:**

DOWN事件被传递给C的onTouchEvent方法，该方法返回true，表示处理这个事件;

因为C正在处理这个事件，那么DOWN事件将不再往上传递给B和A的onTouchEvent()；

该事件列的其他事件（Move、Up）也将传递给C的onTouchEvent();

![](http://upload-images.jianshu.io/upload_images/4642697-ca12c00c79780b57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

![onTouchEvent](https://upload-images.jianshu.io/upload_images/944365-aea821bbb613c195.png)

(记住这个图的传递顺序,面试的时候能够画出来,就很详细了)

# 安卓OnTouchListener，onTouchEvent，onClickListener执行顺序

log信息为 
![这里写图片描述](https://img-blog.csdn.net/20170803194104919?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveHcxMzc4MjUxMzYyMQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

*   [事件分发机制](http://www.jianshu.com/p/e99b5e8bd67b)

可见是首先执行OnTouchListener，之后为onTouchEvent，最后才执行onClickListener内的方法，至于为什么OnTouchListener和onTouchEvent执行了两次，是因为在DOWN和UP时两个方法都被调用，至于onClickListener则只在UP的时候调用

对于在onTouchEvent消费事件的情况：**在哪个View的onTouchEvent 返回true，那么ACTION_MOVE和ACTION_UP的事件从上往下传到这个View后就不再往下传递了，而直接传给自己的onTouchEvent 并结束本次事件传递过程。**

对于ACTION_MOVE、ACTION_UP总结：**ACTION_DOWN事件在哪个控件消费了（return true）， 那么ACTION_MOVE和ACTION_UP就会从上往下（通过dispatchTouchEvent）做事件分发往下传，就只会传到这个控件，不会继续往下传，如果ACTION_DOWN事件是在dispatchTouchEvent消费，那么事件到此为止停止传递，如果ACTION_DOWN事件是在onTouchEvent消费的，那么会把ACTION_MOVE或ACTION_UP事件传给该控件的onTouchEvent处理并结束传递。**

作者：Kelin
链接：https://www.jianshu.com/p/e99b5e8bd67b
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
