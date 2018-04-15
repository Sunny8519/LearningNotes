## View绘制原理之LayoutInflater

#### 一. LayoutInflater的使用

在说明LayoutInflater的原理之前，我们以一个应用场景为例：现在需要给一个xml布局的根标签设置固定宽高，然后把该布局的View加入到另一个容器中显示。哎，挺简单的嘛，我们可以这样实现：

这是父容器xml布局(根布局背景为白色)：

```java
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <android.support.constraint.ConstraintLayout
        android:id="@+id/content"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@android:color/white">
    </android.support.constraint.ConstraintLayout>

</layout>
```

这是即将加入到父容器中的View的xml布局(根布局背景为蓝色)：

```java
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <android.support.constraint.ConstraintLayout
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:background="@color/primary">
    </android.support.constraint.ConstraintLayout>

</layout>
```

这是Java代码：解析子View的布局并加入到父容器中

```java
    //Activity的onCreate()方法
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ActivityLayoutInflaterBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_layout_inflater); //设置Activity的布局
        
        View childView = LayoutInflater.from(this).inflate(R.layout.view_child, null, false); //解析子View的布局
        binding.content.addView(childView); //把子View加到父容器中
    }
```

好，我们来看看显示效果：

<img src="https://upload-images.jianshu.io/upload_images/5231076-b65d0320f3fb9d2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="50%" height="50%">

咦，怎么回事，为什么设置的200x200的固定尺寸没有生效呢，怎么是全屏的蓝色？？这和我们期望的显示效果不一致啊，这时我们可能想到一个办法：在子View的外面再包一层ViewGroup，让它是match_parent的，可能就好了，我们试试。

```java
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.constraint.ConstraintLayout
            android:layout_width="200dp"
            android:layout_height="200dp"
            android:background="@color/primary">
        </android.support.constraint.ConstraintLayout>
    </FrameLayout>
</layout>
```

果真如我们期望的，显示正确了：

<img src="https://upload-images.jianshu.io/upload_images/5231076-f4e5f2234b3056ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="50%" height="50%">

但是我们有没有想过一个问题，这样在子View的的布局外面再套一层ViewGroup真的是必要的吗？说白了，我们还是没有找到问题的根源在哪里，恰巧这样的方式解决了我们遇到的问题。

其实，问题的根源就在这一行代码上：`View childView = LayoutInflater.from(this).inflate(R.layout.view_child, null, false);`，我们可以试着把这行代码中的inflate()方法的第二个参数的null，改为`binding.content`或者``：

```java
@Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ActivityLayoutInflaterBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_layout_inflater); //设置Activity的布局

        View childView = LayoutInflater.from(this).inflate(R.layout.view_child, binding.content, false); //解析子View的布局
        binding.content.addView(childView); //把子View加到父容器中
    }
```

再来看看显示效果：

<img src="https://upload-images.jianshu.io/upload_images/5231076-f4e5f2234b3056ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="50%" height="50%">

也可以正确的显示，那么这行代码到底有什么玄机呢？我们来看一看。

#### 二. 两参inflate()方法和三参inflate()方法

```java
//两参inflate()方法
public View inflate(XmlPullParser parser, @Nullable ViewGroup root) {
        return inflate(parser, root, root != null); //1
}

//三参inflate()方法
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
        final Resources res = getContext().getResources();
        if (DEBUG) {
            Log.d(TAG, "INFLATING from resource: \"" + res.getResourceName(resource) + "\" ("
                    + Integer.toHexString(resource) + ")");
        }

        final XmlResourceParser parser = res.getLayout(resource);
        try {
            return inflate(parser, root, attachToRoot); //2
        } finally {
            parser.close();
        }
}

public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
  ...
}
```

从上面的源码我们能看出两参的inflate()方法最终也是调用了三参的Inflate()方法，两参和三参inflate()方法的第二个参数都是可空的(用@Nullable注解标识)，因此我们可以分多种情况去调用这两个方法。

两参inflate()方法有下面两种用法：

```java
inflate(parser,null); //第二个参数为空
inflate(parser,root); //第二个参数不为空
```

上面两种情况最终对应着三参inflate()方法的两种用法：

```
inflate()
```







参考文章：

[View绘制详解，从LayoutInflater谈起](https://blog.csdn.net/u012702547/article/details/52614444)