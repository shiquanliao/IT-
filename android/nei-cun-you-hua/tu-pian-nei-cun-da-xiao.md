# 图片内存大小

## 图片加载到内存的过程已经变换

|  | MDPI | HDPI | XHDPI | XXHDPI | XXXHDPI |
| :---: | :---: | :---: | :---: | :---: | :---: |
| density | 160 | 240 | 320 | 480 | 640 |
| densityDpi | 1 | 1.5 | 2 | 3 | 4 |

{% hint style="warning" %}
**Any problem** in computer science can be solved by **another layer of indirection**.
{% endhint %}

### Bitmap

**getByteCount**

```java
public final int getByteCount(){
    if(mRecycled){
        LOg.w(TAG, "Called getByteCount() on a reclycle()'d bitmap! "
             + "This is  undefined behavior!");
        return 0;
    }

    // int result permits bitmaps up to 46340 x 46340
    return getRowBtyes() * getHeight();
}
```

**getAllocationByteCount**

```java
public final int getAllocationByteCount(){
    if(mRecycled){
        Log.w(TAG, "Called getAllocationByteCount() on a recycle()'d bitmap! "
             + "This is undefined behavior!");
        return 0;
    }
    return nativeGetAllocationByteCount(mNavitvePtr);
}
```

[两者的区别](https://juejin.cn/post/6844903604822736909)

## 图片加载的路径

**图片文件的 宽高: 112 \* 131, 目标机器的dpi为2.75**

* assets

  ```kotlin
  // 如果是 ----------------png
  BitmapFactory.decodeStream(assets.open("droid.png"))
  // png一般用argb_8888格式加载  
  // 宽 * 高 * 4 = 112 * 131 * 4 = 58688 B
  // 如果使用rgb_565的话大小就是:  112 * 131 * 2 = 29344 B

  // 如果是 ----------------jpg
  //jpg一般用rgb_565格式加载
  // 宽 * 高 * 2 = 112 * 131 * 2 = 29344 B
  BitmapFactory.decodeStream(assets.open("droid.jpg"), null,
                            Options().also{
                                it.inPreferredConfig = Bitmap.Config.RGB_565
                            })
  ```

* res/raw
* res/drawable-mdpi
* res/drawable-hdpi

  ```kotlin
  // 大小跟densityDpi有关, 跟屏幕密度也有关
  //新图的高度 = 原图高度 * (设备的 dpi / 目录对应的 dpi )
  //新图的宽度 = 原图宽度 * (设备的 dpi / 目录对应的 dpi )

  BitmapFactory.decodeResource(resoures, R.drawable.droid_hdpi)
  // 宽 = 112 * (2.75 / 1.5) = 205
  // 高 = 131 * (2.75 / 1.5) = 240
  // 加载格式为argb_8888
  // 占用内存为 205 * 240 * 4 = 196800 B
  ```

* res/drawable-xhdpi
* res/drawable-xxhdpi

  ```kotlin
  BitmapFactory.decodeResource(resoures, R.drawable.droid_xhdpi)
  // 宽 = 112 * (2.75 / 3) = 103
  // 高 = 131 * (2.75 / 3) = 120
  // 加载格式为argb_8888
  // 占用内存为 103 * 120 * 4 = 49440 B
  ```

* res/drawable-xxxhdpi

## Drawable中的图片加载流程

BitmapFactory

```java
public static Bitmap decodeResource(Resources res, int id, Options opts){
    validate(opts);
    Bitmap bm = null;
    InputStream is = null;

    final TypeValue value = new TypeValue();
    is = res.openRawResource(id, value);

    bm = decodeResourceStream(res, value, is, null, opts);

    return bm;
}

public static Bitmap decodeResourceStream(...){
    ...
    if(opts.inDensity == 0 && value != null){
        final int density = value.density;
        if(density == TypeValue.DENSITY_DEFALUT)
            opts.inDensity = DisplayMetrics.DENSITY_DEFALUT;
        else if(density != TypeValue.DENSITY_NONE)
            opts.inDensity = density;
    }
    if(opts.inTargetDensity == 0 && res != null)
        opts.inTargetDensity = res.getDisplayMetrics().densityDpi;
    return decodeStream(is, pad, opts);
}
```

{% tabs %}
{% tab title="BitmapFactory.cpp" %}
```java
static jobject doDecode(..., jobject options){
}
```
{% endtab %}
{% endtabs %}

## 图片内存占用大小和优化

