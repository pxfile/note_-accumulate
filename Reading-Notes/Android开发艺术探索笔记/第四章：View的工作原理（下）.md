# View的工作原理（下）
---

>我们上篇BB了这么多，这篇就多多少少要来点实战了，上篇主席叫我多点自己的理解，那我就多点真诚，少点套路了，老司机，开车吧！

>我们这一篇就扯一个内容，那就是自定义View

- 自定义View
	- 自定义View的分类
	- 自定义View的须知
	- 自定义View的实例
	- 自定义View的思想

## 一.自定义View的分类
>自定义View百花齐放，没有什么具体的分类，不过可以从特性大致的分为4类，其实在我看来，就三类，继承原生View，继承View和继承ViewGroup。

- 1.继承View重写onDraw方法
>重写了绘制，一般就是想自己实现某些图形了，因为原生控件已经满足不了你了，很显然这需要绘制的方式来完成，采用这个方式需要自身支=warp_content,并且pading也要自己处理，比较考验你的功底了

- 2.继承ViewGroup派生出来的Layout
>这个相当于重写容器了，当某些效果看起来像是View的组合的时候，就是他上场的时候了，不过这个很复杂，需要合理的使用测量和布局这两个过程，还要兼顾子元素的这两个过程

- 3.继承特定的View
>比如TextView，就是重写原生的View嘛，比如你想让TextView默认有颜色之类的，有一些小改动，这个就可以用它的，他相对来说比较简单，这个就不需要自己支持包裹内容和pading了

- 4.继承特定的ViewGroup
>这个和上述一样，只不过是重写容器而已，这个也比较常见，事件分发的时候用的也多

## 二.自定义View的须知
>这节大致的说一下注意事项

- 1.让View支持warp_content
>这个在之前将测量的时候说过，如果你不特殊处理一下是达不到满意的效果的，这里就不重复了

- 2.如果有有必要，让你的View支持padding
>这是因为如果你不处理下的话，那么该属性是不会生效的，在ViewGroup也是一样

- 3.尽量不要在View中使用Handler
>为什么不能用，是因为没有必要，View本身就有一系列的post方法，当然，你想用也没人拦着你，我倒是觉得handler写起来代码简洁很多

- 4.View中如果有线程或者动画，需要及时停止，参考View#onDetachedFromWindow
>这个问题那就更好理解了，你要是不停止这个线程或者动画，容易导致内存溢出的，所以你要在一个合适的机会销毁这些资源，在Activity有生命周期，而在View中，当View被remove的时候，onDetachedFromWindow会被调用，，和此方法对应的是onAttachedToWindow

- 5.View带有滑动嵌套时，需要处理好滑动冲突
>滑动冲突之前就BB过，这里就不讲了

## 三.自定义View的实例

- 1.继承View重写onDraw方法
>我们来实现一个很简单的图形：圆。尽管如此，还是有很多细节需要注意的，实现的过程中需要考虑warp_content和padding，OK，我们先来看代码

```java
public class CircleView extends View {

    //颜色
    private int mColor = Color.RED;
    //画笔样式
    private Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);

    public CircleView(Context context) {
        super(context);
        init();
    }

    public CircleView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public CircleView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    //初始化
    private void init() {
        //设置颜色
        mPaint.setColor(mColor);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //View的宽
        int width = getWidth();
        //View的高
        int height = getHeight();
        //圆的半径 = 宽和高比较出的数 / 2
        int radiu = Math.min(width, height) / 2;
        //绘制圆
        canvas.drawCircle(width / 2, height / 2, radiu, mPaint);
    }
}

```

>上面的代码就绘制出了一个圆，运行看下效果

--------------------------------------------------------------------------

>上面的代码很简单，估摸着会点自定义的完全能写出来，我们写这个案例就是要抛砖引玉，不信，我们接着看下去，我们把布局改成这个样子

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.liuguilin.viewwork.view.CircleView
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:background="#000000" />

</LinearLayout>

```

>现在我们来运行一下你就会看到不一样的效果了

--------------------------------------------------------------------------


>接下来我们再调整一下，只给他增加一个

```xml
	android:layout_margin="20dp"
```

>这样会是什么效果呢？

--------------------------------------------------------------------------

>这样按理说也是我们预期的效果，对吧，这样的话margin属性是生效的，这是因为margin由父容器所控制的，所以不需要View去动，我们进一步实验，我现在给他继续增加，加上一个padding

```xml
	android:padding="20dp"
```

>这里是重头戏了，我们运行后会发现，他没什么反应呀，我们之前说过，如果你直接继承View,在测量的时候需要做点处理的，不然的话，你的warp_content就和match_parent是一样的了。

>为了解决这几个问题，我们需要做如下的处理

>首先，关于warp_content的问题，我们只需要指定一个warp_content模式宽/高即可，比如设置200px作为默认的宽高

>其次，针对padding的问题，我们再绘制的时候考虑进去就好了，修改后的onDraw如下


```xml
   @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //padding值
        int left = getPaddingLeft();
        int right = getPaddingRight();
        int top = getPaddingTop();
        int bottom = getPaddingBottom();
        //View的宽
        int width = getWidth() - left - right;
        //View的高
        int height = getHeight() - top - bottom;
        //圆的半径 = 宽和高比较出的数 / 2
        int radiu = Math.min(width, height) / 2;
        //绘制圆
        canvas.drawCircle(left + width / 2, top + height / 2, radiu, mPaint);
    }

```

>这样就解决了，主要的逻辑就是绘制的时候考虑到View四周的空白即可，圆心和半径都会考虑到，现在我们来运行下，就有效果了

--------------------------------------------------------------------------

>最后，为了让View更加容易应用，我们需要提供一些自定义的属性，这些怎么玩呢，我们继续看

>第一步实在values目录下面创建自定义属性的xml，比如attrs.xml,也可以其他名字，名字没什么限制，不过为了规范，还是...你懂的，我们就来写一个

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="CircleView">
        <attr name="circle_color" format="color" />
    </declare-styleable>
</resources>
```

>这个很简单吧，我们只定义了一个颜色的属性，这里面有个format是类型，看下就懂了，然后呢

>第二步，在View的构造方法里解析到我们这个属性，仔细看代码:

```xml
    public CircleView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        TypedArray type = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
        //没有指定颜色的话默认红色
        mColor = type.getColor(R.styleable.CircleView_circle_color, Color.RED);
        type.recycle();
        init();
    }

```

>这段代码就是加载一个资源文件，拿到里面的属性，如果没有指定的话，默认就是红色了，那我们要使用的话，写一个命名空间，然后...：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.liuguilin.viewwork.view.CircleView
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:layout_margin="20dp"
        android:background="#000000"
        android:padding="20dp"
        app:circle_color="@color/colorPrimary" />

</LinearLayout>


```

>上门的布局唯一要注意的就是这个命名空间了 xmlns:app="http://schemas.android.com/apk/res-auto"，然后就可以使用app:属性的方式添加了，那我们运行一下，效果也很明显，来看下全部的代码吧:

```java
public class CircleView extends View {

    //颜色
    private int mColor = Color.RED;
    //画笔样式
    private Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);

    public CircleView(Context context) {
        super(context);
        init();
    }

    public CircleView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public CircleView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        TypedArray type = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
        //没有指定颜色的话默认红色
        mColor = type.getColor(R.styleable.CircleView_circle_color, Color.RED);
        type.recycle();
        init();
    }

    //初始化
    private void init() {
        //设置颜色
        mPaint.setColor(mColor);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSpecSize = MeasureSpec.getMode(widthMeasureSpec);
        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSpecSize = MeasureSpec.getMode(heightMeasureSpec);
        if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(200, 200);
        } else if (widthSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(200, heightSpecSize);
        } else if (heightSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(widthSpecSize, 200);
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //padding值
        int left = getPaddingLeft();
        int right = getPaddingRight();
        int top = getPaddingTop();
        int bottom = getPaddingBottom();
        //View的宽
        int width = getWidth() - left - right;
        //View的高
        int height = getHeight() - top - bottom;
        //圆的半径 = 宽和高比较出的数 / 2
        int radiu = Math.min(width, height) / 2;
        //绘制圆
        canvas.drawCircle(left + width / 2, top + height / 2, radiu, mPaint);
    }
}
```

>这代码清晰脱俗吧，简单好记，就是这样

- 2.继承ViewGroup派生出来的Layout
>这个同等于自定义布局，在之前介绍滑动的时候，有过类似的例子，主席就偷懒的搬上来了，当时分析滑动冲突的两种自定义View：HorizontalScrollViewEx和StickyLayout,其中HorizontalScrollViewEx就是通过继承ViewGroup来实现的，我们再次来分析他的测量和布局过程

>这里BB一句，要规范的写View，需要一定的代价，这个，需要去看线性布局去了解了，他们的实现都很复杂，对于HorizontalScrollViewEx来说，就不这么精细了

>回顾下HorizontalScrollViewEx的功能，他类似于ViewPager，或者说水平方向的线性布局，它内部的View可以竖直滑动，解决他的冲突的代码就不提了，我们主要还是看下他的测量

```java
   @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int measureWidth = 0;
        int measureHeight = 0;
        final int childCount = getChildCount();
        measureChildren(widthMeasureSpec,heightMeasureSpec);

        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSpecSize = MeasureSpec.getMode(widthMeasureSpec);
        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSpecSize = MeasureSpec.getMode(heightMeasureSpec);
        if(childCount ==0){
            setMeasuredDimension(0, 0);
        }else if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST) {
            final View childView = getChildAt(0);
            measureWidth = childView.getMeasuredWidth() * childCount;
            measureHeight = childView.getMeasuredHeight();
            setMeasuredDimension(measureWidth, measureHeight);
        } else if (widthSpecMode == MeasureSpec.AT_MOST) {
            final View childView = getChildAt(0);
            measureWidth = childView.getMeasuredWidth() * childCount;
            setMeasuredDimension(measureWidth, heightSpecSize);
        } else if (heightSpecMode == MeasureSpec.AT_MOST) {
            final View childView = getChildAt(0);
            measureHeight = childView.getMeasuredHeight();
            setMeasuredDimension(widthSpecSize, measureHeight);
        }
    }

```

>这里发现一点小bug，不过不碍事，这里的逻辑呢，可以这样理理，首先有没有子元素，没有就全部都是0，有的话再去判断是否是warp_content,,如果是包裹内容，那这个控件的宽度就是所以的总和了，如果高度采用包裹内容，那这个控件就是第一个子元素的高度，这样说应该好理解一点

>再回来说说规范性，上面的代码可以说有两点吧，首先，是不应该直接设置为0，还有就是测量的时候没有考虑到padding和子元素的maggin，好的我们继续来看下onLayout

```java
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int childLeft = 0;
        final int childCount = getChildCount();
        mChildSize = childCount;
        for (int i = 0; i < childCount; i++) {
            final View childView = getChildAt(i);
            final int childWidth = childView.getMeasuredWidth();
            mChildWidth = childWidth;
            childView.layout(childLeft, 0, childLeft + childWidth, childView.getMeasuredHeight());
            childLeft += childWidth;
        }
    }


```

>这个布局的逻辑也没多少代码，我们拿到子元素之后将其放在合适的位置，位置是从左往右的，但是仍然没有考虑padding和子元素的maggin，这个也不是很规范，好的，那我们直接撸完整代码:


```java
public class HorizontalScrollViewEx extends ViewGroup {

    private int mChildrenSize;
    private int mChildWidth;
    private int mChildIndex;

    //分别记录上次滑动的坐标
    private int mLastX = 0;
    private int mLastY = 0;

    //分别记录上次滑动的坐标
    private int mLastXIntercept = 0;
    private int mLastYIntercept = 0;

    private Scroller mScroller;

    private VelocityTracker mVelocityTracker;

    public HorizontalScrollViewEx(Context context) {
        super(context);
        init();
    }

    public HorizontalScrollViewEx(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public HorizontalScrollViewEx(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        if (mScroller == null) {
            mScroller = new Scroller(getContext());
            mVelocityTracker = VelocityTracker.obtain();
        }
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean intercepted = false;
        int x = (int) ev.getX();
        int y = (int) ev.getY();

        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                intercepted = false;
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                    intercepted = true;
                }
                break;
            case MotionEvent.ACTION_MOVE:
                int deltaX = x - mLastXIntercept;
                int deltaY = y - mLastYIntercept;
                if (Math.abs(deltaX) > Math.abs(deltaY)) {
                    intercepted = true;
                } else {
                    intercepted = false;
                }
                break;
            case MotionEvent.ACTION_UP:
                intercepted = false;
                break;
        }
        mLastX = x;
        mLastY = y;
        mLastXIntercept = x;
        mLastYIntercept = y;
        return intercepted;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        mVelocityTracker.addMovement(event);
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                }
                break;
            case MotionEvent.ACTION_MOVE:
                int scrollX = getScrollX();
                mVelocityTracker.computeCurrentVelocity(1000);
                float xVelocity = mVelocityTracker.getXVelocity();

                if (Math.abs(xVelocity) >= 50) {
                    mChildIndex = xVelocity > 0 ? mChildIndex - 1 : mChildIndex + 1;
                } else {
                    mChildIndex = (scrollX + mChildWidth / 2) / mChildWidth;
                }
                mChildIndex = Math.max(0, Math.min(mChildIndex, mChildrenSize - 1));
                int dx = mChildIndex * mChildWidth - scrollX;
                smoothScrollBy(dx, 0);
                mVelocityTracker.clear();
                break;
        }
        mLastX = x;
        mLastY = y;
        return true;
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int measureWidth = 0;
        int measureHeight = 0;
        final int childCount = getChildCount();
        measureChildren(widthMeasureSpec, heightMeasureSpec);

        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSpecSize = MeasureSpec.getMode(widthMeasureSpec);
        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSpecSize = MeasureSpec.getMode(heightMeasureSpec);
        if (childCount == 0) {
            setMeasuredDimension(0, 0);
        } else if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST) {
            final View childView = getChildAt(0);
            measureWidth = childView.getMeasuredWidth() * childCount;
            measureHeight = childView.getMeasuredHeight();
            setMeasuredDimension(measureWidth, measureHeight);
        } else if (widthSpecMode == MeasureSpec.AT_MOST) {
            final View childView = getChildAt(0);
            measureWidth = childView.getMeasuredWidth() * childCount;
            setMeasuredDimension(measureWidth, heightSpecSize);
        } else if (heightSpecMode == MeasureSpec.AT_MOST) {
            final View childView = getChildAt(0);
            measureHeight = childView.getMeasuredHeight();
            setMeasuredDimension(widthSpecSize, measureHeight);
        }
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int childLeft = 0;
        final int childCount = getChildCount();
        mChildrenSize = childCount;
        for (int i = 0; i < childCount; i++) {
            final View childView = getChildAt(i);
            final int childWidth = childView.getMeasuredWidth();
            mChildWidth = childWidth;
            childView.layout(childLeft, 0, childLeft + childWidth, childView.getMeasuredHeight());
            childLeft += childWidth;
        }
    }

    private void smoothScrollBy(int dx, int dy) {
        mScroller.startScroll(getScrollX(), 0, dx, 0, 500);
        invalidate();
    }

    @Override
    public void computeScroll() {
        if (mScroller.computeScrollOffset()) {
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            postInvalidate();
        }
    }

    @Override
    protected void onDetachedFromWindow() {
        mVelocityTracker.recycle();
        super.onDetachedFromWindow();
    }
}

```

