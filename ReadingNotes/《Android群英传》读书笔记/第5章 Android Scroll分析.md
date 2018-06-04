## 第5章 Android Scroll分析

### 1. 滑动效果的本质

滑动一个View，本质上就是移动View的位置，原理与动画相似，那么如何改变一个View的位置呢？通过改变View的坐标就可以实现。

### 2. Android坐标系

Android坐标系是指以屏幕左上角为原点，向右为x轴正方向，向下为y轴正方向的坐标系，刚好和我们数学中常用的直角坐标系关于x轴对称。

获取关于Android坐标系的方法：

```java
//方法一
View.getLocationOnScreen();
View.getLocationOnScreen(int[] location);

//方法二
MotionEvent.getRawX();
MotionEvent.getRawY();
```

方法一是View类中自带的两个获取View相对于Android坐标系的位置坐标(View的左上角)，方法二用来获取手指触摸点相对于Android坐标系的位置坐标。

注：相对于屏幕左上角的坐标涵盖了通知栏的高度，也就是屏幕的最左上角。

### 3. 视图坐标系

视图坐标系参考的是父视图，它描述的是子视图在父视图中的位置，同样也是向右为x轴正方向，向下为y轴正方向。

获取视图坐标系的方法：

```java
MotionEvent.getX();
MotionEvent.getY();
```

在View中还有一个方法能够获取到View相对于它的父Window的坐标：

```java
View.getLocationInWindow(int[] location);
```

### 4. MotionEvent

MotionEvent中定义了多种触控事件的类型，常用的类型如下：

```java
//单点触摸按下动作
public static final int ACTION_DOWN             = 0;

//单点触摸抬起动作
public static final int ACTION_UP               = 1;

//单点触摸移动动作(不离开屏幕)
public static final int ACTION_MOVE             = 2;

//触摸动作取消
public static final int ACTION_CANCEL           = 3;

//触摸动作超出了UI元素的边界
public static final int ACTION_OUTSIDE          = 4;

//多点触摸按下动作
public static final int ACTION_POINTER_DOWN     = 5;

//多点触摸抬起动作
public static final int ACTION_POINTER_UP       = 6;
```

#### 4.1 View提供的获取坐标的方法

```java
//获取View相对于父容器的坐标
View.getLeft();
View.getTop();
View.getRight();
View.getBottom();

//获取View相对于屏幕左上角的坐标
View.getLocationOnScreen();
View.getLocationOnScreen(int[] location);

//获取View相对于父Window左上角坐标
View.getLocationInWindow(int[] location);
```

#### 4.2 MotionEvent提供的获取坐标的方法

```java
//获取当前触摸点相对于屏幕左上角坐标(绝对坐标)
MotionEvent.getRawX();
MotionEvent.getRawY();

//获取当前触摸点相对于控件左上角坐标(视图坐标)
MotionEvent.getX();
MotionEvent.getY();
```

### 5. 实现滑动的七种方法

#### 5.1 layout()方法

覆写View的onTouchEvent(MotionEvent)方法，在该方法中不断计算触摸点的偏移量，然后通过偏移量设置View的坐标，设置调用的方法为layout()；

```java
    float lastX = 0f;
    float lastY = 0f;

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        float x = event.getX();
        float y = event.getY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                lastX = x;
                lastY = y;
                break;
            case MotionEvent.ACTION_MOVE:
                int offsetX = (int) (x - lastX);
                int offsetY = (int) (y - lastY);
                layout(getLeft() + offsetX, getTop() + offsetY,
                        getRight() + offsetX, getBottom() + offsetY);
                break;
            case MotionEvent.ACTION_UP:
                //Do Nothing
                break;
        }
        return super.onTouchEvent(event);
    }
```

上面这种方式是通过获取视图坐标来计算滑动偏移量，这种方式仅仅需要在ACTION_DOWN中保存一次按下时触摸点的坐标值就可以了，不用在ACTION_MOVE或者switch语句的外部每次都记录上一次触摸点的坐标，原因：这里每次取出来的x,y坐标是相对于当前控件。

```java
    float lastX = 0f;
    float lastY = 0f;

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        float x = event.getRawX();
        float y = event.getRawY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                break;
            case MotionEvent.ACTION_MOVE:
                int offsetX = (int) (x - lastX);
                int offsetY = (int) (y - lastY);
                layout(getLeft() + offsetX, getTop() + offsetY,
                        getRight() + offsetX, getBottom() + offsetY);
                break;
            case MotionEvent.ACTION_UP:
                //Do Nothing
                break;
        }
        lastX = x;
        lastY = y;
        return super.onTouchEvent(event);
    }
```

上面这种方式获取的是Android坐标系中的触摸点坐标，这时就需要在每次调用onTouchEvent()方法的时候都要记录一下上次的坐标点，如果还和取视图坐标点的方式一样来计算偏移量，那么会导致计算出来的偏移量越来越大，View也就做不到随着手指流畅的滑动了。

#### 5.2 offsetLeftAndRight()与offsetTopAndBottom()

```java
//同时对left和right进行偏移
offsetLeftAndRight(offsetX);

//同时对top和bottom进行偏移
offsetTopAndBottom(offsetY);
```

上述计算偏移量的方式和5.1节中的一样。

#### 5.3 LayoutParams

通过LayoutParams来改变View的位置在本质上是改变View的Margin值，比如：

```java
ViewGroup.MarginLayoutParams lp = (ViewGroup.MarginLayoutParams)getLayoutParams();
lp.leftMargin = getLeft() + offsetX;
lp.topMargin = getTop() + offsetY;
setLayoutParams(lp);
```

只有MarginLayoutParams或者其子类才可以通过这种方式来改变View的位置，换句话说就是父容器的LayoutParams是MarginLayoutParams或者MarginLayoutParams的子类。

#### 5.4 scrollTo与scrollBy

scrollTo(x,y)表示移动到指定的点，scrollBy(dx,dy)表示移动的增量为dx,dy。

scrollTo(x,y)和scrollBy(dx,dy)移动的是View的内容(ViewGroup移动的是所有子View，View移动的是内容，比如TextView移动的就是文字)。

我们在使用scrollBy(dx,dy)时应该要注意的是dx,dy为负值时，View才会向正方向移动，可以想象成我们移动的是屏幕，把屏幕向坐标的负方向移动，那么屏幕里面的内容自然就向正方向移动了。

#### 5.5 Scroller



