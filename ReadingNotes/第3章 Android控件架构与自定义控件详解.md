## 第3章 Android控件架构与自定义控件详解

#### 1. Android控件架构

在Activity中使用findViewById()是在视图树种以树的深度优先遍历来查找对应元素。

每棵视图树都有一个一个ViewParent对象(ViewRootImpl)，这棵视图树上的所有交互事件都由它来统一调度和分配，它是整个视图树的核心。

通过requestWindowFeature(Window.FEATURE_NO_TITLE)来设置全屏，该方法的调用需要在setContentView()之前。

#### 2. View的测量

View的测量是在onMeasure()方法种进行的。

onMeasure()方法中的两个参数都是32位的int型值，高2位表示测量的模式，低30位表示测量的大小。

测量模式分为EXACTLY，AT_MOST，UNSPECIFIED。

View默认情况下只对EXACTLY有效，因此我们想要自定义的View也能使用wrap_content就需要重写onMeasure()方法，在该方法中对AT_MOST模式下的尺寸做一些定制化操作，一般我们很少使用到UNSPECIFIED模式。

我们在onMeasure()方法中处理AT_MOST模式的View大小时，最终取的是我们指定的View的大小和SpecSize的较小值(通常会做一个Math.min操作)，因为AT_MOST模式是不允许子View的大小超过父容器指定的大小。

如果自定义View不重写onMeasure()方法，为该View指定wrap_content时会填充整个父布局。

#### 3. View的绘制

View中Canvas属于系统2D绘图。

onDraw(Canvas canvas)中默认会传入一个Canvas对象，可以调用这个Canvas对象的API做一些绘图操作。我们还可以在其他地方使用Canvas，不过要先实例化一个Canvas对象，然后为这个Canvas对象装载一个Bitmap对象，比如：

```java
Canvas canvas = new Canvas(bitmap);
```

这样在这个canvas对象上做的所有绘图操作都会记录在bitmap中，如果在View的onDraw()方法中用onDraw()提供的Canvas绘制这个bitmap对象，就能把刚才所做的所有绘图操作绘制出来，比如：

```java
@Override
public void onDraw(Canvas canvas){
    ...
    canvas.drawBitmap(bitmap,0,0,null);//绘制bitmap
    ...
}
```

#### 4. ViewGroup的测量

**ViewGroup是子View的容器，当ViewGroup的大小为wrap_content的时候，ViewGroup需要对子View进行遍历去获得所有子View的大小，然后决定自己的大小，在其他模式下则会通过指定值来设置自身的大小。**

在子View和自身大小测量完毕之后，一般会通过onLayout()方法来控制子View的显示位置。

ViewGroup一般不会覆写onDraw或者dispathDraw()方法，但是某些情况下，我们需要覆写来达到某些显示效果。

ViewGroup的onDraw()方法需要在指定背景颜色时才会被调用(**这点待验证**)，但是ViewGroup会通过dispatchDraw()方法来绘制子View，因此如果我们想在ViewGroup中绘制一些内容的话可以覆写dispatchDraw()方法来进行绘制，有一点需要注意的是一定要在覆写的dispatchDraw()方法中调用父类的dispatchDraw()方法。

#### 5. 自定义View

View中比较重要的回调方法：

- onFinishInflate()：控件从xml文件中解析完后回调。
- onSizeChanged()：控件大小改变时回调。

实现自定义View的三种方式：

- 对现有控件进行拓展。
- 通过组合来实现新的控件。
- 重写View来实现全新的控件。

对现有控件进行拓展时，可以通过覆写onDraw()方法绘制自己想要的内容，比如对TextView进行扩展：

```java
@Override
public void onDraw(Canvas canvas){
    //绘制文字之前实现自己的绘制逻辑，这里绘制的内容将会在TextView本身绘制内容的下方
    super.onDraw(canvas);
    //绘制文字之后实现自己的绘制逻辑，这里绘制的内容会位于TextView本身绘制内容的上方
}
```

在利用LinearGradient和Matrix实现文字闪烁效果过程中：

- View中onWindowFocusChanged()方法是在每次Window的焦点发生变化时被调用，比如锁屏和解除锁屏，退出和进入当前页面。

以下为xml文件中的View从解析到被绘制过程中比较重要的回调方法的回调顺序：

```
onFinishInflate
onAttachedToWindow
onMeasure
onMeasure
onMeasure
onMeasure
onSizeChanged
onLayout
onWindowFocusChanged
onDraw
```

从这个回调顺序可知，onSizeChanged()方法是在onMeasure()方法之后被回调的，也就是View的大小发生变化时回调，View一旦大小发生改变必然会触发View的重新测量，接着onSizeChanged()方法也会被回调。

自定义View实现文字闪烁的关键代码如下：

```java
@Override
protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        Log.i(TAG, "onSizeChanged");

        if (this.mViewWidth == 0) {
            this.mViewWidth = getMeasuredWidth();

            if (this.mViewWidth > 0) {
                this.mLinearGradient = new LinearGradient(
                        -this.mViewWidth / 2, 0,
                        this.mViewWidth / 2, 0,
                        new int[]{Color.BLACK, 0xffffffff, Color.BLACK},
                        null, Shader.TileMode.CLAMP
                );
                getPaint().setShader(this.mLinearGradient);
                this.mGradientMatrix = new Matrix();
            }
        }
}

@Override
protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        Log.i(TAG, "onDraw");

        if (this.mGradientMatrix != null) {
            this.mTranslate += this.mViewWidth / 5f;
            if (this.mTranslate > this.mViewWidth) {
                this.mTranslate = 0;
            }

            this.mGradientMatrix.setTranslate(this.mTranslate, 0);
            this.mLinearGradient.setLocalMatrix(this.mGradientMatrix);

            postInvalidateDelayed(100);
        }
}
```

postInvalidateDelayed()每次只会触发onDraw()方法。

```java
//系统命名空间
xmlns:android="http://schemas.android.com/apk/res/android"

//自定义命名空间,app也可以改为其他名称，一般用于自定View的属性
xmlns:app="http://schemas.android.com/apk/res-auto"
```

xmlns全称为xml namespace

通过组合实现新控件一般需要继承布局控件，比如LinearLayout，FrameLayout，RelativeLayout等，通过Java代码或者加载xml文件的方式实现组合控件。

