---
layout:     post
title:      StatusBarUtil 状态栏工具类
subtitle:   状态栏
date:       2018-02-12
author:     pxf
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - 状态栏
    - Android
---
### StatusBarUtil 状态栏工具类
[StatusBarUtil 状态栏工具类（实现沉浸式状态栏/变色状态栏)](http://jaeger.itscoder.com/android/2016/03/27/statusbar-util.html)

测试结果：
华为荣耀 4A Android版本5.1.1 状态栏始终无法调节(微信也无法调节状态栏，是华为系统自己禁止了状态栏)。
小米4 可以调节

### app里集成和封装StatusBarUtil
* 1.在基类GMActivity里的gmInit方法
![gmInit方法](http://ou21vt4uz.bkt.clouddn.com/StatusBarUtil1.png)

* 2.设置有2个开关   
(1)是否需要设置statusBar 
> (有些页面是全屏显示，不需要显示statusBar，例如 欢迎页 WelcomeActivity，启动页 SplashActivity)       
 
 (2)是否头部是ImageView的界面设置状态栏
![设置状态栏](http://ou21vt4uz.bkt.clouddn.com/StatusBarUtil2.png)
> (首页，个人页，圈子详情的头部是图片)

* 3.头部是ImageView的界面设置状态栏页面，需要根据不同的UI需求做相应的设置  
> 例如首页，生成一个和状态栏高度一样背景是白色的StatusBarView，显示banner图片时StatusBarView透明度为0，当向上滑动页面到banner图完全消失显示搜索框时，StatusBarView的透明度为1，这样页面布局就不会和状态栏重叠。(基本思想都是这样，不同的是StatusBarView的显示根据页面要求不同显示也不同)
* 4.具体实现
![颜色风格](http://ou21vt4uz.bkt.clouddn.com/StatusBarUtil3.png)
> 只需传不同的颜色值就可以任意更换statusBar的颜色风格
  
![深色模式](http://ou21vt4uz.bkt.clouddn.com/StatusBarUtil4.png)   
> 由于状态栏是纯白色的，所以状态栏字体必须是深色模式。
 
![5.0版本的适配](http://ou21vt4uz.bkt.clouddn.com/StatusBarUtil5.png) 
> 非小米，非魅族并且是5.0版本的适配

![5.0版本的适配](http://ou21vt4uz.bkt.clouddn.com/StatusBarUtil6.png)
  
* 4.设置深色状态栏字体模式   
> 很多国内三方Android系统都有深色状态栏字体模式，但是目前只看到了小米和魅族公开了各自的实现方法，支持底层Android4.4以上的版本。而Android官方在6.0版本才有了深色状态栏字体API。
所以Android4.4以上系统版本可以修改状态栏颜色，但是只有小米的MIUI、魅族的Flyme和Android6.0以上系统可以把状态栏文字和图标换成深色。

 (1)设置状态栏图标为深色和魅族特定的文字风格
 ![](http://ou21vt4uz.bkt.clouddn.com/StatusBarUtil7.png) 
 
 (2)设置状态栏字体图标为深色，需要MIUIV6以上
 ![](http://ou21vt4uz.bkt.clouddn.com/StatusBarUtil8.png)
 
 (3)官方在Android6.0中提供了亮色状态栏模式，配置很简单
 ![](http://ou21vt4uz.bkt.clouddn.com/StatusBarUtil9.png)
 
* **怎么样设置图片通顶**

 * 1.复写图片通顶方法
 
 ```
 @Override     
    public boolean isImageViewStatusBar() {
        return true;
    }
    
 ```
  * 2.页面layout文件中添加一个占位View，目的是为了防止页面视图与状态栏内容重叠
 
  ```
  <View
        android:id="@+id/personal_top_view"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:background="@color/white"/>
  ```
 * 3.动态设置占位View的高度，因为不同分辨率手机状态栏高度也不同
 
  ```
  //生成一个和状态栏大小相同的矩形条
    @Bind(R.id.personal_top_view)
    public View statusBarView;
  ```
  ```
  statusBarView.getLayoutParams().height = StatusBarUtil.getStatusBarHeight(mContext); 
  ```
  以上就是最基本的图片通顶的设置了，接下来可以需要根据不同的UI需求做相应的处理。