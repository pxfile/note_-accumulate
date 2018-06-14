RecyclerView 和 ListView
===


### 布局效果对比
ListView只有一种滑动布局
RecyclerView通过布局管理器能展示多种布局

### API 使用对比
ListView 的基础使用大家再熟悉不过，其使用的关键点主要如下：

*   继承重写 BaseAdapter 类
*   自定义 ViewHolder 和 convertView 一起完成复用优化工作

由于 ListView 已经老生常谈，所以此处就不去写示例代码了。 RecyclerView 基础使用关键点同样有两点：

*   继承重写 RecyclerView.Adapter 和 RecyclerView.ViewHolder
*   设置布局管理器，控制布局效果

### 空数据处理

* ListView 提供了 setEmptyView 这个 API 来让我们处理 Adapter 中数据为空的情况，只需轻轻一 set 就能搞定一切。代码设置和效果如下

```

mListView = (ListView) findViewById(R.id.listview);
        mListView.setEmptyView(findViewById(R.id.empty_layout));//设置内容为空时显示的视图

```
* RecyclerView没有相应的API，需要自己动手去完成

### HeaderView 和 FooterView

* 在 ListView 的设计中，存在着 HeaderView 和 FooterView 两种类型的视图，并且系统也提供了相应的 API 来让我们设置

* RecyclerView没有相应的API，需要自己动手去完成

### 局部刷新
* ListView 并没有提供局部刷新刷新某个 Item 的 API 给我们，同样自己自足，套路大致如下方的 updateItemView：

```

public class AuthorListAdapter extends BaseAdapter {

    ...

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ...
        return convertView;
    }

    /**
     * 更新Item视图，减少不必要的重绘
     *
     * @param listView
     * @param position
     */
    public void updateItemView(ListView listView, int position) {
        //换算成 Item View 在 ViewGroup 中的 index
        int index = position - listView.getFirstVisiblePosition();
        if (index >= 0 && index < listView.getChildCount()) {
            //更新数据
            AuthorInfo authorInfo = mAuthorInfoList.get(position);
            authorInfo.setNickName("Google Android");
            authorInfo.setMotto("My name is Android .");
            authorInfo.setPortrait(R.mipmap.ic_launcher);
            //更新单个Item
            View itemView = listView.getChildAt(index);
            getView(position, itemView, listView);
        }
    }

}

```

即可实现刷新单个 Item 的效果

[RecyclerView.Adapter](https://link.jianshu.com/?t=https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html) 则我们提供了 notifyItemChanged 用于更新单个 Item View 的刷新，我们可以省去自己写局部更新的工作。

![](https://upload-images.jianshu.io/upload_images/912181-b8051dce7450291e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

### 动画效果

* 作为 ListView 自身并没有为我们提供封装好的 API 来实现动画效果切换。所以，如果要给 ListView 的 Item 加动画，我们只能自己通过属性动画来操作 Item 的视图。 Github 也有很多封装得好好的开源库给我们用，如：[ListViewAnimations](https://link.jianshu.com?t=https://github.com/nhaarman/ListViewAnimations)

* RecyclerView 则为我们提供了很多基本的动画 API ，如下方的**增删移改**

![](https://upload-images.jianshu.io/upload_images/912181-af00cdaa75008b0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

简单的调用即可实现相应的效果，用起来方便很多，视觉交互上也会更好些

### 监听 Item 的事件
* ListView 为我们准备了几个专门用于监听 Item 的回调接口，如单击、长按、选中某个 Item 等

![](https://upload-images.jianshu.io/upload_images/912181-eb82b15ab33d5bc8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

* RecyclerView ，它并没有像 ListView 提供太多关于 Item 的某种事件监听，唯一的就是 addOnItemTouchListener

### 嵌套滚动机制
RecyclerView支持嵌套滑动
ListView不支持

## RecyclerView与ListView的滑动对比

* RecyclerView的滑动

RecyclerView的滑动过程可以分为2个阶段：手指在屏幕上移动，使RecyclerView滑动的过程，可以称为scroll；手指离开屏幕，RecyclerView继续滑动一段距离的过程，可以称为fling。

当RecyclerView接收到ACTION_MOVE事件后，会先计算出手指移动距离（dy），并与滑动阀值（mTouchSlop）比较，当大于此阀值时将滑动状态设置为SCROLL_STATE_DRAGGING，而后调用scrollByInternal()方法，使RecyclerView滑动，这样RecyclerView的滑动的第一阶段scroll就完成了；当接收到ACTION_UP事件时，会根据之前的滑动距离与时间计算出一个初速度yvel，这步计算是由VelocityTracker实现的，然后再以此初速度，调用方法fling()，完成RecyclerView滑动的第二阶段fling。

* ListView的滑动