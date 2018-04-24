## View绘制原理之LayoutInflater

#### 一. LayoutInflater的使用

在说明LayoutInflater的原理之前，我们以一个应用场景为例来阐述一下开发中可能遇到的问题：

现在需要给一个xml布局的根标签设置固定宽高，然后把该布局的View加入到另一个容器中显示。哎，挺简单的嘛，我们可以这样实现：

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

咦，怎么回事，为什么设置的200x200的固定尺寸没有生效呢，怎么是全屏的蓝色？？这和我们期望的显示效果不一致啊，这时我们可能想到一个解决办法：在子View的外面再包一层ViewGroup，让它是match_parent的，可能就好了，我们试试。

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

但是我们有没有想过一个问题，这样在子View的的布局外面再套一层ViewGroup真的是必要的吗？说白了，我们还没有找到问题的根源在哪里，恰巧这样的方式解决了我们遇到的问题。

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

//三参inflate()重载方法
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
  ...
}
```

从上面的源码我们能看出两参的inflate()方法最终也是调用了三参的`inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot)`方法，两参和三参inflate()方法的第二个参数都是可空的(用@Nullable注解标识)，因此我们可以分多种情况去调用这两个方法。

两参inflate()方法有下面两种用法：

```java
inflate(parser,null); //第二个参数为空
inflate(parser,root); //第二个参数不为空
```

上面两种情况最终对应着三参inflate()方法的两种用法：

```java
inflate(parser,null,false);
inflate(parser,root,true);
```

如果调用三参的inflate()方法，还会多出以下两种情况：

```java
inflate(parser,null,true);
inflate(parser,root,false);
```

所以，综上所述，调用inflate()方法会有四种姿势，下面我们来分别验证这四种用法对应的View在ViewGroup中会如何显示...

#### 三. inflate()不同参数值情况下的View显示效果

##### inflate(parser,null,false)

```java
@Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ActivityLayoutInflaterBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_layout_inflater); //设置Activity的布局

        View childView = LayoutInflater.from(this).inflate(R.layout.view_child, null, false); //解析子View的布局
        binding.content.addView(childView); //把子View加到父容器中
    }
```

效果：

<img src="https://upload-images.jianshu.io/upload_images/5231076-b65d0320f3fb9d2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="50%" height="50%">

##### inflate(parser,root,false)

```java
 @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ActivityLayoutInflaterBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_layout_inflater); //设置Activity的布局

        View childView = LayoutInflater.from(this).inflate(R.layout.view_child, binding.content, false); //解析子View的布局
        binding.content.addView(childView); //把子View加到父容器中
    }
```

效果：

<img src="https://upload-images.jianshu.io/upload_images/5231076-f4e5f2234b3056ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="50%" height="50%">

##### inflate(parser,null,true)

```java
@Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ActivityLayoutInflaterBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_layout_inflater); //设置Activity的布局

        View childView = LayoutInflater.from(this).inflate(R.layout.view_child, null, true); //解析子View的布局
        binding.content.addView(childView); //把子View加到父容器中
    }
```

效果：

<img src="https://upload-images.jianshu.io/upload_images/5231076-b65d0320f3fb9d2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="50%" height="50%">

##### inflate(parser,root,true)

```java
@Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ActivityLayoutInflaterBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_layout_inflater); //设置Activity的布局

        View childView = LayoutInflater.from(this).inflate(R.layout.view_child, binding.content, true); //解析子View的布局
        binding.content.addView(childView); //把子View加到父容器中
    }

//抛出异常
Caused by: java.lang.IllegalStateException: The specified child already has a parent. You must call removeView() on the child's parent first.
```

这里抛出了异常，应用程序退出。从异常信息很容易得出我们向bingding.content父View添加子View重复添加了，那么也就是`View childView = LayoutInflater.from(this).inflate(R.layout.view_child, binding.content, true);`这行代码已经把子View加入到父View中了，下一行代码`binding.content.addView(childView);`是不需要的。我们把它删了再看看运行效果。

```java
@Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ActivityLayoutInflaterBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_layout_inflater); //设置Activity的布局

        View childView = LayoutInflater.from(this).inflate(R.layout.view_child, binding.content, true); //解析子View的布局
//        binding.content.addView(childView); //把子View加到父容器中
    }
```

效果：

<img src="https://upload-images.jianshu.io/upload_images/5231076-f4e5f2234b3056ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="50%" height="50%">

这回就能正确显示了，而且子View的宽高也没有失效。

从上面不同参数对应不同的显示效果来看，我们有必要看看inflate()内部的实现逻辑，搞清导致这种状况的内在原因，这有助于我们对inflate()方法的理解。

#### 四. inflate()方法源码分析

不论是`inflate(XmlPullParser parser, @Nullable ViewGroup root)`还是`inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot)`方法，最终调用的都是`inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot)`，这个方法实现了inflate()的主要逻辑，我们来看一看：

```java
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");

            final Context inflaterContext = mContext;
            final AttributeSet attrs = Xml.asAttributeSet(parser);
            Context lastContext = (Context) mConstructorArgs[0];
            mConstructorArgs[0] = inflaterContext;
            View result = root;

            try {
                // Look for the root node.
                int type;
                while ((type = parser.next()) != XmlPullParser.START_TAG &&
                        type != XmlPullParser.END_DOCUMENT) {
                    // Empty
                }

                if (type != XmlPullParser.START_TAG) {
                    throw new InflateException(parser.getPositionDescription()
                            + ": No start tag found!");
                }

                final String name = parser.getName();
                
                if (DEBUG) {
                    System.out.println("**************************");
                    System.out.println("Creating root view: "
                            + name);
                    System.out.println("**************************");
                }

                if (TAG_MERGE.equals(name)) {
                    if (root == null || !attachToRoot) {
                        throw new InflateException("<merge /> can be used only with a valid "
                                + "ViewGroup root and attachToRoot=true");
                    }

                    rInflate(parser, root, inflaterContext, attrs, false);
                } else {
                    // Temp is the root view that was found in the xml
                    final View temp = createViewFromTag(root, name, inflaterContext, attrs);
                    // 获取xml根布局View
                    final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                    ViewGroup.LayoutParams params = null;

                    if (root != null) {
                        if (DEBUG) {
                            System.out.println("Creating params from root: " +
                                    root);
                        }
                        // Create layout params that match root, if supplied
                        params = root.generateLayoutParams(attrs); //获取能够匹配父容器的LayoutParams
                        //attachToRoot为false时会给生成的xml根View设置LayoutParams，这样xml根View设置的宽高属性就不会失效
                        if (!attachToRoot) {
                            // Set the layout params for temp if we are not
                            // attaching. (If we are, we use addView, below)
                            temp.setLayoutParams(params);
                        }
                    }

                    if (DEBUG) {
                        System.out.println("-----> start inflating children");
                    }

                    // Inflate all children under temp against its context.
                    rInflateChildren(parser, temp, attrs, true); //递归加载子布局

                    if (DEBUG) {
                        System.out.println("-----> done inflating children");
                    }

                    // We are supposed to attach all the views we found (int temp)
                    // to root. Do that now.
                    // root不等于null,attachToRoot为true时把xml生成的视图树add进root中
                    if (root != null && attachToRoot) { 
                        root.addView(temp, params);
                    }

                    // Decide whether to return the root that was passed in or the
                    // top view found in xml.
                    // root为null或者attachToRoot为false时把xml生成的视图树赋值给result
                    if (root == null || !attachToRoot) {
                        result = temp;
                    }
                }

            } catch (XmlPullParserException e) {
                final InflateException ie = new InflateException(e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } catch (Exception e) {
                final InflateException ie = new InflateException(parser.getPositionDescription()
                        + ": " + e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } finally {
                // Don't retain static reference on context.
                mConstructorArgs[0] = lastContext;
                mConstructorArgs[1] = null;

                Trace.traceEnd(Trace.TRACE_TAG_VIEW);
            }

            return result;
        }
    }
```

参考文章：

[View绘制详解，从LayoutInflater谈起](https://blog.csdn.net/u012702547/article/details/52614444)

[三个案例带你看懂LayoutInflater中inflate方法两个参数和三个参数的区别](https://blog.csdn.net/u012702547/article/details/52628453)

[Android应用setContentView与LayoutInflater加载解析机制源码分析](https://blog.csdn.net/yanbober/article/details/45970721)

