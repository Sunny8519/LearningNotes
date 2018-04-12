## 开发记录之CoordinatorLayout使用

#### 1. CoordinatorLayout的作用

> CoordinatorLayout is a super-powered {@link android.widget.FrameLayout FrameLayout}.
>
> CoordinatorLayout is intended for two primary use cases:
>
> As a top-level application decor or chrome layout
>
> As a container for a specific interaction with one or more child views

从类的描述大概可以知道CoordinatorLayout是一个“super-powered"FrameLayout。作用主要两个：

- 作为顶层布局
- 调度协调子布局

CoordinatorLayout通过设置子View的Behaviors来调度子View，系统已经提供了AppBarLayout.Behavior，AppBarLayout.ScrollingViewBehavior，FloatingActionButton.Behavior，SwipeDismissBehavior<V extends View>等

使用CoordinatorLayout需要在Gradle中加入Support Design Library：

> implementation 'com.android.support:design:xx.x.x'

#### 2. CoordinatorLayout与FloatingActionButton

使用CoordinatorLayout的布局文件：

```java
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <android.support.design.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.design.widget.FloatingActionButton
            android:id="@+id/btn_float"
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:layout_gravity="end|bottom"
            android:layout_margin="16dp" />
    </android.support.design.widget.CoordinatorLayout>

</layout>
```

![mmexport1523550388934.gif](https://upload-images.jianshu.io/upload_images/5231076-f38c3198a73fad94.gif?imageMogr2/auto-orient/strip)

再来看看使用FrameLayout作为根布局时的效果：

```java
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.design.widget.FloatingActionButton
            android:id="@+id/btn_float"
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:layout_gravity="end|bottom"
            android:layout_margin="16dp" />
    </FrameLayout>

</layout>
```

![20180413_003536.gif](https://upload-images.jianshu.io/upload_images/5231076-e15ec8238ac64b0b.gif?imageMogr2/auto-orient/strip)

从两个gif图中应该能看出差别来，即使用了CoordinatorLayout的FloatingActionButton在Snackbar显示的时候会有一个向上的动画，按钮不会被覆盖，并且在Snackbar消失的时候按钮回到原来的位置，使用FrameLayout则没有这样的效果，显得很生硬死板。



参考文章：

[android CoordinatorLayout使用](https://blog.csdn.net/xyz_lmn/article/details/48055919)