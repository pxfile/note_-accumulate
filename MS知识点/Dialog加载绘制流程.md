Dialog加载绘制流程
===
[Android源码解析Window系列第（二）篇---Dialog加载绘制流程](https://www.jianshu.com/p/f9303d30eb2b)

* AlertDialog和Activity一样，内部有一个Window，我们构造AlertDialog.Builder，通过Builder设置Dialog各种属性，,这些参数会被放在一个名为P（AlertController类型）的变量中，

* 在调用AlertDialog.Builder.create方法的时候，内部首先会new一个 AlertDialog，AlertDialog的父类Dialog的构造函数中会new一个PhoneWindow赋值给AlertDialog中的Window，并且为它设置了回调。

* AlertDialog创建之后执行apply方法，将P中的参数设置赋值给Dialog,后我们调用Dialog.show方法展示窗口，内部调用dispatchOnCreate，最终会走到setContentView，到此Dialog的Window和Dialog视图关联到了一起，

* 最后执行mWindowManager.addView方法，通过WindowManager将DecorView添加到Window之中,此时Dialog显示在了我们面前。
