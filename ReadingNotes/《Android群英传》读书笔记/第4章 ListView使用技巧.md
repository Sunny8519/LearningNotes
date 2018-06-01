## 第2章 ListView使用技巧

### 1. ListView常用的优化技巧

- 使用ViewHolder模式提高效率

在继承BaseAdapter的Adapter类中，实现一个ViewHolder的内部类，该内部类中保存Item中用到的控件引用，这样在Adapter中的getView()方法中不用每次都findViewById()了，可以从ViewHolder中获取缓存的控件。

缓存多少个Item，就会缓存相应个数的ViewHolder，随着这些Item的重复使用，ViewHolder也是被重复使用的。

```java
public class ListViewAdapter extends BaseAdapter {
    private List<String> mDatas;
    private LayoutInflater mInflater;

    public ListViewAdapter(@NonNull Context context, @NonNull List<String> datas) {
        this.mDatas = datas;
        this.mInflater = LayoutInflater.from(context);
    }

    @Override
    public int getCount() {
        return this.mDatas.size();
    }

    @Override
    public Object getItem(int position) {
        return this.mDatas.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder viewHolder;
        if (convertView == null) {
            viewHolder = new ViewHolder();

            convertView = this.mInflater.inflate(R.layout.item_list_view, parent, false);
            viewHolder.tvText = convertView.findViewById(R.id.tv_text);
            convertView.setTag(viewHolder);
        } else {
            viewHolder = (ViewHolder) convertView.getTag();
        }
        viewHolder.tvText.setText(this.mDatas.get(position));

        return convertView;
    }

    class ViewHolder {
        TextView tvText;
    }
}
```

### 2. ListView设置分割线

ListView的分割线可以通过两个属性来控制：

```java
android:divider="@android:color/darker_gray"
android:dividerHeight="10dp"
```

`android:divider`可以设置颜色，也可以设置图片资源，如果把该属性值设为`android:divider="@null"`，分割线就是透明的，当然你也可以设置一个透明色。

### 3. 隐藏ListView的滚动条

```java
//设置属性
android:scrollbars="none"
```

### 4. 取消ListView的Item点击效果

点击ListView中的一项时，系统默认会出现一个点击效果，在5.x以上是水波纹，在5.x以下则是一个改变背景颜色的效果，我们可以通过listSelector属性来取消或者改变点击效果。

```java
android:listSelector="#00000000"
```

### 5. 设置显示某一个指定Item

设置显示某个指定Item可通过如下代码：

```java
listView.setSelection(N);//N是第N个Item
```

这个方法的显示效果类似于scrollTo()，是瞬间完成的，没有过度的动画。要实现平滑移动可通过如下等方法：

```java
listView.smoothScrollByOffset(offset);
listView.smoothScrollToPosition(position);
```

### 6. 动态更新ListView

```java
dataList.add("new");
adapter.notifyDataSetChanged();
```

唯一要注意的一点是数据集dataList要和第一次设置进adapter是同一个对象。

### 7. 获取ListView中指定的Item

```java
View view = mListView.getChildAt(i);
```

我们都知道ViewGroup中有`getChildAt()`方法，ListView也是一个ViewGroup，理所当然的，它也有该方法。

### 8. 处理空ListView

ListView在无数据的时候不会有任何页面展示，为了增加用户体验，我们通常需要添加一个无数据时的展示页面，ListView提供了一个方法能够很方便的做到这一点：`setEmptyView()`

### 9. ListView的滑动监听

```java
mListView.setOnScrollListener(new AbsListView.OnScrollListener() {
            @Override
            public void onScrollStateChanged(AbsListView view, int scrollState) {
                switch (scrollState) {
                    case SCROLL_STATE_IDLE:
                        Log.i(TAG, "SCROLL_STATE_IDLE");
                        break;
                    case SCROLL_STATE_TOUCH_SCROLL:
                        Log.i(TAG, "SCROLL_STATE_TOUCH_SCROLL");
                        break;
                    case SCROLL_STATE_FLING:
                        Log.i(TAG, "SCROLL_STATE_FLING");
                        break;
                    default:
                        break;
                }
            }

            @Override
            public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
                Log.i(TAG, "onScroll is called");
            }
        });
```

在OnScrollListener接口中只有两个方法，分别为onScrollStateChanged()和onScroll()，在ListView滑动过程中，onScrollStateChanged()方法最多只会被调用三次，通常情况下会调用两次，调用两次时，方法参数scrollState的值分别为SCROLL_STATE_TOUCH_SCROLL和SCROLL_STATE_IDLE，SCROLL_STATE_TOUCH_SCROLL表示正在滚动，SCROLL_STATE_IDLE表示滑动停止。调用三次的情况就是多了一个SCROLL_STATE_FLING，该值表示手指在快速滑动后ListView由于惯性继续滚动。onScroll()方法在ListView滚动的过程中会一直被调用。firstVisibleItem表示当前可见的第一个Item，visibleItemCount表示当前可见的Item数量，totalItemCount表示ListView整个Item的数量。

### 10. ListView拓展

#### 10.1 实现弹性ListView

什么是弹性ListView呢？就是在ListView滑动到顶部或者底部时再向上或者向下滑动时会有一个弹性的效果，一放手又回到底部或者底部的位置，IOS中的列表基本都有这种效果，这种交互明显要更好。

实现弹性ListView方案一：

```java
//覆写overScrollBy()方法
@Override
protected boolean overScrollBy(int deltaX, int deltaY, int scrollX, int scrollY, int scrollRangeX, int scrollRangeY, int maxOverScrollX, int maxOverScrollY, boolean isTouchEvent) {
        return super.overScrollBy(deltaX, deltaY, scrollX, scrollY, scrollRangeX, scrollRangeY, maxOverScrollX, dpToPx(50)/*修改该处的值就可实现弹性滑动*/, isTouchEvent);
}
```

上面这种方式虽然能够实现ListView的弹性滑动，但是效果不好，亲测有bug，bug现象：当我们手指向上或者向下一次性滑动到底部时，由于弹性效果多出来的一部分不能自动恢复。

#### 10.2 自动显示、隐藏布局的ListView

实现这一效果

