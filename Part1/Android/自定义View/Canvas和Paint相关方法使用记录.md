## 一. Canvas.drawXXX()和Paint基础

#### 1. Canvas.drawColor(@ColorInt int color)

在整个View的区域绘制上指定的颜色，该方法可以用来绘制背景，也可以用来绘制具有一定透明色的前景遮罩.

类似的方法：

drawRGB(int r,int g,int b)，该方法没有透明度；

drawARGB(int a , int r , int g , int b)，带有透明度。

**这类方法的作用主要是在绘制之前为View绘制背景色或者在View绘制完之后绘制半透明的蒙版。**

#### 2. Canvas.drawCircle(float centerX,float centerY,float radius,Paint paint)

(centerX,centerY)表示圆的圆心坐标，radius表示圆的半径，paint表示绘制圆的画笔。

**在Canvas绘制的过程中，我们可以把Canvas当作一个有坐标系的画布，默认情况下该画布的坐标原点是View的左上角，水平方向是x轴，右正左负，竖直方向是y轴，下正上负；通常对Canvas做translate，rotate等变换相当于变换了Canvas的坐标系**

![](https://user-gold-cdn.xitu.io/2017/7/10/e889c11486ac3b15733b209b1123fca1?imageslim)

#### 3. Canvas.drawRect(float left,float top,float right,float bottom,Paint paint)

(left,top)是矩形左上角的坐标，(right,bottom)是矩形右下角的坐标。

该方法还有两个重载方法：

Canvas.drawRect(RectF rect,Paint paint)

Canvas.drawRect(Rect rect,Paint paint)

我们可以通过构造RectF或者Rect来绘制矩形。

#### 4. Canvas.drawPoint(float x,float y,Paint paint)

(x,y)是点的坐标，点的大小可以通过Paint.setStrokeWidth(width)来设置；点的形状可以通过Paint.setStrokeCap(cap)，cap是Paint.Cap枚举，共有三种：ROUND表示圆形，SQUARE或者BUTT表示方形。不论点有多大，(x,y)表示的都是这些形状的中心点，如下图所示：

![drawPoint.png](https://upload-images.jianshu.io/upload_images/5231076-4e4c679c8b74bf75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**其实Paint.setStrokeCap(cap)设置的是画笔画出的线条端点的形状，这个方法不是专门用来设置点的形状的，端点有圆头，平头和方头。**

```java
paint.setStrokeWidth(20);
paint.setStrokeCap(Paint.Cap.SQUARE);//方头
canvas.drawLine(0, 0, 0, 100, paint);
```

效果：

![SQUARE.png](https://upload-images.jianshu.io/upload_images/5231076-a567956dae074e6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
paint.setStrokeWidth(20);
paint.setStrokeCap(Paint.Cap.ROUND);//圆头
canvas.drawLine(0, 0, 0, 100, paint);
```

效果：

![ROUND.png](https://upload-images.jianshu.io/upload_images/5231076-dc5c2ca59edcd996.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
paint.setStrokeWidth(20);
paint.setStrokeCap(Paint.Cap.BUTT);//平头
canvas.drawLine(0, 0, 0, 100, paint);
```

效果：

![BUTT.png](https://upload-images.jianshu.io/upload_images/5231076-8c812e8f8725e524.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



类似的方法还有：

drawPoints(float[] pts,int offset,int count,Paint paint)

drawPoints(float[] pts,Paint paint)

举例：

```java
float[] points = {0, 0, 50, 50, 50, 100, 100, 50, 100, 100, 150, 50, 150, 100};
// 绘制四个点：(50, 50) (50, 100) (100, 50) (100, 100)
canvas.drawPoints(points, 2 /* 跳过两个数，即前两个 0 */,
          8 /* 一共绘制 8 个数（4 个点）*/, paint);
```

**注意：这里如果坐标数量为奇数的话最后一个点不会被绘制**

#### 5. Canvas.drawOval(float left,float top,float right,float bottom,Paint paint)

该方法用于绘制椭圆，但是这个方法只有API21以上才有

![oval.png](https://upload-images.jianshu.io/upload_images/5231076-9786af5c537d5e5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

left表示a点的横坐标，top表示b点的纵坐标，right表示c点的横坐标，bottom表示d点的纵坐标

#### 6. Canvas.drawLine(float startX,float startY,float stopX,float stopY,Paint paint)

(startX,startY)表示线的起点坐标，(stopX,stopY)表示线的终点坐标，由于直线不是封闭图形，因此Paint.setStyle(style)对于画直线来说没有影响。

批量画线方法：

Canvas.drawLines(float[] pts,int offset,int count,Paint paint)

Canvas.drawLines(float[] pts,Paint paint)

举例：

```java
float[] points = {20, 20, 120, 20, 70, 20, 70, 120, 20, 120, 120, 120, 150, 20, 250, 20, 150, 20, 150, 120, 250, 20, 250, 120, 150, 120, 250, 120};
canvas.drawLines(points, paint);
```

在批量画点的方法中，点是两个两个为一对，一个表示x轴坐标，一个表示y轴坐标，在批量画线中，就是四个四个一对了，前两个坐标点表示线的起始点坐标，后两个坐标点表示线的终点坐标。

#### 7. Canvas.drawRoundRect(float left,float top,float right,float bottom,float rx,float ry,Paint paint)

(left,top)表示矩形左上角坐标，(right,bottom)表示矩形右下角坐标，rx和ry分别表示圆角的横向半径和纵向半径

```java
Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);
paint.setColor(Color.BLACK);
paint.setStyle(Paint.Style.STROKE);
paint.setStrokeWidth(2);
canvas.drawRoundRect(50, 50, 400, 400, 20, 20, paint);
paint.setColor(Color.RED);
canvas.drawRect(50, 50, 400, 400, paint);
```

效果图：

![roundRect.png](https://upload-images.jianshu.io/upload_images/5231076-87a39b09bf74faa4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重载方法：Canvas.drawRoundRect(RectF rect,float rx, float ry,Paint paint)，效果与上面一样

Canvas.drawRoundRect()方法只有在API21以上才能使用。

#### 8. Canvas.drawArc(float left,float top,float right,float bottom,float startAngle,float sweepAngle,boolean useCenter,Paint paint)

画弧线是基于椭圆，因此这里的left,top,right,bottom是先固定椭圆的位置，startAngle表示这个圆弧的起始角度，在Canvas的坐标系中，x轴正方向为0度，顺时针表示的是正角度，逆时针表示的是负角度，sweepAngle表示的是弧线扫过的角度，useCenter表示是否连接到椭圆的圆心，false表示不连接到圆心，这时画出来的就是一个弧形，true表示连接到圆心，这时画出来的就是一个扇形。

#### 9. Canvas.drawPath(Path path,Paint paint)

drawPath()方法是通过描述路径的方式来绘制图形，路径可以是各种图形，因此它一般用来绘制复杂图形。

Path可以描述直线，二次曲线，三次曲线，圆，椭圆，弧形，矩形，圆角矩形等，把这些图形结合起来就能表示复杂的图形。

Path.moveTo(x,y)方法对Path.addXXX()方法没有作用，但会对Path.xxxTo()方法起作用，表示线的绘制起点。

- addXXX()：添加子图形

比如addCircle(float x,float y,float radius,Direction dir)，x，y，radius表示圆心和半径，dir表示画圆的方向：CW表示顺时针，CCW表示逆时针

```JAVA
Path path = new Path();
path.lineTo(150, 150);//这一步画出来的线与下一步画出来的圆不相连
path.addCircle(250, 250, 50, Path.Direction.CCW);//逆时针画圆，画圆的起点是与x轴正向相交的点
path.lineTo(100, 200);//这一步与上一步的圆相连，这是因为lineTo()需要从当前位置开始绘制，那么在圆画完之后当前位置就是画圆的起始位置
canvas.drawPath(path, paint);
```

![lineTo.png](https://upload-images.jianshu.io/upload_images/5231076-55a6116e3ccb47fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**绘制出上面图形的Paint用的时Style.STROKE，如果使用FILL，只会画出一个黑色的圆，直线不会绘制，如果使用FILL_AND_STROKE，直线和黑色圆都会绘制出来**

特别的：addArc(float left,float top,float right,float bottom,float startAngle,float sweepAngle)和arcTo(float left,float top,float right,float bottom,float startAngle,float sweepAngle,boolean forceMoveTo)方法是只能画弧线，不能画扇形，forceMoveTo参数为true时表示“抬一下笔移动过去”，也就是中间不是连续的，为false时表示“直接拖着笔过去”，中间是连续的，比如下面这样：

```java
paint.setStyle(Style.STROKE);
path.lineTo(100, 100);
path.arcTo(100, 100, 300, 300, -90, 90, true); // 强制移动到弧形起点（无痕迹）,相当于addArc()
```

![](https://user-gold-cdn.xitu.io/2017/7/10/47bab7f17180dc78d5fcab0739654882?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```java
paint.setStyle(Style.STROKE);
path.lineTo(100, 100);
path.arcTo(100, 100, 300, 300, -90, 90, false); // 直接连线连到弧形起点（有痕迹）
```

![](https://user-gold-cdn.xitu.io/2017/7/10/0548755a36f968bafde6d420810797a8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- xxxTo()：画线(直线或者曲线)

lineTo(float x,float y)表示从当前位置向目标位置画一条直线，第一次画线时当前位置是(0,0)点，当然这个当前位置我们可以用moveTo(float x,float y)来修改，当我们从当前位置(比如(0,0))画到了(50,50)这个点，画完后当前位置就是(50,50)这个点，下次我们再画线时就是在(50,50)这个点的基础上继续画。

rLineTo(float x,float y)表示相对当前位置的相对坐标，相当于把当前位置的点当做坐标系的原点，然后在此基础上画线，因此这里的(x,y)坐标是相对于当前位置来说的。

- close()：**封闭当前子图形**

addXXX()方法的每次调用都会新增一个独立的子图形，而lineTo()，arcTo()等方法每一次在断线时(每次抬笔)就标志着一个子图形的结束以及一个新的子图形的开始。

不是所有的子图形都需要close()方法来封闭，如果需要填充图形时(即Paint.Style为FILL或者FILL_AND_STROKE)，path会自动封闭子图形。

```java
paint.setStyle(Style.FILL);
path.moveTo(100, 100);
path.lineTo(200, 100);
path.lineTo(150, 150);
// 这里只绘制了两条边，但由于 Style 是 FILL ，所以绘制时会自动封口
```

![](https://user-gold-cdn.xitu.io/2017/7/10/8903a42917ac71487fc7151a9694abb2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- setFillType(Path.FillType ft)：设置填充方式

FillType的取值有四种：EVEN_ODD、WINDING(默认值)、INVERSE_EVEN_ODD、INVERSE_WINDING

从前缀就能看出来，后面两种的效果与前面两种相反，所以只记录前面两种的用法

**EVEN_ODD(交叉填充)原理：**

即even_odd rule(奇偶原则)，对于平面中的任意一点，向任意方向射出一条射线，这条射线与图形相交的次数如果是奇数，则认为这个点是在图形内部，在绘制时是要被涂色的，如果相交的次数是偶数，则认为这个点是在图形外部，绘制时不会被涂色。

![](https://user-gold-cdn.xitu.io/2017/7/10/6eab1882cbbc67c5c9bef7ffff7cb11e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**WINDING(全填充)原理：**

即non-zero winding rule(非零环绕数原则)，前提是图形中的所有线条都是有绘制方向的，比如：

![](https://user-gold-cdn.xitu.io/2017/7/10/11ddb9358de84a4159253afef8c3953e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

平面内的一点向任意方向射出一条射线，以i = 0为初始值，射线遇到顺时针的交点时，把i加1，遇到逆时针的交点时把i减1，最后得到的结果不为0，则表示这个点在图形内部，在绘制时是需要被涂色的，如果最终i = 0,则表示这个点在图形外部，绘制时是不需要涂色的。

![](https://user-gold-cdn.xitu.io/2017/7/10/af36e2f187093a0cf0377a52797306f7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

案例：



![](https://user-gold-cdn.xitu.io/2017/7/10/811c7813fe34a9505aa35d5ed3caa1dd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```java
//代码案例
Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);
paint.setColor(Color.BLACK);
paint.setStyle(Paint.Style.FILL);//注意这里要是填充的
paint.setStrokeWidth(2);
paint.setTextAlign(Paint.Align.CENTER);
Path path = new Path();
path.setFillType(Path.FillType.EVEN_ODD);
path.addCircle(100, 100, 50, Path.Direction.CW);
path.addCircle(140, 100, 50, Path.Direction.CW);
canvas.drawPath(path, paint);
```

![EVEN_ODD.PNG](https://upload-images.jianshu.io/upload_images/5231076-2339f6ebaa27bff1.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 10. Canvas.drawBitmap(Bitmap bitmap,float left,float top,Paint paint)

(left,top)表示把Bitmap绘制到(left,top)位置上(左上角)

#### 11. Canvas.drawText(String text,float x,float y,Paint paint)

(x,y)在默认情况下表示左下角的坐标点，我们可以通过设置Paint.setTextAlign(Paint.Align)来改变(x,y)表示的坐标点，比如：`paint.setTextAlign(Paint.Align.CENTER);`就表示(x,y)是字体baseline的中点