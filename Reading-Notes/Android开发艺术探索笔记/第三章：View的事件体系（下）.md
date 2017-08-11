#Android艺术开发探索第三章————View的事件体系（下）
---
>在这里就能学习到很多，主要还是对View的事件分发做一个体系的了解

##一.View的事件分发
>上篇大致的说了一下View的基础知识和滑动，现在我们再来聊聊一个比较核心的知识点，那就是事件分发了，而且他还是一个难点，我们更加应该掌握，View的滑动冲突一直都是很苦恼的，这里，我们就来一起探索一下

###1.点击事件的传递规则
>我们分析的点击事件可不是View.OnClickListener，而是我们MotionEvent，即点击事件，关于MotionEvent在上篇说过一点点，所谓点击事件的事件分发，其实就是对MotionEvent事件的分发过程，即当一个MoonEvent产生了以后，系统需要把这个事件传递给一个具体的View，而这个传递的过程就是分发过程。点击事件的分发过程由三个很重要的分发来完成.dispatchTouchEvent，onInterceptTouchEvent和onTouchEvent,下面我们先介绍一下这几个方法。

- **puhlic boolean dispatch fouchEvent(MotionEvent ev)**
>用来进行事件的分发。如果事件能够传递给当前View，那么此方法一定会被调用，返回结果受当前View的onTouchEvent和下级View的dispatchTouchEvent方法的影响，表示是否消耗当前事件。

- **public boolean onIntercept fouchEven(MotionEvent event)**
>在上述方法内部调用，用来判断是否拦截某个事件，如果当前View拦截了某个事件，那么在同一个事件序列当中，此方法不会被再次调用，返回结果表示是否拦截当前事件

- **public boolean onTouchEvent(MotionEvent event)**
>在dispatchTouchEvent方法中调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一个事件序列中，当前View无法再次接收到事件。

>上述三个方法到底有什么区别呢?它们是什么关系呢？其实它们的关系可以用如下伪代码可以了解

```
	@Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        boolean consume = false;
        if(onInterceptTouchEvent(ev)){
            consume = onTouchEvent(ev);
        }else {
            consume = child.dispatchTouchEvent(ev);
        }
        return consume;
        
    }
```

>上述的伪代码已经将三者的区别说明了，我们可以大致的了解传递的规则就是，对于一个根ViewGroup来说，点击事件产生以后，首先传递给
产，这时它的dispatchTouchEvent就会被调用，如果这个ViewGroup的onIntereptTouchEvent方法返回true就表示它要控截当前事件，接着事件就会交给这个ViewGroup处理，则他的onTouchEvent方法就会被调用；如果这个ViewGroup的onIntereptTouchEvent方法返回false就表示不需要拦截当前事件，这时当前事件就会继续传递给它的子元素，接着子元素的onIntereptTouchEvent方法就会被调用，如此反复直到事件被最终处理。

>当一个View需要处理事件时，如果它设置了OnTouchListener，那么OnTouchListener中的onTooch方法会被回调。这时事件如何处理还要看onTouch的返回值，如果返回false,那当前的View的方法OnTouchListener会被调用；如果返回true，那么onTouchEvent方法将不会被调用。由此可见，给View设置的OnTouchListener，其优先级比onTouchEvent要高，在onTouchEvent方法中，如果当前设置的有OnClickListener，那么它的onClick方法会用。可以看出，平时我们常用的OnClickListener，其优先级最低，即处于事尾端。

>当一个点击事件产生后，它的传递过程遵循如下顺序：Activity>Window-View，即事件总是先传递给Activity,Activity再传递给Window，最后Window再传递给顶级View顶级View接收到事件后，就会按照事件分发机制去分发事件。考虑一种情况，如果一个view的onTouchEvent返回false，那么它的父容器的onTouchEvent将会被调用，依此类推,如果所有的元素都不处理这个事件，那么这个事件将会最终传递给Activity处理，即Activity的onTouchEvent方法会被调用。这个过程其实也很好理解，我们可以换一种思路，假如点击事件是一个难题，这个难题最终被上级领导分给了一个程序员去处理（这是事件分发过程），结果这个程序员搞不定(onTouchEvent返回false)，现在该怎么办呢？难题必须要解决，那只能交给水平更高的上级解决（上级的onTouchEvent被调用)，如果上级再搞不定，那只能交给上级的上级去解决，就这样将难题一层层地向上抛，这是公司内部一种很常见的处理问题的过程。从这个角度来看，View的事件传递过程还是很贴近现实的，毕竟程序员也生活在现实中。

>关于事件传递的机制，这里给出一些结论，根据这些结论可以更好地理解整个传递机制，如下所示。

- （1）同一个事件序列是指从手指接触屏幕的那一刻起，到手指离开屏慕的那一刻结束，在这个过程中所产生的一系列事件，这个事件序列以down事件开始，中间含有数量不定的move事件，最后以up结束

- （2）正常情况下，一个事件序列只能被一个Visw拦截且消耗。这一条的原因可以参考（3），因为一旦一个元素拦截了某此事件，那么同一个事件序列内的所有事件都会直接交给它处理，因此同一个事件序列中的事件不能分别由两个View同时处理，但是通过特殊手段可以做到，比如一个Vew将本该自己处理的事件通过onTouchEvent强行传递给其他View处理。

- （3)某个View一旦决定拦截，那么这一个事件序列都只能由它来处理（如果事件序列能够传递给它的话)，并且它的onInterceprTouchEvent不会再被调用。这条也很好理解，就是说当一个View决定拦截一个事件后，那么系统会把同一个事件序列内的其他方法都直接交给它来处理，因此就不用再调用这个View的onInterceptTouchEvent去询问它是否要拦截了。

- (4）某个View一旦开始处理事件，如果它不消耗ACTON_DOWN事件(onTouchEvent返回了false)，那么同一事件序列中的其他事件都不会再交给它来处理，并且事件将重新交由它的父元素去处理，即父元素的onTouchEvent会被调用。意思就是事件一旦交给一个View处理，那么它就必须消耗掉，否则同一事件序列中剩下的事件就不再交给它来处理了，这就好比上级交给程序员一件事，如果这件事没有处理好，短期内上级就不敢再把事情交给这个程序员做了，二者是类似的道理。

- （5）如果View不消耗除ACTION_DOWN以外的其他事件，那么这个点击事件会消失，此时父元素的onTouchEvent并不会被调用，并且当前View可以持续收到后续的事件，最终这些消失的点击事件会传递给Activity处理。

- （6)ViewGroup默认不拦截任何事件。Android源码中ViewGroup的onInterceptTouchEvent方法默认返回false

- （7）View没有onInterceptTouchEvent方法，一旦有点击事件传递给它，那么它的onTouchEvent方法就会被调用。

- （8）view的onTouchEvent默认都会消耗事件（返回true)，除非它是不可点击的(clickable和longClickable同时为false)，View的longClickable属性默认都为false,clickable属性要分情况，比如Button的clickable属性默认为true，而TextView 的clickable属性默认为false

- （9)view 的enable.属性不影响onTouchEvent的默认返回值。哪怕一个View是disable状态的，只要它的clickable或者longclickable有一个为true，那么它的onTouchEvent就返会true。

- （10）onclick会发生的前提实际当前的View是可点击的，并且他收到了down和up的事件
-  (11)事件传递过程是由外到内的，理解就是事件总是先传递给父元素，然后再由父元素分发给子View，通过requestDisallowInterptTouchEvent方法可以再子元素中干预元素的事件分发过程，但是ACTION_DOWN除外


###2.事件分发的源码解析

####1.Activity对点击事件的分发过程 
>点击事件用MotionEvent来表示，当一个点击操作发生的时候，事件最先传递给Activity，由Activity的dispatchTouchEvent来进行事件的派发，具体的工作是由Activity内部的window来完成的，window会将事件传递给decor view,decor view一般都是当前界面的底层容器（setContentView所设置的父容器），通过Activity.getWindow.getDecorView()获得，我们可用先从dispatchTouchEvent的源码看起：

```
 public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
```

>首先事件交给Activity所依附的window，如果true那就结束了，false的话就没人处理，所有View的onTouchEvent都返回false，那么Activity的onTouch就会被调用

>接下来我们看下window是如何将事件传递给ViewGroup的，通过源码我们知道，window时一个抽象类，而window的super.dispatchTouchEvent(ev)方法也是抽象的，因此我们必须找到window的实现类

```
public abstract boolean superDispatchTouchEvent(MotionEvent event);
```

>那么window的实现类是什么呢？就是phonewindow，这点源码中有一段注释就说明了，意思大概就是winodw类控制顶级的View的外观和行为机制，他的唯一实现位于android.policy.PhoneWinodw中，当你要实例化这个window的时候，你并不知道他的细节，因此这个类会被重构，只有一个工厂方法可以使用，尽管这看起来有点模糊，不过我们可以看一下这个类的实现，尽管实例化的时候此类会被重构，只是重构而已，功能都是类似的

>由于Window的唯一实现是PhoneWindow，我们看一下他的处理是什么

```
public boolean superDispatchTouchEvent(MotionEvent ev){
        return mDecor.superDispatchTouchEvent(ev);
    }

```
>到这里逻辑就很清晰了，phoneWindow传递给了DecorView，这个FecorView是什么呢？

```
public class DecorView extends FrameLayout implements RootViewSurfaceTaker {
    
    private DecorView mDecor;
    
    @Override
    public final View getDecorView(){
        if(mDecor == null){
            installDesor():
        }0
        return  mDecor;
    }
}

```

>我们知道，通过((ViewGroup)getwindow().getDecorView().findViewByld(android.R.idcontent)).getChildAt（0）这种方式就可以获取Activity所设置的View，这个mDecor显然就是个子View。目前事件传递到了Decorview这里，由于DecorView继承自FrameLayou且是父View，所以最终事件会传递给View。换句话来说，事件肯定会传递到View，不然应用如何响应点击事件呢?不过这不是我们的重点，重点是事件到了View以后应该如何传递，这对我们更有用，从这里开始，事件已经传递到顶级View了，就是在Activity中通过
seContentview所设置的View，另外顶级View也叫根View，顶级View一般来说都是VewGroup。

####2.顶级View对事件的分发过程
>关于点击事件如何在View中进行分发，上一节已经做了详细的介绍，这里再大致回顾一下。点击事件达到顶级View（一般是一个ViewGroup)以后，会调用ViewGiroup的的dispatchTouchEvent方法，然后的逻辑是这样的：如果顶级ViewGroup拦截事件即
onIntercepTouchEvent返回true，则事件由ViewGroup处理，这时如果ViewGroup的mOnTouchListener被设置，则onTouch会被调用，否则onTouchEvent会被调用。也就是说如果都提供的话，onTouch会屏蔽掉onTouchEvent。在onTouchEvent中，如果设置了
mOnTouchListener,则onClick会被调用。如果顶级ViewGroup不拦截事件，则事件会传递给它所在的点击事件链上的子View，这时子View的dispatchTouchEvent会被调用。到此为止，事件已经从顶级View传递给了下一层View，接下来的传递过程和顶级View是一致的，
如此循环，完成整个事件的分发。

>首先看ViewGroup对点击事件的分发过程，其主要实现在ViewGroup的dispatchTouchEvent方法中，这个方法比较长，这里分段说明。先看下面一段，很显然，它描述的是当View是否拦截点击事情这个逻辑。

```
	 // Check for interception.
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                // There are no touch targets and this action is not an initial down
                // so this view group continues to intercept touches.
                intercepted = true;
            }
```

>从上面代码我们可以看出，ViewGroup在如下两种情况下会判断是否要拦截当前事件：事件类型为ACTION_DOWN或者mFirstTouchTarget!=null,ACTION_DOWN事件好理解，那么mFirstTouchTargetl=null是什么意思呢？这个从后面的代码逻辑可以看
出来，当事件由ViewGroup的子元素成功处理时，mFirstTouchTarget会被赋值并指向子元素，换种方式来说，当ViewGroup不拦截事件并将事件交由子元素处理时mFirstTouchTarget != null。反过来，一旦事件由当前ViewGroup拦截时，mFirstTouchTarget !=null就不成立。那么当ACTION_MOVE和ACTION_UP事件到来时，由于actionMasked == MotionEvent.ACTION_DOWN|| mFirstTouchTarget != null这个条件为false，将导致ViewGroup 的onInterceptTouchEvent不会再被调用，并且同一序列中的其他事件都会默认交给它处理。

>当然，这里有一种特殊情况，那就是FLAG_DISALLOW_INTERCEPT标记位，这个标记位是通过requestDisallowInterceptTouchEvent方法来设置的，一般用于子View中。FLAG_DISALLOW_INTERCEPT一旦设置后，ViewGroup将无法拦截除了ACTION_DOWN以外的其他点击事件。为什么说是除了ACTION_DOWN以外的其他事件呢?这是因为ViewGroup在分发事件时，如果是ACTION_DOWN就会重置
FLAG_DISALLOW_INTERCEPT这个标记位，将导致子View中设置的这个标记位无效。因此，当面对ACTION_DOWN事件时，ViewGroup总是会调用自己的onInterceptTouchEvent方法来询问自己是否要拦截事件，这一点从源码中也可以看出来,在下面的代码中，ViewGroup会在ACTION_DOWN事件到来时做重置的作用。而在requsstTouchState方法中会对FLAG_DISALLOW_INTERCEPT进行重置，因此子View调用requestDisallowInterceptTouchEvent方法并不能影响ViewGroup对ACTION_DOWN事件的处理：

```
 		// Handle an initial down.
            if (actionMasked == MotionEvent.ACTION_DOWN) {
                // Throw away all previous state when starting a new touch gesture.
                // The framework may have dropped the up or cancel event for the previous gesture
                // due to an app switch, ANR, or some other state change.
                cancelAndClearTouchTargets(ev);
                resetTouchState();
            }
```

>从上面的源码分析，我们可用得出结论，当ViewGroup决定拦截事件之后，那么后续的点击事件，将会默认交给他处理并且不再调用他的onInterceptTouchEvent方法，这就证实了我们刚才所说的标记位无效的理论，当然前提是ViewGroup不拦截ACTION_DOWN事件，接下来我们再看下ViewGroup不拦截事件的时候，事件会向下分发由他的子View进行处理：

```
						final View[] children = mChildren;
                        for (int i = childrenCount - 1; i >= 0; i--) {
                            final int childIndex = getAndVerifyPreorderedIndex(
                                    childrenCount, i, customOrder);
                            final View child = getAndVerifyPreorderedView(
                                    preorderedList, children, childIndex);

                            // If there is a view that has accessibility focus we want it
                            // to get the event first and if not handled we will perform a
                            // normal dispatch. We may do a double iteration but this is
                            // safer given the timeframe.
                            if (childWithAccessibilityFocus != null) {
                                if (childWithAccessibilityFocus != child) {
                                    continue;
                                }
                                childWithAccessibilityFocus = null;
                                i = childrenCount - 1;
                            }

                            if (!canViewReceivePointerEvents(child)
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
                                ev.setTargetAccessibilityFocus(false);
                                continue;
                            }

                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                // Child is already receiving touch within its bounds.
                                // Give it the new pointer in addition to the ones it is handling.
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }

                            resetCancelNextUpFlag(child);
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                                // Child wants to receive touch within its bounds.
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {
                                    // childIndex points into presorted list, find original index
                                    for (int j = 0; j < childrenCount; j++) {
                                        if (children[childIndex] == mChildren[j]) {
                                            mLastTouchDownIndex = j;
                                            break;
                                        }
                                    }
                                } else {
                                    mLastTouchDownIndex = childIndex;
                                }
                                mLastTouchDownX = ev.getX();
                                mLastTouchDownY = ev.getY();
                                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                alreadyDispatchedToNewTouchTarget = true;
                                break;
                            }
```

>上面的代码逻辑还是比较清晰的，首先遍历的是ViewGroup的所有子元素，然后判断子元素是否能接受这个点击事件主要是两点来衡量，子元素是否在播动画和点击是按的坐标是否落在子元素的区域内，如果某子元素满足这两个条件，那么事件就会传递给他处理，可以看到，dispatchTransformedTouchEvent实际上调用的就是子元素的dispatchTransformedTouchEvent方法，在他的内部有下面的这段话，而在上面的代码中child传递的不是null，因此他会直接调用子元素的dispatchTouchEvent方法，这样事件就交由子元素处理处理，这就从而完成这一轮事件分发。

```
        if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                handled = child.dispatchTouchEvent(event);
            }
```

>如果子元素的dispatchTouchEvent返回true，这时我们暂时不考虑事件在子元素的怎么分发的，那么mFirstTouchTarget就会被赋值同时跳出for循环：

```
newTouchTarget = addTouchTarget(child, idBitsToAssign);
alreadyDispatchedToNewTouchTarget = true;
break;
```
>这几行代码就完成了mFirstTouchTarget的赋值并且并终止对子元素的遍历，如果子元素的dispatchTouchEvent返回false，ViewGroup就会把事件分给下一个子元素

>其实mFirstTouchTarget真正的赋值过程是在addTouchTarget内部完成的，从下面的addTouchTarget的内部结构就可以看出，mFirstTouchTarget其实是一种单链表的结构，mFirstTouchTarget是否被赋值，将直接影响到ViewGroup对事件的拦截机制，如果mFirstTouchTarget为null，那么ViewGroup的默认拦截下来统一序列中所有的点击事件

```
private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
        final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
        target.next = mFirstTouchTarget;
        mFirstTouchTarget = target;
        return target;
    }
```

>如果遍历所有的子元素后事件都没有被合适的处理，这包含两种情况，第一是Viewgroup没有子元素，第二是子元素处理了点击事件，但是在
dispatchTouchEvent中返回false，这一版是因为子元素在onTouchEvent返回了false，这两种情况下，ViewGroup会自己处理点击事件，看代码：

```
 	// Dispatch to touch targets.
            if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } 
```

>注意上面这段话，这里的第三个参数child为null，从上面的分析我们可用知道，他会调用supe.dispatchTouchEvent(event)，很显然，这里就转到了View的dispatchTouchEvent方法，就是点击事件开始给View處理了

####3.View对点击事件的处理
>View对点击事件的处理稍微有点简单， 这里注意，这里的View不包含ViewGroup，先看他的dispatchTouchEvent方法

```
 	public boolean dispatchTouchEvent(MotionEvent event) {
        // If the event should be handled by accessibility focus first.
        if (event.isTargetAccessibilityFocus()) {
            // We don't have focus or no virtual descendant has it, do not handle the event.
            if (!isAccessibilityFocusedViewOrHost()) {
                return false;
            }
            // We have focus and got the event, then use normal event dispatch.
            event.setTargetAccessibilityFocus(false);
        }

        boolean result = false;

        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Defensive cleanup for new gesture
            stopNestedScroll();
        }

        if (onFilterTouchEventForSecurity(event)) {
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }

            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }

        if (!result && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }

        // Clean up after nested scrolls if this is the end of a gesture;
        // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
        // of the gesture.
        if (actionMasked == MotionEvent.ACTION_UP ||
                actionMasked == MotionEvent.ACTION_CANCEL ||
                (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
            stopNestedScroll();
        }

        return result;
    }
```
>View点击事件的处理，因为他只是一个View，他没有子元素所以无法向下传递，所以只能自己处理点击事件，从上门的源码可以看出View对点击事件的处理过程，首选会判断你有没有设置onTouchListener，如果onTouchListener中的onTouch为true，那么onTouchEvent就不会被调用，可见onTouchListener的优先级高于onTouchEvent，这样做到好处就是方便在外界处理点击事件;

>接着我们再来分析下onTouchEvent的实现，先看当View处于不可用的状态下点击事件的处理过程，如下，很显然，不可用状态下的View照样会消耗点击事件，尽管他看起来不可用：

```
	if ((viewFlags & ENABLED_MASK) == DISABLED) {
            if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                setPressed(false);
            }
            // A disabled view that is clickable still consumes the touch
            // events, it just doesn't respond to them.
            return (((viewFlags & CLICKABLE) == CLICKABLE
                    || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                    || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE);
        }

```

>接着，如果View设置有代理，那么还会执行TouchDelegate的onTouchEvent方法，这个onTouchEvent的工作机制看起来和onTouchListener类似，这里就不深究了

```
    if (mTouchDelegate != null) {
            if (mTouchDelegate.onTouchEvent(event)) {
                return true;
            }
        }
```

>下面再看一下onTouchEvent中点击事件的具体处理，如下所示：

```

        if (((viewFlags & CLICKABLE) == CLICKABLE ||
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE) ||
                (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE) {
            switch (action) {
                case MotionEvent.ACTION_UP:
                    boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                    if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                        // take focus if we don't have it already and we should in
                        // touch mode.
                        boolean focusTaken = false;
                        if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                            focusTaken = requestFocus();
                        }

                        if (prepressed) {
                            // The button is being released before we actually
                            // showed it as pressed.  Make it show the pressed
                            // state now (before scheduling the click) to ensure
                            // the user sees it.
                            setPressed(true, x, y);
                       }

                        if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                            // This is a tap, so remove the longpress check
                            removeLongPressCallback();

                            // Only perform take click actions if we were in the pressed state
                            if (!focusTaken) {
                                // Use a Runnable and post this rather than calling
                                // performClick directly. This lets other visual state
                                // of the view update before click actions start.
                                if (mPerformClick == null) {
                                    mPerformClick = new PerformClick();
                                }
                                if (!post(mPerformClick)) {
                                    performClick();
                                }
                            }
                        }
                 black;
             }
        ....
    return true;
    }
```

>从上面的代码来看，只要View的CLICKABLE和LONG_CLICKABLE有一个为tru，那么他就会消耗这个事件，即onTouchEvent返回true，不管他是不是DISABLE状态，这就证实了上面的第8,9,10结论，然后就是当ACTION_UP事件发生之后，会触发performClick方法，如果View设置了onClickListener，那么performClick方法内部就会调用他的onClick方法

```
	public boolean performClick() {
        final boolean result;
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
            playSoundEffect(SoundEffectConstants.CLICK);
            li.mOnClickListener.onClick(this);
            result = true;
        } else {
            result = false;
        }

        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
        return result;
    }

```

>View的LONG_CLICKABLE属性默认为false，而CLICKABLE属性是否为false和具体的View有关，确切的说是可点击的View其CLICKABLE为true，不可点击的为false，比如button是可点击的，textview是不可点击的，通过setonclik或者longclik都是可以改变状态的，这点我们看源码：

```
 public void setOnClickListener(@Nullable OnClickListener l) {
        if (!isClickable()) {
            setClickable(true);
        }
        getListenerInfo().mOnClickListener = l;
    }
    
    public void setOnLongClickListener(@Nullable OnLongClickListener l) {
        if (!isLongClickable()) {
            setLongClickable(true);
        }
        getListenerInfo().mOnLongClickListener = l;
    }

```

>到这里，点击事件的分发机制源码就分析完成了，结合滑动事件可能理解的更好一点

##二.View的滑动冲突
>这节要说的是滑动冲突，也是很多人碰到过的，也比较麻烦的，滑动冲突时怎样产生的？其实在界面中只要内外两层同时可以滑动，这个时候就会产生滑动冲突，如何解决滑动冲突？这既是一键困难的事情又是一件简单的事情，说困难时因为许多开发者面对滑动冲突都会显得束手无策，说简单是因为滑动冲突的解决办法有固定的套路，只要知道这个套路就好了

###1.常见的滑动冲突场景
>常见的滑动一般有三个方面

- 外部滑动方向和内部滑动方向不一致
- 外部滑动方向和内部滑动方向一致
- 上面两种情况的嵌套

---图片

>先说场景1，主要是将ViewPager和Fragment配合使用所组成的页面滑动效果，主流应用几乎都会使用这个效果。在这种效果中，可以通过左右滑动来切换页面，而每个页面内部往往又是一个Listview。本来这种情况下是有滑动冲突的，但是viewPager内部处理了这种滑动冲突，因此采用ViewPager时我们无须关注这个问题，如果我们采用的不是ViewPager而是ScrollView等，那就必须手动处理滑动冲突了，
否则造成的后果就是内外两层只能有一层能够滑动，这是因为两者之间的滑动事件有冲突。除了这种典型情况外，还存在其他情况，比如外部上下滑动、内部左右滑动等，但是它们属于同一类滑动冲突。

>再说场景2，这种情况就稍微复杂一些，当内外两层都在同一个方向可以滑动的时候，显然存在逻辑问题。因为当手指开始滑动的时候，系统无法知道用户到底是想让哪一层滑动，所以当手指滑动的时候就会出现问题，要么只有一层能滑动，要么就是内外两层都滑动得很卡顿。在实际的开发中，这种场景主要是指内外两层同时能上下滑动或者内外两层，同时能左右滑动。

>最后说下场景3，场景3是场景1和场景2两种情况的嵌套，因此场景3的滑动冲突看起来就更加复杂了。比如在许多应用中会有这么一个效果：内层有一个场景1中的滑动效果，然后外层又有一个场景2中的滑动效果。具体说就是，外部有一个SlidingMenu效果，然后内部有一个ViewPager,ViewPager的每一个页面中又是一个Listview。虽然说场景3的滑动冲突看起来更复杂，但是它是几个单一的滑动冲突的叠加，因此只需要分别处理内层和中层、中层和外层之间的滑动冲突即可，而具体的处理方法其实是和场景1、场景2相同的。

>从本质上来说，这三种滑动冲突场景的复杂度其实是相同的，因为它们的区别仅仅是滑动策略的不同，至于解决滑动冲突的方法，它们几个是通用的

###2.滑动冲突的处理规则
>一般来说，不管滑动冲突多么复杂，它都有既定的规则，根据这些规则我们就可以选择合适的方法去处理。

>如上图所示，对于场景1，它的处理规则是：当用户左右滑动时，需要让外部的View拦截点击事件，当用户上下滑动时，需要让内部View拦截点击事件。这个时候我们就可以根据它们的特征来解决滑动冲突，具体来说是：根据滑动是水平滑动还是竖直滑动来判断到底由谁来拦截事件，如下图所示，根据滑动过程中两个点之间的坐标就可以得出到到底由谁来拦截事行；如何根据坐标来获取滑动的方向呢？这个很简单，有很多可以参考，比如可以依据滑动路径和水平方向做形成的夹角，也可以依据水平方向和竖直方向上的距离差来判断，某些特殊时候还可以依据水平和竖直方向的速度差来做判断。这里我们可以通过水平和竖直方向的距离差来判断，比如竖直方向滑动的距离大就判断为竖直滑动。否则判断为水平滑动。根据这个规则就可以进行下一步的解决方法制定了。

>对于场景2来说，比较特殊，它无法根据滑动的角度、距离差以及速度差来做判断，但是这个时候一般都能在业务上找到突破点，比如业务上有规定：当处于某种状态时需要外部View响应用户的滑动，而处于另外一种状态时则需要内部View来响应View的滑动,根据这种业务上的需求我们也能得出相应的处理规则，有了处理规则同样可以进行下一步处理。这种场景通过文字描述可能比较抽象，在下一节会通过实际的例子来演示这种情况的解决方案，那时就容易理解了，这里先有这个概念即可

-----图

>对于场景3来说，它的滑动规则就更复杂了，和场景2一样，它也无法直接根据滑动的角度、距离差以及速度差来做判断，同样还是只能从业务上找到突破点，具体方法和场景2一样，都是从业务的需求上得出相应的处理规则，在下一节将会通过实际的例子来说明这种情况的解决方案。

###3.滑动冲突的解决方式

>在上节中描述了三种典型的滑动冲突场景，在本节将会一一分析各种场景并给出其中解决方法。首先我们要分析第一种滑动冲突场景，这也是最简单、最典型的一种滑动冲突，因为它的滑动规则比较简单，不管多复杂的滑动冲突，它们之间的区别仅仅是滑动的规则不同而已。抛开滑动规则不说，我们需要找到一种不依赖具体的滑动规则的解决办法，，在这里，我们就根据场景1的情况来得出通用的解决方案，然后场景2和场景3我们只需要修改有关滑动规则的逻辑即可。

>上面说过，针对场景1中的滑动，我们可以根据滑动的距离差来进行判断，这个距离差就是我们的的滑动规则。如果用ViewPager去实现场景中的效果，我们不需要手动处理滑动冲突，因为ViewPager已经帮我们做了，但是我们这里为了更好的演示滑动冲突的解决思想，我们并没有采用ViewPager，其中在滑动过程中得到滑动的角度是很简单的，但是到底要怎么做才能将点击事件交给合适的View去做呢？这里可以用到事件分发，给出两个解决办法

####1.外部拦截法
>所谓的外部拦截费是指点击事件都先经过父容器的拦截处理，如果父容器需要这个事件就给给他，这里我们还得重写我们的onIterceptTouchEvent，如下：

```
	@Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean intercepted = false;
        int x = (int) ev.getX();
        int y = (int) ev.getY();
        switch (ev.getAction()){
            case MotionEvent.ACTION_DOWN:
                intercepted = false;
                break;
            case MotionEvent.ACTION_MOVE:
                if("父容器的点击事件"){
                    intercepted = true;
                }else {
                    intercepted = false;
                }
                break;
            case MotionEvent.ACTION_UP:
                intercepted = false;
                break;
        }
        mLastXIntercept = x;
        mLastYIntercept = x;
        return intercepted;
    }

```

>上述代码是外部拦截法的典型逻辑，针对不同的滑动冲突，只需要修改父容器需要当前点击事件这个条件即可，其他均不需做修改并且也不能修改。这里对上述代码再描述一下，在onInterceptTouchEvent方法中，首先是ACTION_DOWN这个事件，父容器必须返回false，即不拦截ACTION_DOWN事件，这是因为一旦父容器拦截了ACTION_DOWN，那么后续的ACTION_MOVE和ACTION_UP事件都会直接交由父容器处理，这个时候事件没法再传递给子元素了；其次是ACTION_MOVE事件，这个事件可以根据需要来决定是否拦截，如果父容器需要拦截就返回true，否则返回false；最后是ACTION_UP事件，这里必须要返回false，因为ACTION_UP事件本身没有太多意义考虑一种情况，假设事件交由子元素处理，如果父容器在ACTION_UP时返回了true，会导致子元素无法接收到ACTION_UP事件，这个时候子元素中的onClick事件就无法触发，但是父容器比较特殊，一旦它开始拦截任何一个事件，那么后续的事件都会交给它处理，而ACTION_UP作为最后一个事件也必定可以传递给父容器，即便父容器的onInterceptTouchEvent方法在ACTION_UP时返回了false。

####2.内部拦截法

>内部拦截法是指父容器不拦截任何事件，所有的事件都传递给子元素，如果子元素要消耗此事件就直接消耗掉，否则就交由父容器进行处理，这种方法和Android中的事件分发机制不一致，需要配合requestDisallowInterceptTouchEvent方法才能正常工作，使用起来较外部拦截法稍显复杂。它的伪代码如下，我们需要重写子元素的dispatchTouchEvent方法：

```
 	@Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                getParent().requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_MOVE:
                int deltaX =  x - mLastX;
                int deltaY =  x - mLastY;
                if("父容器的点击事件"){
                    getParent().requestDisallowInterceptTouchEvent(false);
                }
                break;
            case MotionEvent.ACTION_UP:

                break;
        }
        mLastX = x;
        mLastY = y;
        return super.dispatchTouchEvent(event);
    }
```

>上述代码就是内部拦截法的典型代码，当面对不同的滑动策略只需要修改里面的条件即可，其他不需要做改动，除了子元素需要处理之外，父元素默认也要拦截除ACTION_DOWN之外的其他事件，这样当子元素调用getParent().requestDisallowInterceptTouchEvent(true)方法时，父元素才能继续拦截所需要的事件

>为什么父容器不能拦截ACTION_DOWN事件呢？那是因为ACTION_DOWN事件并接受FLAG_DISALLOW_DOWN这个标记位的控制，所以一旦父容器拦截，那么所有的事件都无法传递到子元素中，这样额你不拦截就无法起作用了，父元素要做的如下修改

```
	@Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        int action = ev.getAction();
        if(action == MotionEvent.ACTION_DOWN){
            return false;
        }else {
            return true;
        }
    }

```

>下面通过一个实例来分别接受这两种用法，我们来实现一个类似于ViewPgaer中嵌套ListView的效果，为了制造滑动冲突，我们写一个类似ViewPager的控件即可，名字叫做HorizontalScrollViewEx，这个控件的具体实现思想会在第四章进行详细介绍，这里我只说滑动冲突部分

>为了实现ViewPager的效果，我们定义一个类似于水平的线性布局，只不过他可以水平滑动，初始化时，我们可以再他的内部添加诺干个ListView，这样一来，由于ListView是可以竖直滑动的，就造成了典型的滑动冲突，先看下Activity的实现：


```
public class MainActivity extends AppCompatActivity {

    public static final String TAG = "MainActivity";
    private HorizontalScrollViewEx mListContainer;
    private int w,h;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Log.i(TAG,"onCreate");
        initView();
    }

    private void initView() {
        LayoutInflater inflater = getLayoutInflater();
        mListContainer = findViewById(R.id.container);
        //屏幕宽高
        WindowManager wm = (WindowManager) getSystemService(WINDOW_SERVICE);
        w = wm.getDefaultDisplay().getWidth();
        h = wm.getDefaultDisplay().getHeight();
        for (int i = 0; i < 3; i++) {
            ViewGroup layout = inflater.inflate(R.layout.content_layout,mListContainer,false);
            layout.getLayoutParams().width = w;
            TextView textview = (TextView) layout.findViewById(R.id.title);
            textview.setText("page"  + (i+1));
            layout.setBackgroundColor(Color.rgb(255/(i+1),255/(i+1),0));
            createList(layout);
            mListContainer.addView(layout);
        }
    }

    private void createList(ViewGroup layout) {
        ListView listview = (ListView) layout.findViewById(R.id.list);
        ArrayList<String>datas= new ArrayList<>();
        for (int i = 0; i < 50; i++) {
            datas.add("names" + i);
        }
        ArrayAdapter<String>adapter = new ArrayAdapter<String>(this,R.layout.content_list_item,R.id.name,datas);
        listview.setAdapter(adapter);
    }
}
```

>上述的代碼很簡單，就是创建了三个ListView，然后添加进去

>首先我们采用外部拦截法来解决这个问题，按照之前的分析，我们只需要修改父容器需要拦截的条件即可，对于本例来说，父容器的拦截条件就是滑动过程中水平距离差比竖直距离差要大，在这种情况下，父容器就拦截当前点击事件，根据这一个条件进行修改，我们先来看他的拦截事件：

```
	@Override
    public boolean onInterceptHoverEvent(MotionEvent event) {
        boolean intercepted = false;
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                intercepted = true;
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                    intercepted = true;
                }
                break;
            case MotionEvent.ACTION_MOVE:
                int deltax = x - mLastXIntercept;
                int deltaY = y = mLastYIntercept;
                if (Math.abs(deltax) > Math.abs(deltaY)) {
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

```

>从上面的代码可以看出，他和外部拦截的伪代码差别很小，只是把父容器的拦截条件换成了实际的逻辑，在滑动过程中，当水平方向的距离大就判断水平滑动，为了能够水平滑动所以让父容器拦截事件，而竖直距离大于就不拦截，于是事件传递给了ListView，所以ListView能上下滑动，这就解决了冲突了

>考虑到一种情况，如果此时用户正在水平滑动，但是水平滑动停止之前如果用户再迅速的进行竖直滑动，就会导致界面在水平滑动无法滑动到终点，而处于一种中间状态，为了避免这种不友好的体验，我们水平正在滑动的时候，下一个序列的点击仍然交给父容器，具体我们看代码

```
package com.liuguilin.viewsample;

/*
 *  项目名：  ViewSample 
 *  包名：    com.liuguilin.viewsample
 *  文件名:   HorizontalScrollViewEx
 *  创建者:   LGL
 *  创建时间:  2016/11/5 16:43
 *  描述：    TODO
 */

import android.content.Context;
import android.view.MotionEvent;
import android.view.VelocityTracker;
import android.view.ViewGroup;
import android.widget.Scroller;

public class HorizontalScrollViewEx extends ViewGroup {

    public static final String TAG = "HorizontalScrollViewEx";

    private int mChindrensize;
    private int mChindrenWidth;
    private int mChindrenIndex;
    //分别记录上次滑动的坐标
    private int mLastX = 0;
    private int mLastY = 0;
    //分别记录上次滑动的坐标
    private int mLastXIntercept = 0;
    private int mLastYIntercept = 0;

    private Scroller mScroller;
    private VelocityTracker mVelocityTracker;

    private void init() {
        mScroller = new Scroller(getContext());
        mVelocityTracker = VelocityTracker.obtain();
    }

    public HorizontalScrollViewEx(Context context) {
        super(context);
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {

    }

    @Override
    public boolean onInterceptHoverEvent(MotionEvent event) {
        boolean intercepted = false;
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                intercepted = true;
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                    intercepted = true;
                }
                break;
            case MotionEvent.ACTION_MOVE:
                int deltax = x - mLastXIntercept;
                int deltaY = y = mLastYIntercept;
                if (Math.abs(deltax) > Math.abs(deltaY)) {
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
                int deltaX = x - mLastX;
                int deltaY = y - mLastY;
                scrollBy(-deltaX, 0);
                break;
            case MotionEvent.ACTION_UP:
                int scrollX = getScrollX();
                int scrollToChildIndex = scrollX / mChindrenWidth;
                mVelocityTracker.computeCurrentVelocity(1000);
                float xVelocity = mVelocityTracker.getXVelocity();
                if (Math.abs(xVelocity) >= 50) {
                    mChindrenIndex = xVelocity > 0 ? mChindrenIndex - 1 : mChindrenIndex + 1;
                } else {
                    mChindrenIndex = (scrollX + mChindrenWidth / 2) / mChindrenWidth;
                }
                mChindrenIndex = Math.max(0, Math.min(mChindrenIndex, mChindrensize - 1));
                int dx = mChindrenIndex * mChindrenWidth - scrollX;
                ssmoothScrollBy(dx, 0);
                mVelocityTracker.clear();
                break;
        }
        mLastX = x;
        mLastY = y;
        return true;
    }

    private void ssmoothScrollBy(int dx, int i) {
        mScroller.startScroll(getScrollX(),0,dx,500);
        invalidate();
    }

    @Override
    public void computeScroll() {
        if(mScroller.computeScrollOffset()){
            scrollTo(mScroller.getCurrX(),mScroller.getCurrY());
            postInvalidate();
        }
    }
}


```

>如果采用内部拦截法也是可以的，按照前面对内部拦截法的分析，我们只需要修改ListView的dispatchTouchEvent方法中的父容器的拦截逻辑，同时让父拦截MOVE和Up事件即可。为了重写Listview的dispatchTouchfvent方法，我们必须自定义一个ListView，称为ListViewEx，然后对内部拦截法的模板代码进行修改，根据需要，ListViewEx的实现如下所示：

```
public class ListViewEx extends ListView {

    public static final String TAG = "ListViewEx";
    private HorizontalScrollViewEx mHorizontalScrollViewEx;

    private int mLastX = 0;
    private int mLastY = 0;

    public ListViewEx(Context context) {
        super(context);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        int x = (int) ev.getX();
        int y = (int) ev.getY();
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mHorizontalScrollViewEx.requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_MOVE:
                int delatX = x - mLastX;
                int delatY = y - mLastY;
                if (Math.abs(delatX) > Math.abs(delatY)) {
                    mHorizontalScrollViewEx.requestDisallowInterceptTouchEvent(false);
                }
                break;
        }
        mLastX = x;
        mLastY = y;
        return super.dispatchTouchEvent(ev);
    }
}


```

>除了对ListView的修改，我们还需要修改HorizontalScrollViewEx的事件分发

```
 @Override
    public boolean onInterceptHoverEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        int action = event.getAction();
        if(action == MotionEvent.ACTION_DOWN){
            mLastX = x;
            mLastY = y;
            if(!mScroller.isFinished()){
                mScroller.abortAnimation();
                return true;
            }
            return false;
        }else {
            return true;
        }
    }

```

>上面的代码就是内部拦截法的示例，其中mScrollerabortAnimationO这一句不是必须的，在当前这种情形下主要是为了优化滑动体验。从实现上来看，内部拦截法的操作要稍微复杂一些，因此推荐采用外部拦截法来解决常见的滑动冲突。

>前面说过，只要我们根据场景1的情况来得出通用的解决方案，那么对于场景2和场景3来说我们只需要修改相关滑动规则的逻辑即可，下面我们就来演示如何利用场景1得出的通用的解决方案来解决更复杂的滑动冲突。这里只详细分析场景2中的滑动冲突，对于场景3中的叠加型滑动冲突，由于它可以拆解为单一的滑动冲突，所以其滑动冲突的解题思想和场景1、场景2中的单一滑动冲突的解决思想一致，只需要分别解决每层之间的滑动冲突即可，再加上本书的篇幅有限，这里就不对场景3进行详细分析了。

>对于场景2来说，它的解决方法和场景1一样，只是滑动规则不同而已，在前面我们己经得出了通用的解决方案，因此这里我们只需要替换父容器的拦截规则即可。注意，这里不再演示如何通过内部拦截法来解决场景2中的滑动冲突，因为内部拦截法没有外部拦截法简单易用，所以推荐采用外部拦截法来解决常见的滑动冲突。

>下面通过一个实际的例子来分析场景2，首先我们提供一个可以上下滑动的父容器，这里就叫 StickyLayout，它看起来就像是可以上下滑动的竖直的LinearLayout，然后在它的内部中的滑动冲突了。当然这个StickyLayout是有滑动规则的：当Header显示时或者ListView
滑动到顶部时，由StickyLayout拦截事件：当Header隐藏时，这要分情况，如果Listview已经滑动到顶部并且当前手势是向下滑动的话，这个时候还是StickyLayout拦截事件，其他情况则由ListView拦截事件。这种滑动规则看起来有点复杂，为了解决它们之间的滑动冲突，我们还是需要重写父容器StickyLayout的onintercepTouchEvent方法，至于ListView则不用做任何修改，我们来看一下StickyLayout的具体实现，滑动冲突相关的主要代码：

```
public class StickyLayout extends LinearLayout {

    private int mTouchSlop;
    private int mLastX = 0;
    private int mLastY = 0;

    private int mLastXIntercept = 0;
    private int mLastYIntercept = 0;

    public StickyLayout(Context context) {
        super(context);
    }

    @Override
    public boolean onInterceptHoverEvent(MotionEvent event) {
        int intercepted = 0;
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mLastXIntercept = x;
                mLastYIntercept = y;
                mLastX = x;
                mLastY = y;
                intercepted = 0;
                break;
            case MotionEvent.ACTION_MOVE:
                int deltaX = x - mLastXIntercept;
                int deltaY = y - mLastYIntercept;
                if (mDisallowInterceptTouchEventOnHeader && y <= getHeaderHeight()) {
                    intercepted = 0;
                } else if (Math.abs(deltaY) <= Math.abs(deltaX)) {
                    intercepted = 0;
                } else if (mStatus == STATUS_EXPANDED && deltaY <= -mTouchSlop) {
                    intercepted = 1;
                } else if (mGiveUpTouchEventListener != null) {
                    if (mGiveUpTouchEventListener.giveUpTouchEvent(event) && deltaY >= mTouchSlop) {
                        intercepted = 1;
                    }
                }
                break;
            case MotionEvent.ACTION_UP:
                intercepted = 0;
                mLastYIntercept = mLastYIntercept = 0;
                break;
        }
        return intercepted != 0 && mIsSticky;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (!mIsSticky) {
            return true;
        }
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:

                break;
            case MotionEvent.ACTION_MOVE:
                int deltaX = x - mLastX;
                int deltaY = y - mLastY;
                mHeaderHeight += deltaY;
                setHeaderHeight(mHeaderHeight):
                break;
            case MotionEvent.ACTION_UP:
                int destHeight = 0;
                if (mHeaderHeight <= mOriginalHeaderHeight * 0.5) {
                    destHeight = 0;
                    mStatus = STATUS_COLLAPSED;
                } else {
                    destHeight = mOriginalHeaderHeight;
                    mStatus = STATUS_EXPANDED;
                }
                this.smoothSetHeaderHeight(mHeaderHeight, destHeight, 500);
                break;
        }
        mLastX = x;
        mLastY = y;
        return true;
    }
}


```

>从上面的代码中，我们看到这个类实现起来还是有点复杂的，我们再下一章会详细的介绍一下View的实现思想，这里大概有一个印象就好，来看下思路：

```
            case MotionEvent.ACTION_MOVE:
                int deltaX = x - mLastXIntercept;
                int deltaY = y - mLastYIntercept;
                if (mDisallowInterceptTouchEventOnHeader && y <= getHeaderHeight()) {
                    intercepted = 0;
                } else if (Math.abs(deltaY) <= Math.abs(deltaX)) {
                    intercepted = 0;
                } else if (mStatus == STATUS_EXPANDED && deltaY <= -mTouchSlop) {
                    intercepted = 1;
                } else if (mGiveUpTouchEventListener != null) {
                    if (mGiveUpTouchEventListener.giveUpTouchEvent(event) && deltaY >= mTouchSlop) {
                        intercepted = 1;
                    }
                }


```

>我们来分析下上面这段话的逻辑，这里的父容器是LAayout，子元素是ListView，首先，当时间落在Header上面时，就不会拦截该事件，接着，如果竖直距离差小于水平距离差，那么父容器野不会拦截该事件，然后，当Header是展开状态并且是向上滑动时父容器拦截事件，，当ListView滑动到顶部并且向下滑动的时候，父容器也会拦截事件，经过这层层的判断，我们就可以达到我们想要的效果，另外，giveUpTouchEvent是一个接口方法，具体实现如下：

```
private boolean giveUpTouchEvent(MotionEvent event) {
        if (expandableListView.getFirstVisiblePosition() == 0) {
            View view = expandableListView.getChildAt(0);
            if (view != null && view.getTop() >= 0) {
                return true;
            }
        }
        return false;
    }

```

































