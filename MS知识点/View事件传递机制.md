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