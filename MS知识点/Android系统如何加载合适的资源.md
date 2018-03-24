Android系统如何加载合适的资源
===
 

[Android源码分析-资源加载机制](http://blog.csdn.net/singwhatiwanna/article/details/24532419)

在activity内部访问资源（字符串，图片等）是很简单的，只要getResources然后就可以得到Resources对象，有了Resources对象就可以访问各种资源了。

## 不同的Context得到的都是同一份资源。

得到资源的方式为context.getResources，而真正的实现位于ContextImpl中的getResources方法，在ContextImpl中有一个成员 private Resources mResources，它就是getResources方法返回的结果，mResources的赋值代码为：
```
mResources = mResourcesManager.getTopLevelResources(mPackageInfo.getResDir(),
Display.DEFAULT_DISPLAY, null, compatInfo, activityToken);
```
下面看一下ResourcesManager的getTopLevelResources方法，这个方法的思想是这样的：在ResourcesManager中，所有的资源对象都被存储在ArrayMap中，首先根据当前的请求参数去查找资源，如果找到了就返回，否则就创建一个资源对象放到ArrayMap中。

## 有一点需要说明的是为什么会有多个资源对象.

原因很简单，因为res下可能存在多个适配不同设备、不同分辨率、不同系统版本的目录，按照android系统的设计，不同设备在访问同一个应用的时候访问的资源可以不同，比如drawable-hdpi和drawable-xhdpi就是典型的例子。

## ResourcesManager采用单例模式，这样就保证了不同的ContextImpl访问的是同一套资源.

注意，这里说的同一套资源未必是同一个资源，因为资源可能位于不同的目录，但它一定是我们的应用的资源，或许这样来描述更准确，在设备参数和显示参数不变的情况下，不同的ContextImpl访问到的是同一份资源。设备参数不变是指手机的屏幕和android版本不变，显示参数不变是指手机的分辨率和横竖屏状态。也就是说，尽管Application、Activity、Service都有自己的ContextImpl，并且每个ContextImpl都有自己的mResources成员，但是由于它们的mResources成员都来自于唯一的ResourcesManager实例，所以它们看似不同的mResources其实都指向的是同一块内存(C语言的概念)，因此，它们的mResources都是同一个对象（在设备参数和显示参数不变的情况下）。

在横竖屏切换的情况下且应用中为横竖屏状态提供了不同的资源，处在横屏状态下的ContextImpl和处在竖屏状态下的ContextImpl访问的资源不是同一个资源对象。

assets.addAssetPath(resDir)这句话的意思是把资源目录里的资源都加载到AssetManager对象中，具体的实现在jni中，大家感兴趣自己去了解下。而资源目录就是我们的res目录，当然resDir可以是一个目录也可以是一个zip文件。有没有想过，如果我们把一个未安装的apk的路径传给这个方法，那么apk中的资源是不是就被加载到AssetManager对象里面了呢？事实证明，的确是这样，具体情况可以参见[Android apk动态加载机制的研究（二）：资源加载和activity生命周期管理](http://blog.csdn.net/singwhatiwanna/article/details/23387079)这篇文章。addAssetPath方法的定义如下，注意到它的注释里面有一个{@hide}关键字，这意味着即使它是public的，但是外界仍然无法访问它，因为android sdk导出的时候会自动忽略隐藏的api，因此只能通过反射来调用。
```
/** 
 * Add an additional set of assets to the asset manager.  This can be 
 * either a directory or ZIP file.  Not for use by applications.  Returns 
 * the cookie of the added asset, or 0 on failure. 
 * {@hide} 
 */  
public final int addAssetPath(String path) {  
    int res = addAssetPathNative(path);  
    return res;  
}  
```
有了AssetManager对象后，我们就可以创建自己的Resources对象了，代码如下:

```
try {  
    AssetManager assetManager = AssetManager.class.newInstance();  
    Method addAssetPath = assetManager.getClass().getMethod("addAssetPath", String.class);  
    addAssetPath.invoke(assetManager, mDexPath);  
    mAssetManager = assetManager;  
} catch (Exception e) {  
    e.printStackTrace();  
}  
Resources currentRes = this.getResources();  
mResources = new Resources(mAssetManager, currentRes.getDisplayMetrics(),  
        currentRes.getConfiguration());
```
有了Resources对象，我们就可以通过Resources对象来访问里面的各种资源了，通过这种方法，我们可以完成一些特殊的功能，比如换肤、换语言包、动态加载apk等.
