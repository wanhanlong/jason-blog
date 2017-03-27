
[Glide源码](https://github.com/bumptech/glide)   
上一篇[ Glide的基础用法 ](https://github.com/jasonLYF/jason-blog/blob/master/Glide%E7%9A%84%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)

### Glide load a Bitmap
某些情况下，我们只是想从网络获取一个Bitmap，但是不会加载到View中，比如加载的控件类型不是ImageView，而是加载图片到自定义 view ，或者加载为Background的形式，或者要对load下来的Bitmap做特殊处理等。   

这种情况下，使用 Glide 的 ViewTarget 就很容易实现：

* load 一个 GlideDrawable 对象
```
        Glide.with(this).load(url).into(new SimpleTarget<GlideDrawable>() {
            @Override
            public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> glideAnimation) {

            }
        });
```
* load 一个 Bitmap 对象
```
        Glide.with(this).load(url).asBitmap().into(new SimpleTarget<Bitmap>() {
            @Override
            public void onResourceReady(Bitmap resource, GlideAnimation<? super Bitmap> glideAnimation) {

            }
        });
```
* load 一个 Bitmap 对象
```
        Glide.with(this).load(url).asBitmap().into(new BitmapImageViewTarget(imageView) {
            @Override
            protected void setResource(Bitmap resource) {

            }
        });
```

###### 关于内存回收
如上述写法所示，Glide允许你在 .into() 方法中去声明 target 的匿名内部类。但是这样做有一个弊端：   

这大大增加了这样一个可能性，即在 Glide 做完图片请求之前， Android 垃圾回收移除了这个匿名内部类对象。最终这可能会导致图像加载完成了，但是回调再也不会被调用。   

所以，尽量保证声明的回调对象是作为一个字段对象的，这样你就可以保护它避免被Android 垃圾回收机制回收。

```
    private SimpleTarget target = new SimpleTarget<Bitmap>() {
        @Override
        public void onResourceReady(Bitmap bitmap, GlideAnimation glideAnimation) {

        }
    };
```
调用时：   
` Glide.with(context).load(url).asBitmap().into(target);`

###### 关于图片大小
我们可以在回调中指定图片的大小来加载合适的图片节省内存。

```
    private SimpleTarget target2 = new SimpleTarget<Bitmap>(250, 250) {
        @Override
        public void onResourceReady(Bitmap bitmap, GlideAnimation glideAnimation) {

        }
    };
```

----------

### 用 [GlideModule](https://github.com/bumptech/glide/wiki/Configuration) 自定义 Glide 的配置
GlideModule 在 Glide 中用来改变 Glide 的全局配置。

使用时首先要创建一个 xxGlideModule 类去实现 GlideModule 接口；该接口有俩个抽象方法：   
`void applyOptions(Context context, GlideBuilder builder);`该方法可以进行Glide的全局配置；       
`void registerComponents(Context context, Glide glide);`该方法中可以注册ModelLoader

然后我们需要在 Manifest 文件中申明这个刚刚创建的类,否则不会被加载或被使用：   
`<meta-data android:name="com.test.glide.MyGlideModule" android:value="GlideModule"/>`

其次，混淆时注意配置 GlideModule   
`-keepnames class * com.test.glide.MyGlideModule`
或者：
`-keep public class * implements com.bumptech.glide.module.GlideModule`

**GlideModule 之 applyOptions(Context context, GlideBuilder builder)**

###### 自定义图片质量
Glide 默认的 Bitmap 格式是 RGB_565，该格式为每个像素使用 2 个字节，这也造成了一个问题就是图片质量不高，比如我们想使用 ARGB_8888 格式（每个像素使用 4 个字节）时就可以通过自定义配置来实现:
```
    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        /**
         * 调整图片加载格式
         */
        builder.setDecodeFormat(DecodeFormat.PREFER_ARGB_8888);
    }
```

**调整 Glide 的内存缓存大小**
```
    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        MemorySizeCalculator calculator = new MemorySizeCalculator(context);
        /**
         * 默认
         */
        int defaultMemoryCacheSize = calculator.getMemoryCacheSize();// 内存缓存大小
        int defaultBitmapPoolSize = calculator.getBitmapPoolSize();// bitmap 池的大小
        /**
         * 计算新的大小
         * 这里调整为默认大小的 2 倍
         */
        int customMemoryCacheSize = (int) (2 * defaultMemoryCacheSize);
        int customBitmapPoolSize = (int) (2 * defaultBitmapPoolSize);
        /**
         * //重设缓存大小
         */
        builder.setMemoryCache(new LruResourceCache(customMemoryCacheSize));
        builder.setBitmapPool(new LruBitmapPool(customBitmapPoolSize));
    }
```
注意：   
setMemoryCache 方法倒是很好理解，这里要重点强调一下 setBitmapPool 方法。   
Bitmap 池用来允许不同尺寸的Bitmap被重用，这可以显著地减少因为图片解码像素数组分配内存而引发的垃圾回收。   
默认情况下Glide使用LruBitmapPool作为Bitmap池，LruBitmapPool采用LRU算法保存最近使用的尺寸的Bitmap。

**调整 Glide 的磁盘缓存大小、目录**   
```
    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        int cacheSize = 100 * 1024 * 1024;//缓存大小为 10 M
        /**
         * 大小
         * 这里把磁盘缓存大小设为 100 M
         * 注意：这两个方法二选一，不能一起使用
         */
        // 缓存到应用的私有目录
        builder.setDiskCache(new InternalCacheDiskCacheFactory(context, cacheSize));
        // 缓存到外部存储
        // builder.setDiskCache(new ExternalCacheDiskCacheFactory(context, cacheSize));

        /**
         * 调整磁盘缓存目录和名称
         */
        // 缓存目录，文件夹名称，缓存大小
        String downloadDirectoryPath = Environment.getDownloadCacheDirectory().getPath();
        builder.setDiskCache(new DiskLruCacheFactory(downloadDirectoryPath, "glideCache", cacheSize));
        
         /**
         * 直接实现DiskCache.Factory接口
         */
        builder.setDiskCache(
                new DiskCache.Factory() {
                    @Override
                    public DiskCache build() {
                        return DiskLruCacheWrapper.get(File directory, int maxSize);
                    }
                });
    }
```

[star Github](https://github.com/jasonLYF/jason-blog)
上一篇[ Glide的基础用法 ](https://github.com/jasonLYF/jason-blog/blob/master/Glide%E7%9A%84%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)

![](http://upload-images.jianshu.io/upload_images/1479978-0ff1a43230b41689.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
