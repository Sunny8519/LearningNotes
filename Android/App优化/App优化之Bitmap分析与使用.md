## App优化之Bitmap分析与使用

#### 1. Bitmap的创建

创建Bitmap的时候，系统不提供`new Bitmap()`的方式去创建，而是通过`BitmapFactory`中的静态方法去创建，比如`BitmapFactory.decodeStream(InputStream);//通过InputStream去解析生成Bitmap`，跟进BitmapFactory中的创建Bitmap的源码，最终都会追溯到这几个native函数：

```java
//记载Bitmap的方式
decodeFile(String)
decodeFile(String,Options)
decodeResource(Resources,int)
decodeResource(Resource,int,Options)
decodeStream(InputStream)
decodeStream(InputStream,Rect,Options)
decodeByteArray(byte[],int,int)
decodeByteArray(byte[],int,int,Options)

//以上Java方法最终调用的native方法
private static native Bitmap nativeDecodeStream(InputStream is, byte[] storage, Rect padding, Options opts);

private static native Bitmap nativeDecodeFileDescriptor(FileDescriptor fd, Rect padding, Options opts);

private static native Bitmap nativeDecodeAsset(long nativeAsset, Rect padding, Options opts);

private static native Bitmap nativeDecodeByteArray(byte[] data, int offset, int length, Options opts);
```

由此，Bitmap的Java对象是从native函数得来，也就是说Bitmap占用的内存空间是在C/C++中的分配出来的，因此，Android中Bitmap的内存管理会涉及两部分：1.native，2.dalvik(ART，也就是Java内存模型中的堆内存)。

查看Bitmap实例化之后的内存使用情况：`adb shell dumpsys meminfo`

#### 2. Bitmap的使用

合理使用Bitmap需要关注以下几点：

- #### **Size**

一张图片转为Bitmap后占用内存是多大呢？这是可以计算的。假如以Config.ARGB_8888参数去创建Bitmap(这是Android4.4及以上版本默认创建Bitmap的Config参数，`public Bitmap.Config inPreferredConfig = Bitmap.Config.ARGB_8888;`)。[这是Google推荐的配置色彩参数](https://developer.android.com/reference/android/graphics/Bitmap.Config.html)。ARGB_8888表示的是一个像素占用4个字节(A表示透明度，占用1字节，R表示红色，占用1字节，G表示绿色，占用1字节，B表示蓝色，占用1字节)。

比如现在有一张照片的尺寸为1289x720，那么这张未经处理的照片转为Bitmap后占用内存大小为：1280x720x4=3686400(byte)=3.6M，一张图片就占用这么大，要是这样的图片有几十张甚至上百张，App可用内存极有可能撑不住，出现OOM。

因此这就需要我们在使用的时候去加载合适的Bitmap大小，通俗说就是用多少加载多少。如何做到？？通过BitmapFactory.Options中的inSampleSize设置Bitmap的压缩比。

官方说法：

> If set to a value > 1, requests the decoder to subsample the original image, returning a smaller image to save memory....For example, inSampleSize == 4 returns
> an image that is 1/4 the width/height of the original, and 1/16 the
> number of pixels. Any value <= 1 is treated the same as 1.

①inSampleSize参数

采样率，Bitmap会根据这个值去算实际需要的Bitmap的大小，该值如果小于1会按1来算，1表示取原图的大小，大于1，表示图片的宽高取1/n(n表示inSampleSize值)，官方文档给出的inSampleSize取值为2的指数，即1,2,4,8等，如果不是2的指数系统会自动向下取整，取一个最接近2的指数的整数来代替，比如现在inSampleSize的值为3，那么系统实际上会按照inSampleSize = 2来计算图片的缩放比例；还需要注意一点的是当宽高的缩放比例不一样的时候，通常是取小的，这样不会导致图片显示不完整或者因为拉伸造成图片的失真。

举例：现在有一张200 x 400的原始图片，需要在一个100 x 100的ImageView上显示，那么宽的缩放比为2，高的缩放比是4，这时候应该取inSampleSize值为2，取小的值，这样实际获得的图片大小为100 x 200，在100 x 100上就能显示出来了，即使高多了一点也没关系，如果按照4的比例来取得话，那么实际获得的图片大小就是50 x 100，宽要比ImageView的宽小一半，这样显示的时候就不能铺满，如果硬要铺满会造成图片宽度被拉伸，从而失真。

但是我们在使用inSampleSize参数的时候，还应该配合着inJustDecodeBounds参数。

②inJustDecodeBounds参数

一般情况下，我们获取图片的原始大小需要先加载图片，这样原始图片还是被加载到了内存中，这违背了高效加载Bitmap的初衷；设置inJustDecodeBounds = true就能在不加载图片的情况下获取到原始图片的大小，然后根据图片原始大小和inSampleSize来计算出实际需要的图片大小，然后再设置inJustDecodeBounds = false就能按照实际需要的图片大小去加载图片到内存。

**总结：**

##### 高效加载Bitmap的流程：

①new一个BitmapFactory.Options对象，设置Options中字段inJustDecodeBounds = true，并加载图片；

②从BitmapFactory.Options中获取图片的原始宽高，宽高分别对应于outWidth和outHeight字段；

③根据采样率的规则和目标View的大小计算出所需inSampleSize值；

④把inJustDecodeBounds设置为false，再根据inSampleSize加载图片。

##### Bitmap高效加载的代码实现

```java
public static Bitmap decodeSampleBitmapByResource(Resources resources, @IdRes int resId, int           reqWidth, int reqHeight) {
    //new一个Options对象并设置inJustDecodeBounds = true
    BitmapFactory.Options op = new BitmapFactory.Options();
    op.inJustDecodeBounds = true;
    //加载图片
    BitmapFactory.decodeResource(resources, resId, op);
    //依据图片的原始大小、目标View的大小以及获取inSampleSize值得规则计算inSampleSize的值
    op.inSampleSize = calculateInSample(op, reqWidth, reqHeight);
    //设置inJustDecodeBounds = false
    op.inJustDecodeBounds = false;
    //再根据inSampleSize加载图片
    return BitmapFactory.decodeResource(resources, resId, op);
}

public static int calculateInSample(BitmapFactory.Options options, int reqWidth, int reqHeight) {
    final int orWidth = options.outWidth;
    final int orHeight = options.outHeight;
    int inSample = 1;
    if (orWidth > reqWidth || orHeight > reqHeight) {
        int halfWidth = orWidth / 2;
        int halfHeight = orHeight / 2;
        while (halfWidth / inSample >= reqWidth && halfHeight / inSample >= reqHeight) {
            inSample *= 2;
        }
    }
    return inSample;
}
```

- **Reuse**

加载少量的图片使用上面那种方式应该就够了，但是如果要加载大量且有可能重复的图片，使用上面的方式就有点杯水车薪了，这时我们应该提供一种图片复用的功能。在BitmapFactory.Options中有一个inBitmap字段可实现已加载的Bitmap的复用(**Bitmap的复用还可以使用三级缓存来做，待补充**)，但是该字段的在Android 4.4之前使用有一定限制：即只支持同等大小的位图。

##### inBitmap使用方式

```java
BitmapFactory.Options options = new BitmapFactory.Options();
//inBitmap只有当inMutable为true的时候是可用的。
options.inMutable = true;
Bitmap reusedBitmap = BitmapFactory.decodeResource(getResources(),R.drawable.reused_btimap,options);
options.inBitmap = reusedBitmap;
```

在用到这张图片的地方使用同一个Options去加载图片，就会复用内存中已经存在过的Bitmap。

比如：

```java
         LinearLayout linearLayout = new LinearLayout(this);
         linearLayout.setLayoutParams(new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT));
         linearLayout.setOrientation(LinearLayout.VERTICAL);
 
         ImageView iv = new ImageView(this);
         iv.setLayoutParams(new ViewGroup.LayoutParams(500,300));
 
         options = new BitmapFactory.Options();
         options.inJustDecodeBounds = true;
         //inBitmap只有当inMutable为true的时候是可用的。
         options.inMutable = true;
         BitmapFactory.decodeResource(getResources(),R.drawable.big_pic,options);
         
         //压缩Bitmap到我们希望的尺寸
         //确保不会OOM
         options.inSampleSize = calculateInSample(options, 500, 300);
         options.inJustDecodeBounds = false;
 
         Bitmap bitmap = BitmapFactory.decodeResource(getResources(),R.drawable.big_pic,options);
         options.inBitmap = bitmap;
 
         iv.setImageBitmap(bitmap);
 
         linearLayout.addView(iv);
 
         ImageView iv1 = new ImageView(this);
         iv1.setLayoutParams(new ViewGroup.LayoutParams(500,300));
         iv1.setImageBitmap( BitmapFactory.decodeResource(getResources(),R.drawable.big_pic,options)); //使用同一个options
         linearLayout.addView(iv1);
 
         ImageView iv2 = new ImageView(this);
         iv2.setLayoutParams(new ViewGroup.LayoutParams(500,300));
         iv2.setImageBitmap( BitmapFactory.decodeResource(getResources(),R.drawable.big_pic,options)); //使用同一个options
         linearLayout.addView(iv2);
```

- **Recycle**

在使用完Bitmap后一定要及时回收Bitmap，否则，native和dalvik内存会被一直占用着，最终导致OOM。

回收Bitmap：

```java
// 先判断是否已经回收
if(bitmap != null && !bitmap.isRecycled()){
    // 回收并且置为null
    bitmap.recycle();
    bitmap = null;
}
System.gc();
```

#### 3. Bitmap的构建流程

```java
// called from JNI
Bitmap(long nativeBitmap, byte[] buffer, int width, int height, int density,
         boolean isMutable, boolean requestPremultiplied,
         byte[] ninePatchChunk, NinePatch.InsetStruct ninePatchInsets)
```



参考文章：

[Android中Bitmap内存优化](https://www.jianshu.com/p/3f6f6e4f1c88)

