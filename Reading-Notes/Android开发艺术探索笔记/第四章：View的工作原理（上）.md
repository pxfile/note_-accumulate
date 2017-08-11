#Android艺术开发探索第四章——View的工作原理（上）
---
>这章就比较好玩了，主要介绍一下View的工作原理，还有自定义View的实现方法，在Android中，View是一个很重要的角色，简单来说，View是Android中视觉的呈现，在界面上Android提供了一套完整的GUI库，里面有很多控件，但是有时候往往并不能满足于需求，所以只有自定义View了，我们会简单的说下流程，然后再去实践

>除了View的三大流程之外，View常见的回调方法也是必须掌握的，比如构造方法，onAttach,onVisibilityChanged，onDetach，另外对于一些有滑动效果的自定义View，还要处理滑动事件和滑动冲突，总的来说，自定义View有几种固定的类型，View或者ViewGroup，有的直接重写原生控件，这个就要看需求了，好的，我们直接开始吧！

##一.初识ViewRoot和DecorView
>在正式介绍View的三大流程之前，我们还是要了解一些基本的概念，所以本章会说下ViewRoot和DecorView

>ViewRoot对应于ViewRootImpl类，他是连接WindowManager和DecorView的纽带，View的三大流程都是通过ViewRoot来完成的，在ActivityThread中，当Activity被创建完毕后，会将DecorView添加到Window值班费，同时会创建ViewRootImpl对象，并将ViewRootImpl对象和DecorView建立联系，这个可以参照官网：

```
root = new ViewRootImpl(view.getContext(),display);
root.setView(view,wparams,panelParentView);
```

>View的绘制流程从ViewRoot的perfromTraversals方法开始，他警告measure，layout和draw三个过程才能将View画出来，，其中measure测量，layout确定view在容器的位置，draw开始绘制在屏幕上，针对perfromTraversals的大致流程，可以看图


--------------------------------------------------------------图

>图中的perfromTraversals会依次调用perfromMeasure，perfromLayout，perfromDraw，他们分别完成顶级View的measure，layout和draw这三大流程，其中在perfromMeasure中会调用measure方法，在measure方法中又调用onMeasure,这个时候measure流程就从父容器传递到子元素了，这样就完成了一次measure过程，接着子元素会重复父容器的measure过程，如此反复的完成了整个View树的遍历，同理，其他两个也是如此，唯一有点区别的是perfromDraw的传递过程是在draw反复中通过dispatchDraw来实现的，不过这并没有什么本质的区别

>measure過程决定了View的宽高，Measure完成之后可以通过getMeasureWidth和getMeasureHeight来获取View测量后的高宽，在所有的情况下塔几乎都是等于最终的宽高，但是特殊情况除外，这点以后说，layout过程决定了view的四个顶点的坐标和实际View的宽高，完成之后，通过getTop，getLeft，getRight,getBottom获得，，Draw决定了View的显示，只有draw方法完成了之后，view才会显示在屏幕上

>如下图，顶级View DecorView，一般情况下他内部会包含一个竖直方向的LinearLayout，这里面有上下两部分，上面是标题栏，下面是内容，在Activity中，我们可用通过setContentView设置布局文件就是放在内容里，而内容栏的id为content,因此我們可以理解为实际上是在setView，那如何得到content呢？你可以ViewGroup content = findviewbyid(android.R.id.content),如何得到我们设置的View呢：content.getChildAt(0),同时，通过源码我们可用知道，DeaorView其实就是一个FrameLayout，View层事件都先经过DecorView，然后传递给View

--------------------------------------------------------------图

##二.理解MeasureSpec
>为了更好的理解View的测量过程，我们还需要理解MeasureSpec，从名字上看，MeasureSpec看起来像“测量规格”或者“测量说明书”，不管怎么翻译，他看起来就好像是或多或少的决定了View的测量过程，通过源码可以发现，MeasureSpec的确参与了View的测量过程，读者可能有疑问，MeasureSpec是干什么的呢？MeasureSpec在很大程度上决定了一个View的尺寸规格，之所以说很大程度上是因为这个过程还收到了父容器的影像，因为父容器影像MeasureSpec的创建过程，在测量过程中，系统会将View的LayoutParams根据父容器所施加的规则转换成对应的MeasureSpec，然后再根据这个measureSpec来测量出View的宽高，MeasureSpec看起来有点复杂，其实他的实现很简单，我们来详细分解一下

###1.MeasureSpec
>MeasureSpec代表一个32位int值，高两位代表SpecMode，低30位代表SpecSize，SpecMode是指测量模式，而SpecSize是指在某个测量模式下的规格大小，下面先看一下，MeasureSpec内部的一些常量定义，通过这些就不难理解MeasureSpec的工作原理了

```
private static final int MODE_SHIFT = 30;
private static final int MODE_MASK  = 0x3 << MODE_SHIFT;
public static final int UNSPECIFIED = 0 << MODE_SHIFT;
public static final int EXACTLY     = 1 << MODE_SHIFT;
public static final int AT_MOST     = 2 << MODE_SHIFT;

	 public static int makeMeasureSpec(@IntRange(from = 0, to = (1 << MeasureSpec.MODE_SHIFT) - 1) int size,
                                          @MeasureSpecMode int mode) {
            if (sUseBrokenMakeMeasureSpec) {
                return size + mode;
            } else {
                return (size & ~MODE_MASK) | (mode & MODE_MASK);
            }
        }
        
        @MeasureSpecMode
        public static int getMode(int measureSpec) {
            //noinspection ResourceType
            return (measureSpec & MODE_MASK);
        }
        
        public static int getSize(int measureSpec) {
            return (measureSpec & ~MODE_MASK);
        }
```

>MeasureSpec通过将SpecMode和SpecSize打包成一个int值来避免过多的对象内存分配，为了方便操作，其提供了打包和解包的作用，SpecMode和specSize也是一个int值，一直SpecMode和specSize可以打包成一个MeasureSpec，一个MeasureSpec可以通过解包的形式来得出其原始的SpecMode和SpecSize，需要注意的是这里提到的MeasureSpec是指MeasureSpec所代表的int值，而非MeasureSpec本身。

>SpecMode有三类，每一类都有特殊的含义

- UNSPECIFIED

>父容器不对View有任何的限制，要多大给多大，这种情况一般用于系统内部，表示一种测量的状态

- EXACTLY

>父容器已经检测出View所需要的精度大小，这个时候View的最终大小就是SpecSize所指定的值，它对应于LayoutParams中的match_parent,和具体的数值这两种模式

- AT_MOST

>父容器指定了一个可用大小，即SpecSize，view的大小不能大于这个值，具体是什么值要看不同view的具体实现，它对应于LayoutParams中wrap_content

###2.MeasureSpec 和 LayoutParams 的对应关系
>系统内部是通过MeasureSpec来进行View的测量，但是正常情况下我们使用View的测量，但是正常情况下我们使用View指定MeasureSpec，但是尽管如此，我们也可以给View设置layoutparams，在view测量的时候，系统会将layoutparams在父容器的约束下转换成对应的MeasureSpec，然后再根据这个MeasureSpec来确定view测量后的宽高，需要注意的是，MeasureSpec不是唯一由layoutparams决定的，layoutparams需要和父容器一起决定view的MeasureSpec从而进一步决定view的宽高，对于顶级view（DecorView）和普通的view来说，MeasureSpec的转换过程有些不同，对于decorview，其MeasureSpec由父容器的MeasureSpec和自身的layoutparams来决定，MeasureSpec一旦确定后，MeasureSpec就可以去为view测量了

>对于DecorView来说，在ViewRootImpl中的measureHierarchy方法中有这么一段代码。他展示了DecorViwew的MeasureSpec创建过程，其中desiredWindowWidth和desiredWindowHeight是屏幕的尺寸

```
 childWidthMeasureSpec = getRootMeasureSpec(baseSize, lp.width);
 childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
 performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
```

>接下来看下getRootMeasureSpec方法的实现：

```
private static int getRootMeasureSpec(int windowSize, int rootDimension) {
        int measureSpec;
        switch (rootDimension) {

        case ViewGroup.LayoutParams.MATCH_PARENT:
            // Window can't resize. Force root view to be windowSize.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
            break;
        case ViewGroup.LayoutParams.WRAP_CONTENT:
            // Window can resize. Set max size for root view.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
            break;
        default:
            // Window wants to be an exact size. Force root view to be that size.
            measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
            break;
        }
        return measureSpec;
    }

```

>通过上述代码，DecorView的MesourSpec的产生过程就很明确了，具体来说其遵守了如下格式，根据layoutparams的宽高的参数来划分

- LayouParams.MATCH_PARENT:精确模式，大小就是窗口的大小
- LayouParams.WRAP_CONTENT:最大模式，大小不定，但是不能超出屏幕的大小
- 固定大小（比如100dp）:精确模式，大小为LayoutParams中指定的大小

>对于普通的View来说，这里是指我们布局中的View,View的measure过程由ViewGroup传递而来，先看下ViewGroup的measureChildWithMargis方法

```
	 protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

>上述的方法会对子元素进行measure，在调用子元素的measure方法之前会通过getChildMeasureSpec方法得到子元素的MesureSpec，从代码上看，很显然，子元素的MesureSpec的创建和父容器的MesureSpec和子元素的LayoutParams有关，此外，还和view的margin有关，具体可以看下ViewGroup的getChildMeasureSpec方法

```
	public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        //noinspection ResourceType
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }

```

>上述方法不难理解，他的主要作用是根据父容器的MeasureSpec同时结合view本身来layoutparams来确定子元素的MesureSpec，参数中的pading是指父容器中已占有的控件大小，因此子元素可以用的大小为父容器的尺寸减去pading，具体代码

```
int specSize = MesureSpec.getSize(spec);
int size = Math.max(0,specSize - pading);

```

>getChildMeasureSpec清楚的展示了普通View的MeasureSpec同时结合View本身的LayoutParams来确定子元素的MeaureSpec的创建规则，更加清晰的理解getChildMeasureSpec的逻辑，这里提供一个表，表中对getChildMeasureSpec的工作原理进行了梳理，表中的parentSize是指父容器中目前可使用的大小：


--------------------------------------------------------------图

>针对这张表，这里再做一下说明。前面已经提到，对于普通View，其MeasureSpec 由父容器的MeasureSpec和自身的LayoutParams来共同决定，那么针对不同的父容器和Viev本身不同的LayoutParams,View就可以有多种MeasureSpec。这里简单说一下，当View采用固定宽/高的时候，不管父容器的MeasureSpec是什么，View 的MeasureSpee都是精确模式，那么View也是精准模式并且其大小是父容器的剩余空间；如果父容器是最大模式，那么View也是最大模式并且其大小不会超过父容器的剩余空间。当View的宽/高是wrap_content时，不管父容器的模式是精准还是最大化，View的模式总是最大化,并且大小不能超过父容器的剩余空间，可能读者会发现，在我们的分析中漏掉了UNSPECIFIED模式，那是因为这个模式主要用于系统内部多次Measure的情形，一般来说，我们不需要关注此模式。

>通过这张表可以看出，只要提供父容器的MeasureSpec和子元素的LayoutParams，就可以快速地确定出子元素的MeasureSpec了，有了 MeasureSpec就可以进一步确定出子元亲测量后的大小了。需要说明的是，表中并非是什么经验总结，它只是getcchildMeasureSpec
这个方法以表格的方式呈现出来而已

###3.View的工作流程
>View的工作流程主要是指measure、layout、draw这三大流程，即测量、布局和绘制，其中measure确定View的测量宽/高，layout确定View的最终宽/高和四个顶点的位置，而draww则将View绘制到屏幕上。

####1.measure过程
>measure过程要分情况来看，如果只是一个原始的View，那么通过measure方法就可以完成了其测量过程，如果是一个ViewGroup，除了完成自己的测量过程外，还会遍历去调用所有子元素的measure方法，各个子元素再递归去执行这个流程，下面针对这两种情况分别讨论

#####1.View的measure过程
Miew 的 measure过程由其measure方法来完成，measure方法是一个final类型的方法，这就意味着子类不能重写此方法，在View的measure方法中去调用View的onMesure方法，因此只需要看onMeasure的实现即可，View的onMesure方法如下所示：

```
	@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(
                getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }

```

>上面的代码很简介，但是简洁不代表简单，setMeasuredDimension会设置View宽/高的测量值，因此我们只需要getDefaultSize方法即可。

```
	public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }
```

>可以看出，getDefaultSize这个逻辑很简单，对于我们来说，我们只需要看AT_MOST和EXACTLY这两种情况，简单的理解，其实getDefaultSize返回的大小就是mesourSpec中的specSize，而这个specSize就是view的大小，这里多次提到测量后的大小，是因为View最终的大小，是在layout阶段的，所以这里必须要加以区分，但是几乎所有情况下的View的测量大小和最终大小是相等的

>至于UNSPECIFIED这种情况，一般用于系统内部的测量过程，在这种情况下，View的大小为getDefaultSize的第一个参数是size，即宽高分别为getSuggestedMinimumWidth和getSuggestedMinimumHeight()这两个方法的返回值：

```
	protected int getSuggestedMinimumWidth() {
        return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
    }
    
    protected int getSuggestedMinimumHeight() {
        return (mBackground == null) ? mMinHeight : max(mMinHeight, mBackground.getMinimumHeight());

    }

```

>这里只分析getSuggestedMinimumWidth方法的实现，getSuggestedMinimumHeight和他的原理是一样的。从 getSuggestedMinimumWidth的代码可以看出，如果View没有设置背景，View的宽度为mMinwidth，而mMinwidth对应于android:minwidth这个属性所指定的值，因此View的宽度即为android:minwidth属性所指定的值。这个属性如果不指定，那么MinWidth则默认为0；如果View指定了背景，则View的宽度为max(mMinwidth
mbackground().getMininumwidth)，mMinwidthh的含义我们已经知道了，那么mBackground.getMinimumWidth()是什么呢?我们看一下Drwable的 getMinimumWidth方法，如下所示：


```
 public int getMinimumWidth() {
        final int intrinsicWidth = getIntrinsicWidth();
        return intrinsicWidth > 0 ? intrinsicWidth : 0;
    }
```


>可以看出，getMinimumWidth返回的就是Drawable的原始宽度，前提是这个Drawable有原始宽度，否则就返回0。那么Drawable在什么情况下有原始宽度呢？这里先举个例子说明一下，ShapeDrawable无原始宽/高，而BitmapDrawable有原始宽/高(图片的尺寸)，详细内容会在第6章进行介绍。

>这里再总结一下getSuggestedMinimumWidth的逻辑：如果View没有设置背景，那么返回android:minwidth这个属性所指定的值，这个值可以为0：如果View设置了背景，则返回 android:minwidth和背景的最小宽度这两者中的最大值，getSuggestedMinimumWidth和getSuggestedMinimumHeight的返回值就是View 在UNSPECIFIED情况下的测量宽/高。

>从getDefaulSize方法的实现来看，View的宽/高由specSize决定，所以我们可以得出如下结论：直接继承View的自定义控件需要重写onMeasure方法并设置wrapcontent时的自身大小，否则在布局中使用wrap_content就相当于使用matchparent。为什么呢?这个原因需要结合上述代码和之前的表才能更好地理解。从上述代码中我们知道，如果View在布局中使用wrapcontent，那么它的specMode是AT_MOST模式，在这种模式下，它的宽/高等于 specSize；查表4-1可知，这种情况下View的specSize是parentSize,而parentSize是父容器中目前可以使用的大小，也就是父容器当前剩余的空间大小。很显然，View的宽/高就等于父容器当前剩余的空间大小，这种效果和在布局中使用match_parent完全一致。如何解决这个问题呢?也很简单，代码如下所示。

```
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSpecSize = MeasureSpec.getMode(widthMeasureSpec);
        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSpecSize = MeasureSpec.getMode(heightMeasureSpec);
        if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(mWidth, mHeight);
        } else if (widthSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(mWidth, heightSpecSize);
        } else if (eightSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(widthSpecSize, mHeight);
        }
    }

```

>在上面的代码中，我们只需要给View指定一个默认的内部宽/高(mWidth和mHeight)),并在wrapcontent时设置此宽/高即可。对于非wrapcontent情形，我们沿用系统的测量值即可，至于这个默认的内部宽/高的大小如何指定，这个没有固定的依据，根据需要灵活指定即可。如果查看TextView、Imageview等的源码就可以知道，针对 wrapcontent情形，它们的onMeasure方法均做了特殊处理，读者可以自行查看它们的源码。


#####2.ViewGroup的measure过程
>对于ViewGroup来说，除了完成自己的measure过程以外，还会遍历去调用所有子元素的measure方法，各个子元素再通归去执行这个过程。和View不同的是，ViewGroup是一个抽象类，因此它没有重写View的onMeasure方法，但是它提供了一个叫measureChildren:

```
   protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
        final int size = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < size; ++i) {
            final View child = children[i];
            if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
            }
        }
    }

```

>从上述代码中看到，在ViewGroup的measure时，会对每一个子元素进行测量，那么这个方法就很好理解了

```
 protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {
        final LayoutParams lp = child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

>很显然，measurechild的思想就是取出子元素的LayoutParams，然后再通过getChidMeasureSpec来创建子元素的MeasureSpec，接着将MeasureSpec直接传递给View的measure方法来进行测量。getchildMeasureSpec的工作过程已经在上面进行了详细分析。

>我们知道，ViewGroup并没有定义其测量的具体过程，这是因为ViewGroup是一个抽象类，其测量过程的onMeasure方法需要各个子类去具体实现，比如LinearLayout，RelativeLayout等，为什么ViewGroup不像View一样对其onMeasure方法做统一的实现呢？那是因为不同的ViewGroup子类有不同的布局特性，这导致它们的测量细节各不相同，比如Lineartayout和RelativeL.ayout这两者的布局特性显然不同，因此ViewGroup无法做统
一实现。下面就通过LinearLayout的onMeasure方法来分析ViewGroup的 measure过程，其他Layout类型读者可以自行分析。

>首先，我们来看一下LinearLayout的onMeasure方法

```
   @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (mOrientation == VERTICAL) {
            measureVertical(widthMeasureSpec, heightMeasureSpec);
        } else {
            measureHorizontal(widthMeasureSpec, heightMeasureSpec);
        }
    }
```

>上述的代码很简单我们选择一个来看下，比如选中竖直方向的LinearLayout测量过程，即measureVertical，他的源码还比较长，我们看:

```
void measureVertical(int widthMeasureSpec, int heightMeasureSpec) {
        mTotalLength = 0;
        int maxWidth = 0;
        int childState = 0;
        int alternativeMaxWidth = 0;
        int weightedMaxWidth = 0;
        boolean allFillParent = true;
        float totalWeight = 0;

        final int count = getVirtualChildCount();
        
        final int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        final int heightMode = MeasureSpec.getMode(heightMeasureSpec);

        boolean matchWidth = false;
        boolean skippedMeasure = false;

        final int baselineChildIndex = mBaselineAlignedChildIndex;        
        final boolean useLargestChild = mUseLargestChild;

        int largestChildHeight = Integer.MIN_VALUE;
        int consumedExcessSpace = 0;

        // See how tall everyone is. Also remember max width.
        for (int i = 0; i < count; ++i) {
            final View child = getVirtualChildAt(i);
            if (child == null) {
                mTotalLength += measureNullChild(i);
                continue;
            }

            if (child.getVisibility() == View.GONE) {
               i += getChildrenSkipCount(child, i);
               continue;
            }

            if (hasDividerBeforeChildAt(i)) {
                mTotalLength += mDividerHeight;
            }

            final LayoutParams lp = (LayoutParams) child.getLayoutParams();

            totalWeight += lp.weight;

            final boolean useExcessSpace = lp.height == 0 && lp.weight > 0;
            if (heightMode == MeasureSpec.EXACTLY && useExcessSpace) {
                // Optimization: don't bother measuring children who are only
                // laid out using excess space. These views will get measured
                // later if we have space to distribute.
                final int totalLength = mTotalLength;
                mTotalLength = Math.max(totalLength, totalLength + lp.topMargin + lp.bottomMargin);
```

>从上面的代码可以看出，系统会遍历子元素并对每一个子元素执行measureChildBeforeLayout方法，这个方法内部会调用子元素的measure方法，这样各个子元素就开始依次进入measure过程，并且系统通过mTotalLength这个变量来存储LinearLayout在竖直方向上的初步高度，没测量一个子元素，mTotalLength就会增加，增加的部分主要包括子元素的高度以及竖直方向上的margin等，当子元素测量完毕之后，LinearLayout会测量自己的大小，看源码:

```
// Add in our padding
        mTotalLength += mPaddingTop + mPaddingBottom;

        int heightSize = mTotalLength;

        // Check against our minimum height
        heightSize = Math.max(heightSize, getSuggestedMinimumHeight());
        
        // Reconcile our calculated size with the heightMeasureSpec
        int heightSizeAndState = resolveSizeAndState(heightSize, heightMeasureSpec, 0);
        heightSize = heightSizeAndState & MEASURED_SIZE_MASK;
        
        // Either expand children with weight to take up available space or
        // shrink them if they extend beyond our current bounds. If we skipped
        // measurement on any children, we need to measure them now.
        int remainingExcess = heightSize - mTotalLength
                + (mAllowInconsistentMeasurement ? 0 : consumedExcessSpace);
        if (skippedMeasure || remainingExcess != 0 && totalWeight > 0.0f) {
            float remainingWeightSum = mWeightSum > 0.0f ? mWeightSum : totalWeight;

            mTotalLength = 0;

            for (int i = 0; i < count; ++i) {
                final View child = getVirtualChildAt(i);
                if (child == null || child.getVisibility() == View.GONE) {
                    continue;
                }

                final LayoutParams lp = (LayoutParams) child.getLayoutParams();
                final float childWeight = lp.weight;
                if (childWeight > 0) {
                    final int share = (int) (childWeight * remainingExcess / remainingWeightSum);
                    remainingExcess -= share;
                    remainingWeightSum -= childWeight;

                    final int childHeight;
                    if (mUseLargestChild && heightMode != MeasureSpec.EXACTLY) {
                        childHeight = largestChildHeight;
                    } else if (lp.height == 0 && (!mAllowInconsistentMeasurement
                            || heightMode == MeasureSpec.EXACTLY)) {
                        // This child needs to be laid out from scratch using
                        // only its share of excess space.
                        childHeight = share;
                    } else {
                        // This child had some intrinsic height to which we
                        // need to add its share of excess space.
                        childHeight = child.getMeasuredHeight() + share;
                    }

                    final int childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(
                            Math.max(0, childHeight), MeasureSpec.EXACTLY);
                    final int childWidthMeasureSpec = getChildMeasureSpec(widthMeasureSpec,
                            mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin,
                            lp.width);
                    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);

                    // Child may now not fit in vertical dimension.
                    childState = combineMeasuredStates(childState, child.getMeasuredState()
                            & (MEASURED_STATE_MASK>>MEASURED_HEIGHT_STATE_SHIFT));
                }

                final int margin =  lp.leftMargin + lp.rightMargin;
                final int measuredWidth = child.getMeasuredWidth() + margin;
                maxWidth = Math.max(maxWidth, measuredWidth);

                boolean matchWidthLocally = widthMode != MeasureSpec.EXACTLY &&
                        lp.width == LayoutParams.MATCH_PARENT;

                alternativeMaxWidth = Math.max(alternativeMaxWidth,
                        matchWidthLocally ? margin : measuredWidth);

                allFillParent = allFillParent && lp.width == LayoutParams.MATCH_PARENT;

                final int totalLength = mTotalLength;
                mTotalLength = Math.max(totalLength, totalLength + child.getMeasuredHeight() +
                        lp.topMargin + lp.bottomMargin + getNextLocationOffset(child));
            }

            // Add in our padding
            mTotalLength += mPaddingTop + mPaddingBottom;
            // TODO: Should we recompute the heightSpec based on the new total length?
        } else {
            alternativeMaxWidth = Math.max(alternativeMaxWidth,
                                           weightedMaxWidth);


            // We have no limit, so make all weighted views as tall as the largest child.
            // Children will have already been measured once.
            if (useLargestChild && heightMode != MeasureSpec.EXACTLY) {
                for (int i = 0; i < count; i++) {
                    final View child = getVirtualChildAt(i);
                    if (child == null || child.getVisibility() == View.GONE) {
                        continue;
                    }

                    final LinearLayout.LayoutParams lp =
                            (LinearLayout.LayoutParams) child.getLayoutParams();

                    float childExtra = lp.weight;
                    if (childExtra > 0) {
                        child.measure(
                                MeasureSpec.makeMeasureSpec(child.getMeasuredWidth(),
                                        MeasureSpec.EXACTLY),
                                MeasureSpec.makeMeasureSpec(largestChildHeight,
                                        MeasureSpec.EXACTLY));
                    }
                }
            }
        }

        if (!allFillParent && widthMode != MeasureSpec.EXACTLY) {
            maxWidth = alternativeMaxWidth;
        }
        
        maxWidth += mPaddingLeft + mPaddingRight;

        // Check against our minimum width
        maxWidth = Math.max(maxWidth, getSuggestedMinimumWidth());
        
        setMeasuredDimension(resolveSizeAndState(maxWidth, widthMeasureSpec, childState),
                heightSizeAndState);

```

>这里对上述代码进行说明，当子元素测量完毕之后，LinearLayout会根据子元素的情况来测量自己的大小，针对竖直的LinearLayout而言，他的水平方向的测量过程遵循View的测量过程，在竖直方向的测量过程和View有些不同，具体来说，是指，如果他的布局中高度采用的是match_parent或者具体值，那么他的绘制过程和View一致，即高度为specSize，如果他的布局中高度采用warp_content，那么她的高度是所有的子元素所占用的高度综合，但是仍然不能超过他的父容器剩余空间，但是他的最终高度还是需要考虑其他的竖直方向上的pading，这个过程进一步参看源码:

```
 public static int resolveSizeAndState(int size, int measureSpec, int childMeasuredState) {
        final int specMode = MeasureSpec.getMode(measureSpec);
        final int specSize = MeasureSpec.getSize(measureSpec);
        final int result;
        switch (specMode) {
            case MeasureSpec.AT_MOST:
                if (specSize < size) {
                    result = specSize | MEASURED_STATE_TOO_SMALL;
                } else {
                    result = size;
                }
                break;
            case MeasureSpec.EXACTLY:
                result = specSize;
                break;
            case MeasureSpec.UNSPECIFIED:
            default:
                result = size;
        }
        return result | (childMeasuredState & MEASURED_STATE_MASK);
    }
```

>View的onMeasure是三大流程中最复杂的一个，measure完成以后，通过getMeasureWidth/Height就可以正确地获取到View的测量宽/高。需要注意的是，在某些极端情况下measure才能确定最终的测量宽/高，在这种情形下，系统可能要多次调用measure方法进行测量，在这种情况下，载onMeasure方法中拿到的测量值很可能是不准确的。一个比较好的习惯是在onLayout方法中去获取View的测量宽/高或者最终宽/高。


>上面已经对Viaw的measure过程进行了详细的分析，现在考虑一种情况，比如我们想在Activity已启动的时候就做一件任务，但是这一件任务需要获取某个View的宽/高，读者可能会说，这很简单啊，在onCreate或者onResume里面去获取这个View的宽/高就行了，读者可以自行试一下，实际上在onCreate、onStart、onResume中均无法正确得View的宽/高信息，这是因为View的measure过程和Activity的生命周期方法不是同步执行的，因此无法保证Activiy执行了onCreate、onStart、onResume时某个Vicw已经完毕了，如果View还没有测量完毕，那么获得的宽/高就是0。有没有什么方法能解决问题呢？答案是有的，这里给出四种方法来解决这个问题：

- (1)Activity/View#onWindowFocusChanged。

>onWindowFocusChanged这个方法的含义是：View已经初始化完毕了，宽/高已经准备好了，这个时候去获取宽/高是没问题的。需要注意的是，onWindowFocusChanged会被调用多次，当Activity的窗口得到焦点和失去焦点时均会被调用一次。具体来说，当Activity继续执行和暂停执行时，onWindowFocusChanged均会被调用，如果频繁地进行onResume和onPause，那么onWindowFocusChanged也会被频繁地调用。典型代码如下：

```
 public void onWindowFocusChanged(boolean hasWindowFocus) {
        InputMethodManager imm = InputMethodManager.peekInstance();
        if (!hasWindowFocus) {
            if (isPressed()) {
                setPressed(false);
            }
            if (imm != null && (mPrivateFlags & PFLAG_FOCUSED) != 0) {
                imm.focusOut(this);
            }
            removeLongPressCallback();
            removeTapCallback();
            onFocusLost();
        } else if (imm != null && (mPrivateFlags & PFLAG_FOCUSED) != 0) {
            imm.focusIn(this);
        }
        refreshDrawableState();
    }
```

- （2）view.post(runnable)

>通过post可以将一个runnable投递到消息队列，然后等到Lopper调用runnable的时候，View也就初始化好了，典型代码如下:

```
    @Override
    protected void onStart() {
        super.onStart();

        mTextView.post(new Runnable() {
            @Override
            public void run() {
                int width = mTextView.getMeasuredWidth();
                int height = mTextView.getMeasuredHeight();
            }
        });
    }
```

- (3)ViewTreeObserver

>使用ViewTreeObserver的众多回调可以完成这个功能，比如使用OnGlobalLayoutListener这个接口，当View树的状态发生改变或者View树内部的View的可见性发生改变，onGlobalLayout方法就会回调，因此这是获取View的宽高一个很好的例子，需要注意的是，伴随着View树状态的改变，这个方法也会被调用多次，典型代码如下

```
    @Override
    protected void onStart() {
        super.onStart();

        ViewTreeObserver observer = mTextView.getViewTreeObserver();
        observer.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {
                mTextView.getViewTreeObserver().removeOnGlobalLayoutListener(this);
                int width = mTextView.getMeasuredWidth();
                int height = mTextView.getMeasuredHeight();
            }
        });
    }
```

- (4)view.measure(int widthMeasureSpec , int heightMeasureSpec)

>通过手动测量View的宽高，这种方法比较复杂，这里要分情况来处理，根据View的LayoutParams来处理

- match_parent

>直接放弃，无法测量出具体的宽高，根据View的测量过程，构造这种measureSpec需要知道parentSize，即父容器的剩下空间，而这个时候我们无法知道parentSize的大小，所以理论上我们不可能测量出View的大小

- 具体的数值

>比如宽高都是100dp，那我们可以这样:

```
        int widthMeasureSpec = View.MeasureSpec.makeMeasureSpec(100, View.MeasureSpec.EXACTLY);
        int heightMeasureSpec = View.MeasureSpec.makeMeasureSpec(100, View.MeasureSpec.EXACTLY);
        mTextView.measure(widthMeasureSpec,heightMeasureSpec);
```

- warap_content

>如下measure

```
        int widthMeasureSpec = View.MeasureSpec.makeMeasureSpec((1<<30)-1, View.MeasureSpec.AT_MOST);
        int heightMeasureSpec = View.MeasureSpec.makeMeasureSpec((1<<30)-1, View.MeasureSpec.AT_MOST);
        mTextView.measure(widthMeasureSpec,heightMeasureSpec);
```

>注意到(1<<30)-1， 通过分析MeasureSpec的实现可以知道，View的尺寸三十位的二进制表示，也就是说最大是30个1（2^30-1）,也就是（1<30-1）,在最大的模式下，我们用View理论上能支持最大值去构造MwasureSpec是合理的

>关于View的measure，网络上有两个错误的用法，为什么说是错误的，首先其违背了系统的内部实现规范（因为无法通过错误的MeasureSpec去得出合理的SpecMode，从而导致measure过程出错，其次不能保证mwasure出正确的结果）

- 第一种错误的方法:

```
        int widthMeasureSpec = View.MeasureSpec.makeMeasureSpec(-1, View.MeasureSpec.UNSPECIFIED);
        int heightMeasureSpec = View.MeasureSpec.makeMeasureSpec(-1, View.MeasureSpec.UNSPECIFIED);
        mTextView.measure(widthMeasureSpec,heightMeasureSpec);
```
- 第二种错误的用法

```
mTextView.measure(LayoutParams.WRAP_CONTENT,LayoutParams.WRAP_CONTENT);
```

### 2.layout过程

>Layout的作用是ViewGroup用来确定子元素的作用的，当ViewGroup的位置被确认之后，他的layout就会去遍历所有子元素并且调用onLayout方法，在layout方法中onLayou又被调用，layout的过程和measure过程相比就要简单很多了，layout方法确定了View本身的位置，而onLayout方法则会确定所有子元素的位置，先看View的layout方法


```
    public void layout(int l, int t, int r, int b) {
        if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
            onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        }

        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;

        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            onLayout(changed, l, t, r, b);
            mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnLayoutChangeListeners != null) {
                ArrayList<OnLayoutChangeListener> listenersCopy =
                        (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
                int numListeners = listenersCopy.size();
                for (int i = 0; i < numListeners; ++i) {
                    listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
                }
            }
        }

        mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
        mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
    }
```

>layout的方法的大致流程如下，首先会通过一个setFrame方法来设定View的四个顶点的位置，即初始化mLeft,mTop,mRight,mBottom这四个值，View的四个顶点一旦确定，那么View在父容器的位置也就确定了，接下来会调用onLayout方法，这个方法的用途是调用父容器确定子元素的位置，和onMeasure类似，onLayout的具体位置实现同样和具体布局有关，所有View和ViewGroup均没有真正的实现onLayout方法，我们来看一下LinearLayout的onLayout

```
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        if (mOrientation == VERTICAL) {
            layoutVertical(l, t, r, b);
        } else {
            layoutHorizontal(l, t, r, b);
        }
    }
```

>很好理解，是吧，，这个和onMeasure有点类似，我们拿layoutVertical来说，先看源码:

```
   void layoutVertical(int left, int top, int right, int bottom) {
        final int paddingLeft = mPaddingLeft;

        int childTop;
        int childLeft;
        
        // Where right end of child should go
        final int width = right - left;
        int childRight = width - mPaddingRight;
        
        // Space available for child
        int childSpace = width - paddingLeft - mPaddingRight;
        
        final int count = getVirtualChildCount();

        final int majorGravity = mGravity & Gravity.VERTICAL_GRAVITY_MASK;
        final int minorGravity = mGravity & Gravity.RELATIVE_HORIZONTAL_GRAVITY_MASK;

        switch (majorGravity) {
           case Gravity.BOTTOM:
               // mTotalLength contains the padding already
               childTop = mPaddingTop + bottom - top - mTotalLength;
               break;

               // mTotalLength contains the padding already
           case Gravity.CENTER_VERTICAL:
               childTop = mPaddingTop + (bottom - top - mTotalLength) / 2;
               break;

           case Gravity.TOP:
           default:
               childTop = mPaddingTop;
               break;
        }

        for (int i = 0; i < count; i++) {
            final View child = getVirtualChildAt(i);
            if (child == null) {
                childTop += measureNullChild(i);
            } else if (child.getVisibility() != GONE) {
                final int childWidth = child.getMeasuredWidth();
                final int childHeight = child.getMeasuredHeight();
                
                final LinearLayout.LayoutParams lp =
                        (LinearLayout.LayoutParams) child.getLayoutParams();
                
                int gravity = lp.gravity;
                if (gravity < 0) {
                    gravity = minorGravity;
                }
                final int layoutDirection = getLayoutDirection();
                final int absoluteGravity = Gravity.getAbsoluteGravity(gravity, layoutDirection);
                switch (absoluteGravity & Gravity.HORIZONTAL_GRAVITY_MASK) {
                    case Gravity.CENTER_HORIZONTAL:
                        childLeft = paddingLeft + ((childSpace - childWidth) / 2)
                                + lp.leftMargin - lp.rightMargin;
                        break;

                    case Gravity.RIGHT:
                        childLeft = childRight - childWidth - lp.rightMargin;
                        break;

                    case Gravity.LEFT:
                    default:
                        childLeft = paddingLeft + lp.leftMargin;
                        break;
                }

                if (hasDividerBeforeChildAt(i)) {
                    childTop += mDividerHeight;
                }

                childTop += lp.topMargin;
                setChildFrame(child, childLeft, childTop + getLocationOffset(child),
                        childWidth, childHeight);
                childTop += childHeight + lp.bottomMargin + getNextLocationOffset(child);

                i += getChildrenSkipCount(child, i);
            }
        }
    }

```

>这里分析一下layoutVertical的代码逻辑，可以看到，此方法会遍历所有子元素并调用setChildFrame方法来为子元素指定对应的位置，其中childTop会逐渐变大，这就意味着后面的子元素会被放置在靠下的位置，这刚好符合树立方向的线性布局，至于setChildFrame，他仅仅是调用元素的layout方法而已，这样的父容器在layout方法中完成自己的定位以后，就通过onLayout方法去调用，子元素又会通过自己的layout方法来确定自己的位置，这样一层一层传递下去完成整个View树的layout过程，setChildFrame方法可以看:


```
    private void setChildFrame(View child, int left, int top, int width, int height) {        
        child.layout(left, top, left + width, top + height);
    }
```

>我们注意到setChildFrame中的width和height实际上就是子元素测量宽高，从下面的代码可以看出

```
final int childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(Math.max(0, childHeight), MeasureSpec.EXACTLY);
final int childWidthMeasureSpec = getChildMeasureSpec(widthMeasureSpec,mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin,lp.width);
child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
```

>而在Layout方法中通过setFrame去设置子元素的四个顶点位置，方法中有这么几句:

```
            mLeft = left;
            mTop = top;
            mRight = right;
            mBottom = bottom;

```

>下面我们来回到之前的问题，View的测量宽高和最终宽高有什么区别，这个问题现在可以具体回答了，View的getMeasureWidth和getWidth这两个方法有什么区别？至于getMeasureHeight和getHeight是完全一样的，为了回答这个问题我们首先来看下getWidth和getHeight具体实现

```
    public final int getWidth() {
        return mRight - mLeft;
    }
    
        public final int getHeight() {
        return mBottom - mTop;
    }

```

>从getWidth和getHeight的源码在结合这四个变量的赋值来看，getWidth返回的刚好是View测量的值，而getHeight也是一样的，所以我们可用回答上面的问题了：在View的默认实现中，View的测量宽高和最终的是一样的，只不过一个是measure过程，一个是layout过程，而最终形成的是layout过程，即两者的赋值时机不同，测量宽高的赋值时机，稍微早一些，因此，在日常开发中，我们可用认为他们是相等的，但是还是有些不相同的，我们可用重写View的layout方法

```
	public void layout(int l,int t,int r, int b){
    	super.layout(l,t,t+100,b+100);
    }
```

>上述代码会导致在任何平台下View的最终宽高总是比测量大于100px,虽然这样这样会导致View显示不正常和没什么意义，但是这证明了测量 不等于 最终，另一种情况是在某种情况下，View需要多次measure才能确定自己的测量宽高，在前几次的测量过程中，其得出的测量宽高是不一致的但最终是一致的

### 3.draw过程
>Drawable是比較简单的，他的作用是将View绘制到屏幕上面，View的绘制过程由如下几个步骤：

- 1.绘制背景
- 2.绘制自己
- 3.绘制children
- 4.绘制装饰

>这一点我们看他的源码就知道了

```
   public void draw(Canvas canvas) {
        final int privateFlags = mPrivateFlags;
        final boolean dirtyOpaque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE &&
                (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
        mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;

        /*
         * Draw traversal performs several drawing steps which must be executed
         * in the appropriate order:
         *
         *      1. Draw the background
         *      2. If necessary, save the canvas' layers to prepare for fading
         *      3. Draw view's content
         *      4. Draw children
         *      5. If necessary, draw the fading edges and restore layers
         *      6. Draw decorations (scrollbars for instance)
         */

        // Step 1, draw the background, if needed
        int saveCount;

        if (!dirtyOpaque) {
            drawBackground(canvas);
        }

        // skip step 2 & 5 if possible (common case)
        final int viewFlags = mViewFlags;
        boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
        boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
        if (!verticalEdges && !horizontalEdges) {
            // Step 3, draw the content
            if (!dirtyOpaque) onDraw(canvas);

            // Step 4, draw the children
            dispatchDraw(canvas);

            // Overlay is part of the content and draws beneath Foreground
            if (mOverlay != null && !mOverlay.isEmpty()) {
                mOverlay.getOverlayView().dispatchDraw(canvas);
            }

            // Step 6, draw decorations (foreground, scrollbars)
            onDrawForeground(canvas);

            // we're done...
            return;
        }

        /*
         * Here we do the full fledged routine...
         * (this is an uncommon case where speed matters less,
         * this is why we repeat some of the tests that have been
         * done above)
         */

        boolean drawTop = false;
        boolean drawBottom = false;
        boolean drawLeft = false;
        boolean drawRight = false;

        float topFadeStrength = 0.0f;
        float bottomFadeStrength = 0.0f;
        float leftFadeStrength = 0.0f;
        float rightFadeStrength = 0.0f;

        // Step 2, save the canvas' layers
        int paddingLeft = mPaddingLeft;

        final boolean offsetRequired = isPaddingOffsetRequired();
        if (offsetRequired) {
            paddingLeft += getLeftPaddingOffset();
        }

        int left = mScrollX + paddingLeft;
        int right = left + mRight - mLeft - mPaddingRight - paddingLeft;
        int top = mScrollY + getFadeTop(offsetRequired);
        int bottom = top + getFadeHeight(offsetRequired);

        if (offsetRequired) {
            right += getRightPaddingOffset();
            bottom += getBottomPaddingOffset();
        }

        final ScrollabilityCache scrollabilityCache = mScrollCache;
        final float fadeHeight = scrollabilityCache.fadingEdgeLength;
        int length = (int) fadeHeight;

        // clip the fade length if top and bottom fades overlap
        // overlapping fades produce odd-looking artifacts
        if (verticalEdges && (top + length > bottom - length)) {
            length = (bottom - top) / 2;
        }

        // also clip horizontal fades if necessary
        if (horizontalEdges && (left + length > right - length)) {
            length = (right - left) / 2;
        }

        if (verticalEdges) {
            topFadeStrength = Math.max(0.0f, Math.min(1.0f, getTopFadingEdgeStrength()));
            drawTop = topFadeStrength * fadeHeight > 1.0f;
            bottomFadeStrength = Math.max(0.0f, Math.min(1.0f, getBottomFadingEdgeStrength()));
            drawBottom = bottomFadeStrength * fadeHeight > 1.0f;
        }

        if (horizontalEdges) {
            leftFadeStrength = Math.max(0.0f, Math.min(1.0f, getLeftFadingEdgeStrength()));
            drawLeft = leftFadeStrength * fadeHeight > 1.0f;
            rightFadeStrength = Math.max(0.0f, Math.min(1.0f, getRightFadingEdgeStrength()));
            drawRight = rightFadeStrength * fadeHeight > 1.0f;
        }

        saveCount = canvas.getSaveCount();

        int solidColor = getSolidColor();
        if (solidColor == 0) {
            final int flags = Canvas.HAS_ALPHA_LAYER_SAVE_FLAG;

            if (drawTop) {
                canvas.saveLayer(left, top, right, top + length, null, flags);
            }

            if (drawBottom) {
                canvas.saveLayer(left, bottom - length, right, bottom, null, flags);
            }

            if (drawLeft) {
                canvas.saveLayer(left, top, left + length, bottom, null, flags);
            }

            if (drawRight) {
                canvas.saveLayer(right - length, top, right, bottom, null, flags);
            }
        } else {
            scrollabilityCache.setFadeColor(solidColor);
        }

        // Step 3, draw the content
        if (!dirtyOpaque) onDraw(canvas);

        // Step 4, draw the children
        dispatchDraw(canvas);

        // Step 5, draw the fade effect and restore layers
        final Paint p = scrollabilityCache.paint;
        final Matrix matrix = scrollabilityCache.matrix;
        final Shader fade = scrollabilityCache.shader;

        if (drawTop) {
            matrix.setScale(1, fadeHeight * topFadeStrength);
            matrix.postTranslate(left, top);
            fade.setLocalMatrix(matrix);
            p.setShader(fade);
            canvas.drawRect(left, top, right, top + length, p);
        }

        if (drawBottom) {
            matrix.setScale(1, fadeHeight * bottomFadeStrength);
            matrix.postRotate(180);
            matrix.postTranslate(left, bottom);
            fade.setLocalMatrix(matrix);
            p.setShader(fade);
            canvas.drawRect(left, bottom - length, right, bottom, p);
        }

        if (drawLeft) {
            matrix.setScale(1, fadeHeight * leftFadeStrength);
            matrix.postRotate(-90);
            matrix.postTranslate(left, top);
            fade.setLocalMatrix(matrix);
            p.setShader(fade);
            canvas.drawRect(left, top, left + length, bottom, p);
        }

        if (drawRight) {
            matrix.setScale(1, fadeHeight * rightFadeStrength);
            matrix.postRotate(90);
            matrix.postTranslate(right, top);
            fade.setLocalMatrix(matrix);
            p.setShader(fade);
            canvas.drawRect(right - length, top, right, bottom, p);
        }

        canvas.restoreToCount(saveCount);

        // Overlay is part of the content and draws beneath Foreground
        if (mOverlay != null && !mOverlay.isEmpty()) {
            mOverlay.getOverlayView().dispatchDraw(canvas);
        }

        // Step 6, draw decorations (foreground, scrollbars)
        onDrawForeground(canvas);
    }

```

>从setwilINotDraw这个方法的注释中可以看出，如果一个View不需要绘制任何内容,那么设置这个标记位为true以后，系统会进行相应的优化。默认情况下，View没有启用这个校化标记位，但是ViewGroup会默认启用这个优化标记位。这个标记位对实际开发的意义是。当我们的自定义控件继承于ViewGroup并且本身不具备绘制功能时，就可以开启这个标记位从而便于系统进行后续的优化。当然，当明确知道一个ViewGroup需要通过onDraw来绘制内容时，我们需要显式地关闭WILL_NOT_DRAW这个标记位。


