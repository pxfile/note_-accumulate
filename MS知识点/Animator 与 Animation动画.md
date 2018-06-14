Android 动画
===


# 简要分析一下Animator 与 Animation，androidanimator

在 Android 的开发过程中, 大家平时一般都或多或少会使用到一些动画, 通常大家一般使用的都是 Animation, 但是实际上Android 在3.0的时候就已经推出了 Animator 框架用以提升 Android 本身的动画效果,虽然我们一般基于2.x 开发的时候无法享受到 Animator 服务,但是这并不影响我们来体验一下他的强大之处.

首先我们先来了解一下 Animation,Animation框架的支持要比 Animator 早得多,从 Android 发布之日起就一直存在,他主要有以下几个子类,AlphaAnimation(透明度), RotateAnimation(旋转), ScaleAnimation(缩放), TranslateAniamtion(平移), AnimationSet(动画集合)

从名字上,我们就能很清楚的知道 Aniamtion 所支持的动画种类还是很少的, 无非是透明,旋转,缩放,平移这几种的子集.

而相较于 Aniamtion而言, Animator 动画则显得更加强大, 他不仅可以针对 View 实行动画, 甚至可以对所有的 Object 执行"动画"操作,并且使用 Animator 之后的动画效果与使用 Animation 的效果也完全不同.

Animator 动画与 Animation 动画实际上有很多类似的接口,例如 duration 和 interceptor, 其作用都是用来判定动画具体的实现时长以及差值器,对于这个,我们就不用过多介绍.

我们先来看一下 Animator 动画与 Animation 动画的相关实现原理: 

**(1)对于 Animation 动画:**

他的实现机制是,在每次进行绘图的时候,通过对整块画布的矩阵进行变换,从而实现一种视图坐标的移动,但实际上其在 View 内部真实的坐标位置及其他相关属性始终恒定.

**(2)对于 Animator 动画:**

Animator 动画的实现机制说起来其实更加简单一点,因为他其实只是计算动画开启之后,结束之前,到某个时间点得时候,某个属性应该有的值,然后通过回调接口去设置具体值,其实 Animator 内部并没有针对某个 view 进行刷新,来实现动画的行为,动画的实现是在设置具体值的时候,方法内部自行调取的类似 invalidate 之类的方法实现的.也就是说,使用 Animator ,内部的属性发生了变化.

说完他们的基本实现原理,我们现在来**对比一下他们的优势劣势**:

**(1)版本兼容**

不得不说,相对于 Animation,Animator 的版本兼容性还是太差,直到 Android3.0才开始出现的 Animator, 是无法满足目前开发环境2.x 的兼容支持的,而且在 android 官方的 support 包中也没有对于低版本的 Animator 进行支持,所以单从版本兼容来看, Animator 还是不够的,不过这是系统历史原因,我们只能接受.

**(2)实现效率**

同样的,这也是 Animator 的一个缺点,由于 Animator 是直接通过设置对象的 setter,getter 方法,来起到动画显示效果的,所以为了满足对任意对象调用正确方法, Animator 使用了 Java 反射机制, 而 Animation 则是直接通过代码对矩阵进行处理,所以就效率这一方面而言, Animator比不上 Animation

已经说了 Animator 相较于 Animation 的两种劣势了,那么我们再来说说 **Animator 相较于 Animation 的优势**

**(3)适用性**

在上一个分析中,我们看到了由于 Animator 使用了反射机制导致其效率偏低,但是这也带来了他适用的对象范围的增加, Animation 仅对 View 这一种对象有用,但是 Animator 可以设置任意对象的属性,使其在某段时间内进行变化

**(4)使用效果**

相信大家平时使用 Animation 的时候,都有发现当正在进行平移移动,或者动画结束后,但位置发生改变的时候,你点击之前的位置,点击效果仍然存在,这就是因为 View 在内部的坐标位置其实没有发生改变,而如果使用 Animator 进行位移变换,那么你的点击位置就会随着动画效果发生相应改变,所以即使你正处在动画过程中,你也可以去点击按钮得到你想要的效果.

以上四点就是 Animation 和 Animator 的优势劣势分析,希望对大家有用

### Unity3D43版本把动画系统改成了Animator，以前Animation的Play automatically自动播放键在哪 

Animator有状态机来控制动画帧的播放
比如有setTrigger的方法
建议你看看官方文档
打开animator界面有默认的state，你新建一个空的state，把空state设置为默认state就好了

### android动画，translateanimation，是否可以实现让两个控件流畅的执行同一动画？ 

是的，你可以使用动画的情况下，一个单独的写了一个内部??方法，再加上视图参数控制传入的使用线程同步播放的。当然，楼上的说是也。不过是一个全球性的影响，不是每一个组件的影响。

#### 采用ValueAnimator，监听动画过程，自己实现属性的改变

首先说说啥是ValueAnimator，ValueAnimator本身不作用于任何对象，也就是说直接使用它没有任何动画效果。它可以对一个值做动画，然后我们可以监听其动画过程，在动画过程中修改我们的对象的属性值，这样也就相当于我们的对象做了动画。
```
private void performAnimate(final View target, final int start, final int end) {
    ValueAnimator valueAnimator = ValueAnimator.ofInt(1, 100);
 
    valueAnimator.addUpdateListener(new AnimatorUpdateListener() {
 
        //持有一个IntEvaluator对象，方便下面估值的时候使用
        private IntEvaluator mEvaluator = new IntEvaluator();
 
        @Override
        public void onAnimationUpdate(ValueAnimator animator) {
            //获得当前动画的进度值，整型，1-100之间
            int currentValue = (Integer)animator.getAnimatedValue();
            Log.d(TAG, current value:  + currentValue);
 
            //计算当前进度占整个动画过程的比例，浮点型，0-1之间
            float fraction = currentValue / 100f;
 
            //这里我偷懒了，不过有现成的干吗不用呢
            //直接调用整型估值器通过比例计算出宽度，然后再设给Button
            target.getLayoutParams().width = mEvaluator.evaluate(fraction, start, end);
            target.requestLayout();
        }
    });
 
    valueAnimator.setDuration(5000).start();
}
 
@Override
public void onClick(View v) {
    if (v == mButton) {
        performAnimate(mButton, mButton.getWidth(), 500);
    }
}
```