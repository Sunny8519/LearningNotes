## 开发记录之封装Zxing二维码生成工具类

```java
/**
 * author : Sunny
 * time : 2018/5/30
 * desc : 动态生成二维码工具类
 */
public class QRCodeUtil {

    @Nullable
    public static Bitmap createQRCodeBitmap(@Nullable String content, int size) {
        return createQRCodeBitmap(content, size, "UTF-8", "H",
                null, Color.BLACK, Color.WHITE, null, null, 0f);
    }

    @Nullable
    public static Bitmap createQRCodeBitmap(@Nullable String content, int size, @ColorInt int color_black, @ColorInt int color_white) {
        return createQRCodeBitmap(content, size, "UTF-8", "H",
                null, color_black, color_white, null, null, 0f);
    }

    @Nullable
    public static Bitmap createQRCodeBitmap(@Nullable String content, int size, @Nullable Bitmap logoBitmap, float logoPercent) {
        return createQRCodeBitmap(content, size, "UTF-8", "H", null, Color.BLACK, Color.WHITE, null, logoBitmap, logoPercent);
    }

    @Nullable
    public static Bitmap createQRCodeBitmap(@Nullable String content, int size, @Nullable Bitmap targetBitmap) {
        return createQRCodeBitmap(content, size, "UTF-8", "H", null, Color.BLACK, Color.WHITE, targetBitmap, null, 0f);
    }

    @Nullable
    public static Bitmap createQRCodeBitmap(@Nullable String content, int size,
                                            @Nullable String character_set, @Nullable String error_correction, @Nullable String margin,
                                            @ColorInt int color_black, @ColorInt int color_white, @Nullable Bitmap targetBitmap,
                                            @Nullable Bitmap logoBitmap, float logoPercent) {
        if (TextUtils.isEmpty(content)) return null;

        if (size <= 0) return null;

        //设置生成二维码所需的参数
        final Map<EncodeHintType, String> hints = new HashMap<>();
        if (!TextUtils.isEmpty(character_set)) {
            //添加字符编码格式，默认是"ISO-8859-1"
            hints.put(EncodeHintType.CHARACTER_SET, character_set);
        }
        if (!TextUtils.isEmpty(error_correction)) {
            //设置容错级别，默认是L
            hints.put(EncodeHintType.ERROR_CORRECTION, error_correction);
        }
        if (!TextUtils.isEmpty(margin)) {
            //设置空白边距，默认是4
            hints.put(EncodeHintType.MARGIN, margin);
        }

        Bitmap bitmap = null;
        try {
            //利用Zxing实现字符串的编码，把字符串转为位矩阵
            BitMatrix matrix = new QRCodeWriter().encode(content, BarcodeFormat.QR_CODE, size, size, hints);

            Bitmap innerTargetBitmap = targetBitmap;
            if (innerTargetBitmap != null) {
                innerTargetBitmap = Bitmap.createScaledBitmap(innerTargetBitmap, size, size, false);
            }

            /*生成二维码中黑色色块像素和白色色块像素的颜色(也就是说可能不是黑白的，看设置)*/
            int[] pixels = new int[size * size];
            for (int y = 0; y < size; y++) {
                for (int x = 0; x < size; x++) {
                    //(x,y)点在矩阵中的值，true表示黑色，false表示白色
                    int count = y * size + x;
                    if (matrix.get(x, y)) { //黑色色块像素
                        if (innerTargetBitmap != null) { //如果targetBitmap不为空，就使用targetBitmap对应位置像素的颜色
                            pixels[count] = innerTargetBitmap.getPixel(x, y);
                        } else {
                            pixels[count] = color_black;
                        }
                    } else { //白色色块像素
                        pixels[count] = color_white;
                    }
                }
            }

            bitmap = Bitmap.createBitmap(size, size, Bitmap.Config.ARGB_8888);
            bitmap.setPixels(pixels, 0, size, 0, 0, size, size);

            if (logoBitmap != null) {
                bitmap = addLogo(bitmap, logoBitmap, logoPercent);
            }
        } catch (WriterException e) {
            e.printStackTrace();
        }
        return bitmap;
    }

    @Nullable
    private static Bitmap addLogo(@Nullable Bitmap srcBitmap, @Nullable Bitmap logoBitmap, float logoPercent) {
        if (srcBitmap == null) return null;

        if (logoBitmap == null) return srcBitmap;

        int srcWidth = srcBitmap.getWidth();
        int srcHeight = srcBitmap.getHeight();
        int logoWidth = logoBitmap.getWidth();
        int logoHeight = logoBitmap.getHeight();

        //计算logo图的缩放比
        float scaleInWidth = srcWidth * logoPercent / logoWidth;
        float scaleInHeight = srcHeight * logoPercent / logoHeight;

        Bitmap bitmap = Bitmap.createBitmap(srcWidth, srcHeight, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        canvas.drawBitmap(srcBitmap, 0, 0, null);

        canvas.scale(scaleInWidth, scaleInHeight, srcWidth / 2, srcHeight / 2);
        canvas.drawBitmap(logoBitmap, srcWidth / 2 - srcWidth * logoPercent / 2, srcHeight / 2 - srcHeight * logoPercent / 2, null);
        return bitmap;
    }
}
```

网络上很多使用Zxing库的时候都是使用的jar包形式，但是Android中我们可能更倾向于使用gradle依赖：
`implementation 'com.google.zxing:core:3.3.3'`

