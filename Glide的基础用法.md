加载使用Glide时首先要   
`compile 'com.github.bumptech.glide:glide:3.7.0'`    
`<uses-permission android:name="android.permission.INTERNET" />`   
Glide加载一张图片的模是：   
`Glide.with(this).load(url).into(imageView)；`   
接下来看看他都有哪些方法。

#### .with()方法
![](http://upload-images.jianshu.io/upload_images/1479978-f1bd0f6e584cb2f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 可以看到Glide的with方法接受的参数有：`Context、Activity、Fragment、FragmentActivity`；Glide会自动从`Activity、Fragment、FragmentActivity`中获取它们的Context‘；
* 我们还可以用`getApplicationContext()`方法获取全局Context，当你在Activity生命周期之外获取图片时就可以用这个方法。
* 接受`Activity、Fragment、FragmentActivity`作为这个方法的参数有一个好处就是会使图片加载会和`Activity、Fragment、FragmentActivity`的生命周期绑定在一起。

#### .load()方法
![](http://upload-images.jianshu.io/upload_images/1479978-8f12496fce3d4d7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个方法的参数告诉我们Glide允许我们加载不同途径的图片   

1. load(String string) 从网络加载   
实际上它的参数可以为一个文件路径、uri或者url
```
Glide.with(this).load(url).into(imageView);
 或
String path = "file://"+ Environment.getExternalStorageDirectory().getPath()+"/mypic.jpg";
Glide.with(this).load(path).into(imageView);
```

2. load(File file) 从文件加载   
` Glide.with(this).load(file).into(imageView);`

3. load(Integer resourceId) 从Resource中加载   
`Glide.with(this).load(R.mipmap.ic_launcher).into(imageView);`
4. load(Uri uri) 从Uri获取   
```
Uri uri = Uri.parse("android.resource://" + this.getPackageName() + "/" + R.mipmap.ic_launcher);
Glide.with(this).load(uri).into(imageView);
```
这是从资源文件中生成的Uri，实际上可以是任何Uri

5. load(byte[] model) 
```
Bitmap bmp = BitmapFactory.decodeResource(getResources(), R.mipmap.error);
byte[] model = Bitmap2Bytes(bmp);
Glide.with(this).load(model).into(imageView);
```
```
public byte[] Bitmap2Bytes(Bitmap bm) {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        bm.compress(Bitmap.CompressFormat.PNG, 100, baos);
        return baos.toByteArray();
}
```

#### DrawableTypeRequest
.load()方法返回一个DrawableTypeRequest对象，DrawableTypeRequest的类继承关系如下：
`public class DrawableTypeRequest<ModelType> extends DrawableRequestBuilder<ModelType> implements DownloadOptions `

接下来就看看在load()方法后还有哪些public方法供我们在加载时使用

######  1 .placeholder()
占位图，也就是从开始加载到加载结束这段时间内显示的默认图片；
两个重载方法`placeholder( int resourceId)`和`placeholder(Drawable drawable)`；   `placeholder( int resourceId)`方法也可以用颜色来作为占位图；

```
Glide.with(this).load(url).placeholder(R.mipmap.loading).into(imageView);
Glide.with(this).load(url).placeholder(R.color.colorAccent).into(imageView);
```

###### 2 .error()
异常图，也就是加载失败后显示的默认图片
和`placeholder`一样，也有两个重载方法，也可以用颜色作为异常图   
`Glide.with(this).load(gifUrl).error(R.mipmap.error).into(imageView);`
###### 3 .fallback()
设置model为空时显示的Drawable。

###### 4 .skipMemoryCache()
跳过内存缓存，也就是说加载的图片不会缓存到内存中，且它并不会影响到磁盘缓存策略。   
如果不调用此方法Glide默认会缓存到内存中去，所以我们没必要调用.skipMemoryCache(false)；   
`Glide.with(this).load(gifUrl).skipMemoryCache(true).into(imageView);`   
另外，Glide也提供了清除内存缓存的方法：
`Glide.get(context).clearMemory();`   

注意，这个方法只能在UI线程中调用，否则会报错：

![](http://upload-images.jianshu.io/upload_images/1479978-d7a0fe4afb9764af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 5 .diskCacheStrategy()
在说这个方法前，了解下Glide磁盘缓存策略，Glide的磁盘缓存策略很高大上，同一个URL它可能会缓存多份图像，为什么这么说呢，举个列子：   
你要加载的图片原始大小是100x100，可是ImageView的大小是50x50，   

Glide会自动判断ImageView的大小，然后只将这么大的图片像素加载到内存当中，帮助我们节省内存开支，我们也就完全不用担心图片内存浪费，甚至是内存溢出的问题。   
这个时候Glide即会缓存原始图像，也会缓存转换后的图像。

所以我们可以用这个方法来设置磁盘缓存策略，此方法的参数给定了四个值：

![](http://upload-images.jianshu.io/upload_images/1479978-de753ed8572c1515.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* DiskCacheStrategy.ALL  缓存所有版本图片，这是Glide默认磁盘缓存策略
* DiskCacheStrategy.NONE  不缓存
* DiskCacheStrategy.SOURCE  缓存原始图片
* DiskCacheStrategy.RESULT  缓存最终图像，也就是根据ImageView大小转换后的图片

`Glide.with(this).load(url).diskCacheStrategy(DiskCacheStrategy.RESULT).into(imageView);`

另外，Glide提供了清空磁盘缓存的方法，这个方法却必须在后台线程中调用：   
`Glide.get(getApplicationContext()).clearDiskCache();`

###### 6 .override()
虽然说Glide会自动判断ImageView的大小去加载相应的图片，但是不排除某些情况下我们需要去指定加载的大小，此时就用这个方法    
`Glide.with(this).load(url).override(200,200).into(imageView);`   
注意，单位是像素，我们需要自己去计算

###### 7 .CenterCrop() & .fitCenter()
这两个方法和ImageView的`android:scaleType="centerCrop | fitCenter"`的意义类似；   
`CenterCrop()`缩放图像让它填充到 ImageView 界限内并且裁剪额外的部分，ImageView 可能会完全填充，但图像可能不会完整显示。   
`fitCenter() `缩放图像让图像测量出来等于或小于 ImageView 的边界范围。该图像将会完全显示，但可能不会填满整个 ImageView。
`Glide.with(this).load(url).fitCenter().into(imageView);`
###### 8 .thumbnail()
缩略图，先加载缩略图 然后在加载全图，可用于加载中显示的图片   
`Glide.with(this).load(url).thumbnail(0.1f).into(imageView);`   
当然，还可以加载一个其他URL的图片作为缩略图
```
DrawableRequestBuilder<String> thumbnailRequest = Glide.with(this).load(thumbUrl);
Glide.with(this).load(url).thumbnail(thumbnailRequest).into(imageView);
```
###### 9 .priority()
设置请求的优先级，此方法给定了如下几个可选值,它们的优先级依次递减：
![](http://upload-images.jianshu.io/upload_images/1479978-3768e5ba096fa3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Glide.with(this).load(url).priority(Priority.HIGH).into(imageView);`

###### 10 .crossFade() & .dontAnimate() & .animate()
* .crossFade() 表示淡入淡出效果
* .dontAnimate() 表示无动画效果
* .animate() 可以加载我们自己定义的动画

###### 11 .listener()
监听请求状态，此方法需要一个RequestListener类型参数，有两个回调方法；   
onException返回true表示我们自己处理掉了异常，false表示交给Glide去处理；方法最好返回false，这样的话Glide会自动回调.error()；   
onResourceReady返回true表示用户自己已经设置好资源，包括截取操作，动画操作之类的，准备好显示；false表示交给Glide。   

```
Glide.with(this)
.load(url)
.listener(new RequestListener<String, GlideDrawable>() {
  @Override
  public boolean onException(Exception e, String model, Target<GlideDrawable> target, boolean isFirstResource) {
    return false;
  }
  @Override
  public boolean onResourceReady(GlideDrawable resource, String model, Target<GlideDrawable> target, boolean isFromMemoryCache, boolean isFirstResource) {
    return false;
  }
})
.error(R.mipmap.error)
.into(imageView);
```

这种匿名类写法每次都会创建新的监听器，造成了不必要的内存申请和引用，我们可以创建一个公共的监听器来统一处理。

###### 12 Gif的加载
Glide一个牛叉的功能就是允许我们去加载Gif格式的图片，而且还非常简单和加载静态图一样，不需要特殊的处理。如下，只要你传人一个gif的URL即可，Glide会自动去识别它是不是Gif；   
`Glide.with(this).load(gifUrl).into(imageView);`

* .asBitmap() 无论你传入的URL对应的是不是一个Gif，Glide都会把它当做静态图，这时的静态图会停留在Gif的第一帧
* .asGif() 无论你传入的URL对应的是不是一个Gif，Glide都会把它当做Gif来处理，如果不是，Glide会回调.error()方法

###### 13 Glide播放本地视频
`Glide.with(this).load(Uri.fromFile(new File(filePath))).into(imageView);`   
注意：Glide只可播放本地视频文件，不可播放网络文件。


本篇文章就记录这些Glide的基础使用，下一篇记录它的更多功能。

![](http://upload-images.jianshu.io/upload_images/1479978-0ff1a43230b41689.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)