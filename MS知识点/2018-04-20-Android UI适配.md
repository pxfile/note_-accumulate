Android UI适配
===

*   Google的官方权威适配文档
*   郭霖：[ Android官方提供的支持不同屏幕大小的全部方法](https://link.jianshu.com/?t=http://blog.csdn.net/guolin_blog/article/details/8830286)
*   Stormzhang：[Android 屏幕适配](https://link.jianshu.com/?t=http://stormzhang.com/android/2014/05/16/android-screen-adaptation/)
*   鸿洋：[Android 屏幕适配方案](https://link.jianshu.com/?t=http://blog.csdn.net/lmj623565791/article/details/45460089)
*   凯子：[ Android屏幕适配全攻略(最权威的官方适配指导)](https://link.jianshu.com/?t=http://blog.csdn.net/zhaokaiqiang1992/article/details/45419023)

## 支持各种屏幕尺寸

### 使用wrap_content、match_parent、weight

### 使用相对布局，禁用绝对布局

### 使用Size限定符

res/layout/main.xml，single-pane(默认)布局
res/layout-large/main.xml，two-pane布局

包含了large限定符，那些被定义为大屏的设备(比如7寸以上的平板)会自动加载此布局，而小屏设备会加载另一个默认的布局

**缺点**：

使用Size限定符有一个问题会让很多程序员感到头疼，large到底是指多大呢？很多应用程序都希望能够更自由地为不同屏幕设备加载不同的布局，不管它们是不是被系统认定为"large"

### 使用Smallest-width限定符

鉴于size限定符的缺陷，这就是Android为什么在3.2以后引入了"Smallest-width"限定符。

Smallest-width限定符允许你设定一个具体的最小值(以dp为单位)来指定屏幕。例如，7寸的平板最小宽度是600dp，所以如果你想让你的UI在这种屏幕上显示two pane，在更小的屏幕上显示single pane，你可以使用**sw600dp**来表示你想在600dp以上宽度的屏幕上使用two pane模式。

res/layout/main.xml，single-pane(默认)布局
res/layout-sw600dp/main.xml，two-pane布局

那些最小屏幕宽度大于600dp的设备会选择layout-sw600dp/main.xml(two-pane)布局，而更小屏幕的设备将会选择layout/main.xml(single-pane)布局

通过使用配置限定符，在运行时根据当前的设备配置自动选择合适的资源了，例如根据各种屏幕尺寸选择不同的布局。

### 使用布局别名

Smallest-width限定符仅在Android 3.2及之后的系统中有效。因而，你也需要同时使用Size限定符(small, normal, large和xlarge)来兼容更早的系统。例如，你想手机上显示single-pane界面，而在7寸平板和更大屏的设备上显示multi-pane界面，你需要提供以下文件：

*   res/layout/main.xml: single-pane布局
*   res/layout-large: multi-pane布局
*   res/layout-sw600dp: multi-pane布局

最后的两个文件是完全相同的，为了要解决这种重复，你需要使用别名技巧。例如，你可以定义以下布局：

*   res/layout/main.xml, single-pane布局
*   res/layout/main_twopanes.xml, two-pane布局

加入以下两个文件：

res/values-large/layout.xml:
```
<resources>
    <item name="main" type="layout">@layout/main_twopanes</item>
</resources>
```
res/values-sw600dp/layout.xml:
```
<resources>
    <item name="main" type="layout">@layout/main_twopanes</item>
</resources>
```
最后两个文件有着相同的内容，但是它们并没有真正去定义布局，它们仅仅只是给main定义了一个别名main_twopanes。这样两个layout.xml都只是引用了@layout/main_twopanes，就避免了重复定义布局文件的情况

### 使用屏幕方向限定符

res/values/layouts.xml

res/values-sw600dp-land/layouts.xml

res/values-sw600dp-port/layouts.xml

res/values-large-land/layouts.xml

res/values-large-port/layouts.xml

### 使用.9图

它是一种被特殊处理过的PNG图片，你可以指定哪些区域可以拉伸而哪些区域不可以

左上是拉伸区域，右下是内容区域

## 支持各种屏幕密度

### 使用非密度制约像素

由于各种屏幕的像素密度都有所不同，因此相同数量的像素在不同设备上的实际大小也有所差异，这样使用像素定义布局尺寸就会产生问题。因此，请务必使用 dp 或 sp 单位指定尺寸。dp 是一种非密度制约像素，其尺寸与 160 dpi 像素的实际尺寸相同。sp 也是一种基本单位，但它可根据用户的偏好文字大小进行调整（即尺度独立性像素），因此我们应将该测量单位用于定义文字大小。

### 提供备用位图

由于 Android 可在具有各种屏幕密度的设备上运行，因此我们提供的位图资源应始终可以满足各类普遍密度范围的要求：低密度、中等密度、高密度以及超高密度。这将有助于我们的图片在所有屏幕密度上都能得到出色的质量和效果。

要生成这些图片，我们应先提取矢量格式的原始资源，然后根据以下尺寸范围针对各密度生成相应的图片。

*   xhdpi：2.0
*   hdpi：1.5
*   mdpi：1.0（最低要求）
*   ldpi：0.75

也就是说，如果我们为 xhdpi 设备生成了 200x200 px尺寸的图片，就应该使用同一资源为 hdpi、mdpi 和 ldpi 设备分别生成 150x150、100x100 和 75x75 尺寸的图片。

然后，将生成的图片文件放在 res/ 下的相应子目录中(mdpi、hdpi、xhdpi、xxhdpi)，系统就会根据运行您应用的设备的屏幕密度自动选择合适的图片。

### 关于高清设计图尺寸

Google官方给出的高清设计图尺寸有两种方案，一种是以mdpi设计，然后对应放大得到更高分辨率的图片，另外一种则是以高分辨率作为设计大小，然后按照倍数对应缩小到小分辨率的图片。

根据经验，我更推荐第二种方法，因为小分辨率在生成高分辨率图片的时候，会出现像素丢失，我不知道是不是有方法可以阻止这种情况发生。

而分辨率可以以1280*720或者是1960*1080作为主要分辨率进行设计。

### ImageView的ScaleType属性

设置不同的ScaleType会得到不同的显示效果，一般情况下，设置为centerCrop能获得较好的适配效果。

### 动态设置

有一些情况下，我们需要动态的设置控件大小或者是位置，比如说popwindow的显示位置和偏移量等，这个时候我们可以动态的获取当前的屏幕属性，然后设置合适的数值
