### 开发记录之Dialog风格Activity

要想实现Dialog风格的Activity我们第一想法就是使用Dialog风格的主题，比如下面这样：

```java
//如果Activity使用的AppCompatActivity就必须使用Theme.AppCompat这一类的主题
<style name="DialogTheme" parent="Theme.AppCompat.Dialog">
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
</style>

//把上面的风格应用于指定Activity
<activity
      android:name=".custom_view.DialogStyleActivity"
      android:theme="@style/DialogTheme" />
```

#### 1. 指定根布局固定宽高

指定根布局固定宽高，并且宽高不超过全屏大小时的显示效果，xml布局如下：

```java
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <android.support.constraint.ConstraintLayout
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:background="@android:color/white">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@color/colorPrimary"
            android:text="DialogStyleActivity"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
    </android.support.constraint.ConstraintLayout>

</layout>
```

预览图如下：

![固定宽高.png](https://upload-images.jianshu.io/upload_images/5231076-85a98c8dde852e57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最终显示效果：

<img src="https://upload-images.jianshu.io/upload_images/5231076-e993943941d2680a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="50%" height="50%">

#### 2. 根布局的宽高指定为wrap_content

xml布局如下：

```java
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <android.support.constraint.ConstraintLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="@android:color/white">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@color/colorPrimary"
            android:text="DialogStyleActivity"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
    </android.support.constraint.ConstraintLayout>

</layout>
```

预览图如下：

![wrap_content.png](https://upload-images.jianshu.io/upload_images/5231076-d8362de2d6ac0b07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最终显示效果：

<img src="https://upload-images.jianshu.io/upload_images/5231076-6871de4697eecede.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="50%" height="50%">

#### 3. 根布局的宽高指定为match_parent

xml文件如下：

```java
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <android.support.constraint.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@android:color/white">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@color/colorPrimary"
            android:text="DialogStyleActivity"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
    </android.support.constraint.ConstraintLayout>

</layout>
```

预览图：

![match_parent.png](https://upload-images.jianshu.io/upload_images/5231076-94090e1907cba683.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最终显示效果图：

<img src="https://upload-images.jianshu.io/upload_images/5231076-6871de4697eecede.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="50%" height="50%">

#### 4. 根布局指定固定宽高，但是超出屏幕大小

xml文件如下：

```java
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <android.support.constraint.ConstraintLayout
        android:layout_width="700dp"
        android:layout_height="900dp"
        android:background="@android:color/white">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@color/colorPrimary"
            android:text="DialogStyleActivity"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
    </android.support.constraint.ConstraintLayout>

</layout>
```

预览图：

![固定宽高.png](https://upload-images.jianshu.io/upload_images/5231076-8d2aff7bf4d9c58f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最终显示效果：

<img src="https://upload-images.jianshu.io/upload_images/5231076-9bbe5db0be05941a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="50%" height="50%">

上下左右会留出一点空隙出来。



