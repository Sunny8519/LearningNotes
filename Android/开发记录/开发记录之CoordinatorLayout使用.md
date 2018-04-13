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

FloatingActionButton是最简单使用CoordinatorLayout的例子，它默认使用的时FloatingActionButton.Behavior

#### 3. CoordinatorLayout与AppBarLayout

先来了解一下AppBarLayout：

> AppBarLayout是一个垂直的LinearLayout，实现了Material Design中app bar的scrolling gestures特性。AppBarLayout的子View应该声明想要具有的“滚动行为”，这可以通过layout_scrollFlags属性或是setScrollFlags()方法来指定。
>
> AppBarLayout只有作为CoordinatorLayout的直接子View时才能正常工作，
>
> 为了让AppBarLayout能够知道何时滚动其子View，我们还应该在CoordinatorLayout布局中提供一个可滚动View，我们称之为scrolling view。scrolling view和AppBarLayout之间的关联，通过将scrolling view的Behavior设为AppBarLayout.ScrollingViewBehavior来建立。

根据上面这段论述，总结来说就是：**当位于CoordinatorLayout中的可滚动的View发生滚动时，AppBarLayout会根据子View声明的滚动行为来对相应View进行滚动。**

然后再来看看CoordinatorLayout和AppBarLayout结合使用的布局：

```java
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <android.support.design.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.design.widget.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <android.support.v7.widget.Toolbar
                android:id="@+id/tool_bar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="?attr/colorPrimary"
                app:layout_scrollFlags="scroll|enterAlways" //1
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

            <android.support.design.widget.TabLayout
                android:id="@+id/tab_layout"
                android:layout_width="match_parent"
                android:layout_height="wrap_content" />
        </android.support.design.widget.AppBarLayout>

        <android.support.v4.view.ViewPager
            android:id="@+id/view_pager"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior"/> //2

        <android.support.design.widget.FloatingActionButton
            android:id="@+id/fab"
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:layout_gravity="end|bottom"
            android:layout_margin="16dp" />
    </android.support.design.widget.CoordinatorLayout>

</layout>
```

主要关注注释1和2两处，注释1处是给需要根据具有滚动属性的View(这里是指ViewPager)而做相应滚动行为的View(这里是指ToolBar)设置滚动属性。

app:layout_scrollFlags属性值有以下几种：

- scroll：所有想要滚动出屏幕的View需要设置这个属性值，不然View只会固定在原来的位置不会动；
- enterAlways：当具有滚动属性的View在向下滚动的时候，该属性值会使View从不可见变为可见；
- enterAlwaysCollapsed：该属性值效果和enterAlways差不多，差别在于当View设置了minHeight属性，当具有滚动属性的View(比如ScrollView)向下滚动时，会先把View下拉minHeight高度，然后当具有滚动属性的View滚动到顶部时，再接着下拉View。
- exitUntilCollapsed：当具有滚动属性的View(比如ScrollView)向上滚动时，View先向上滚动到minHeight值后ScrollView再向上滚动。

app:layout_behavior属性设置为`@string/appbar_scrolling_view_behavior`，它是一串字符串：

```java
<string name="appbar_scrolling_view_behavior" translatable="false">android.support.design.widget.AppBarLayout$ScrollingViewBehavior</string>
```

也就是ScrollingViewBehavior这个类的路径，可想而知设置这个字符串最终会通过反射拿到ScrollingViewBehavior这个对象，这个对象就是ViewPager和ToolBar联动效果的关键。

#### 4. AppBarLayout嵌套CollapsingToolbarLayout



参考文章：

[android CoordinatorLayout使用](https://blog.csdn.net/xyz_lmn/article/details/48055919)

[自定义Behaivor](https://www.jianshu.com/p/f7989a2a3ec2)