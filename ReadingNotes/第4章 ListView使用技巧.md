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