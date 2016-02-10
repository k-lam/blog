

##Bitmap 
Bitmap的使用关键在于，创建Bitmap时控制分配的大小。然后不要保存bitmap的引用，管理好生命周期（在3.0后Bitmap没有被引用到，会马上被recycle的）

###Allocated
Bitmap的需要很大的内存分配，而这些内存即使主动bitmap.recycle后，还是会alocated的，而没有立刻还给系统。除非系统催app还钱。因为这样当我再需要资源的时候，我就可以用已经分配给我的，而不用再次向系统申请。可以看到，一个空白的启动页，一开启app，也会占用到20+M的内存空间。所以app的heap其实包括：
* Max （最大可分配）
* Allocated
而Allocated中大体上是分为已用，和未用，这需要看dvm和以后的art是怎样的。

如果Allocated很大，会影响app运行速度吗？答案是不会，因为系统已经分配了这么多资源给这个app，他在没有再向系统申请更多内存或其他资源的情况下，会正常的得到cpu时间。因为Allocated的时已经分配给app的。


###drawable

* drawable  =  mdpi
* mdpi:   160dpi----1
* hdpi:   240dpi----1.5
* xhdpi:  320dpi----2
* xxhdpi: 480dpi----3  
* xxxhdpi:560dpi----3.5

其实drawable文件夹和drawable-mdpi文件夹一样，如果我们一张图a.png放在drawable中（160dpi），size是`100*100`的，但是我的手机是xxhdpi（480）的，这时如果直接读取bitmap，bitmap会是 `300*300` 的，因为系统发现xxhdpi中没有这张图，但是mdpi文件夹中有，但是因为手机是xxhdpi得，所以要把图片放大3倍来保证质量.

提示，大图必须各个文件夹去适配啊。或者读成bitmap，设置

	options.inDensity = resources.getDisplayMetrics().densityDpi;
    options.inScaled = true;
    //or use this method
    Chanel.getBitmapFitFromDrawable(Resources resources,int resId)
    
为什么会这样，我们看看`BitmapFactory.decodeResource的源码`

	/**
     * Synonym for opening the given resource and calling
     * {@link #decodeResourceStream}.
     *
     * @param res   The resources object containing the image data
     * @param id The resource id of the image data
     * @param opts null-ok; Options that control downsampling and whether the
     *             image should be completely decoded, or just is size returned.
     * @return The decoded bitmap, or null if the image data could not be
     *         decoded, or, if opts is non-null, if opts requested only the
     *         size be returned (in opts.outWidth and opts.outHeight)
     */
    public static Bitmap decodeResource(Resources res, int id, Options opts) {
        Bitmap bm = null;
        InputStream is = null; 
        
        try {
            final TypedValue value = new TypedValue();
            is = res.openRawResource(id, value);

            bm = decodeResourceStream(res, value, is, null, opts);
        } catch (Exception e) {
            /*  do nothing.
                If the exception happened on open, bm will be null.
                If it happened on close, bm is still valid.
            */
        } finally {
            try {
                if (is != null) is.close();
            } catch (IOException e) {
                // Ignore
            }
        }

        if (bm == null && opts != null && opts.inBitmap != null) {
            throw new IllegalArgumentException("Problem decoding into existing bitmap");
        }

        return bm;
    }
    
注意：`final TypedValue value = new TypedValue();
        is = res.openRawResource(id, value);`这两句代码，这里就是通过不同的drawable文件夹读取内容，value是其实是返回参数。如果是drawable，value.density = 0(0是TypedValue.DENSITY_DEFAULT)，而如果是读取drawable-xxhdpi，value.density就会变成480。其他文件夹不用多说了。
        
然后看`bm = decodeResourceStream(res, value, is, null, opts);`

	/**
     * Decode a new Bitmap from an InputStream. This InputStream was obtained from
     * resources, which we pass to be able to scale the bitmap accordingly.
     */
    public static Bitmap decodeResourceStream(Resources res, TypedValue value,
            InputStream is, Rect pad, Options opts) {

        if (opts == null) {
            opts = new Options();
        }

        if (opts.inDensity == 0 && value != null) {
            final int density = value.density;
            if (density == TypedValue.DENSITY_DEFAULT) {
                opts.inDensity = DisplayMetrics.DENSITY_DEFAULT;
            } else if (density != TypedValue.DENSITY_NONE) {
                opts.inDensity = density;
            }
        }
        
        if (opts.inTargetDensity == 0 && res != null) {
            opts.inTargetDensity = res.getDisplayMetrics().densityDpi;
        }
        
        return decodeStream(is, pad, opts);
    }
    
结果出来了，首先，如果没有设置opts.inDensity，就会根据value.density来设置opts.inDensity。而从上一个函数可以知道，value.density是根据这个drawable反正不同文件夹下确定的。如果是在drawable文件夹，value.density就是TypedValue.DENSITY_DEFAULT ，所以opts.inDensity = DisplayMetrics.DENSITY_DEFAULT;也就是160.

这时候发现如果没有设置opts.inTargetDensity，就会被设置成你device的显示设备的dpi。而最终decode出图片的size的缩放倍数，就是根据(inTargetDensity / inDensity)来决定的。

**注意：这个是没有考虑inSmallSize的情况下，另外inScale要设置为true**

所以如果我们一台xxhdpi的机器，在没有设置option的情况下，读取不属于xxhdpi的文件夹图片，bitmap会被放大！

####drawable系统如何进行best-match

>At runtime, the system ensures the best possible display on the current screen with the following procedure for any given resource:

>1. The system uses the appropriate alternative resource
Based on the size and density of the current screen, the system uses any size- and density-specific resource provided in your application. For example, if the device has a high-density screen and the application requests a drawable resource, the system looks for a drawable resource directory that best matches the device configuration. Depending on the other alternative resources available, a resource directory with the hdpi qualifier (such as drawable-hdpi/) might be the best match, so the system uses the drawable resource from this directory.
>2. If no matching resource is available, the system uses the default resource and scales it up or down as needed to match the current screen size and density
The "default" resources are those that are not tagged with a configuration qualifier. For example, the resources in drawable/ are the default drawable resources. The system assumes that default resources are designed for the baseline screen size and density, which is a normal screen size and a medium-density. As such, the system scales default density resources up for high-density screens and down for low-density screens, as appropriate.

>However, when the system is looking for a density-specific resource and does not find it in the density-specific directory, it won't always use the default resources. The system may instead use one of the other density-specific resources in order to provide better results when scaling. For example, when looking for a low-density resource and it is not available, the system prefers to scale-down the high-density version of the resource, because the system can easily scale a high-density resource down to low-density by a factor of 0.5, with fewer artifacts, compared to scaling a medium-density resource by a factor of 0.75.
        
        
####drawable的别名，拒绝复制

>Creating alias resources

When you have a resource that you'd like to use for more than one device configuration (but do not want to provide as a default resource), you do not need to put the same resource in more than one alternative resource directory. Instead, you can (in some cases) create an alternative resource that acts as an alias for a resource saved in your default resource directory.

Note: Not all resources offer a mechanism by which you can create an alias to another resource. In particular, animation, menu, raw, and other unspecified resources in the xml/ directory do not offer this feature.

For example, imagine you have an application icon, icon.png, and need unique version of it for different locales. However, two locales, English-Canadian and French-Canadian, need to use the same version. You might assume that you need to copy the same image into the resource directory for both English-Canadian and French-Canadian, but it's not true. Instead, you can save the image that's used for both as icon_ca.png (any name other than icon.png) and put it in the default res/drawable/ directory. Then create an icon.xml file in res/drawable-en-rCA/ and res/drawable-fr-rCA/ that refers to the icon_ca.png resource using the <bitmap> element. This allows you to store just one version of the PNG file and two small XML files that point to it. (An example XML file is shown below.)

Drawable

To create an alias to an existing drawable, use the <bitmap> element. For example:

	<?xml version="1.0" encoding="utf-8"?>
		<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    	android:src="@drawable/icon_ca" />
    
If you save this file as icon.xml (in an alternative resource directory, such as res/drawable-en-rCA/), it is compiled into a resource that you can reference as R.drawable.icon, but is actually an alias for the R.drawable.icon_ca resource (which is saved in res/drawable/).

但是这样做，图片还是会放大缩小的，不能达到Chanel.getBitmapFitFromDrawable的效果

参考：
[http://developer.android.com/guide/practices/screens_support.html](http://developer.android.com/guide/practices/screens_support.html)

[http://developer.android.com/guide/topics/resources/providing-resources.html#table2](http://developer.android.com/guide/topics/resources/providing-resources.html#table2)
