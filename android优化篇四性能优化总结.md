Android作为一种移动设备，无论是内存还是CPU性能都会有一定的限制，无法和PC设备相比拟，有鉴于此，Android程序不可能无限制的使用内存和CPU。**过多的使用内存会导致程序每寸溢出即OOM,而过多的使用CPU资源（比如做大量耗时任务）会导致手机变得卡顿甚至出现程序无响应（即ANR）的情况。** 所以作为Androider，我们对性能优化的技能就尤为重要，这里主要从[【布局优化】、【绘制优化】](https://github.com/jasonLYF/jason-blog/blob/master/android%E4%BC%98%E5%8C%96%E7%AF%87%E4%B8%80%E5%B8%83%E5%B1%80%E5%8F%8A%E7%BB%98%E5%88%B6%E4%BC%98%E5%8C%96.md)、[【内存泄漏优化】](https://github.com/jasonLYF/jason-blog/blob/master/android%E4%BC%98%E5%8C%96%E7%AF%87%E4%BA%8C%E5%86%85%E5%AD%98%E4%BC%98%E5%8C%96.md)、【响应速度优化】、[【列表优化】](https://github.com/jasonLYF/jason-blog/blob/master/android%E4%BC%98%E5%8C%96%E7%AF%87%E4%B8%89ListView%E4%BC%98%E5%8C%96.md)、[【Bitmap优化】](http://www.jianshu.com/p/635fceca82d3)、[【线程优化】](http://www.jianshu.com/p/3da543063b8c) 等方面进行优化。

## 一、布局优化
1. 尽量保持布局层级的扁平化，避免出现重复的嵌套布局
2. 在Android中单独的布局性能： FrameLayout>LinearLayout>RelativeLayout
3. 安卓SDK中有一个Hierarchy Viewer工具可以帮助你优化自己的布局

## 二、绘制优化
绘制优化主要是指View的onDraw方法避免执行大量的操作以及避免过度绘制：
1. onDraw方法中不要创建新的局部对象。这是因为onDraw方法可能会被频繁的调用，这样就会在一瞬间产生大量的临时对象，这不仅会占用更多的内存而且还会导致系统更加频繁gc，降低了程序的执行效率
2. onDraw方法中不要做耗时任务，也不要执行太多次的循环操作。这样会造成View的绘制不流畅。
3. 移除掉window的默认纯色背景
`【在setContentView()之后调用getWindow().setBackgroundDrawable(null)】`或者`【在theme中添加 android:windowbackground="null" 】`
4. 移除XML布局文件中非必需的Background
5. 合理使用draw9patch
6. 能不用Alpha尽量不要用Alpha，因为它会绘制俩次
7. 按需显示占位背景图片
8. 通过减少invalidate()方法的调用来减少onDraw()方法的调用，同时，尽可能的使用四个参数的invalidate()方法，如果是没有参数的invalidate()方法会绘制整个view
9. 可以的话 使用硬件加速

##### [更多布局及绘制优化方案请查看我的另一篇博客](http://www.jianshu.com/p/ab15167c48da)

## 三、内存的优化
* **内存泄漏：** 是指程序在申请内存后，无法释放已申请的内存空间。内存泄漏简单地说就是申请了一块内存空间，使用完毕后没有释放掉。它的一般表现方式是程序运行时间越长，占用内存越多，最终用尽全部内存，整个系统崩溃。由程序申请的一块内存，且没有任何一个指针指向它，那么这块内存就泄露了。内存泄漏一般不会导致程序异常，但它会导致程序的内存占用过大，这将提高内存溢出的几率。所以，内存泄露是内存溢出的一种诱因，不是唯一因素。
内存泄漏是造成应用程序OOM的主要原因之一
* **内存溢出：** 是指程序在申请内存时，没有足够的内存空间供其使用，出现out of memory；比如申请了一个integer,但给它存了long才能存下的数，那就是内存溢出。

#### 导致内存泄漏的原因及解决方案：
###### 1、静态变量导致的内存泄漏
**静态变量：** 静态变量是在类被load的时候分配内存的，并且存在于方法区。当类被卸载的时候，静态变量被销毁。只要静态变量没有被销毁也没有置null，其对象一直被保持引用，也即引用计数不可能是0，因此不会被垃圾回收。

###### 2、[单列模式](http://www.jianshu.com/p/0dd09187baeb)导致的内存泄漏
当单列中传入一个Activity的Context后，该单列就持有Activity的引用，我们知道单例的生命周期和Application的一样长，所以当Activity退出时它的内存并不会被回收。
所以，单例中的Context最好使用Application的Context,这样可以有效防止内存泄漏

###### 3、[属性动画](http://www.jianshu.com/p/156976d1eb01)导致的内存泄漏
属性动画中有一类无限循环的动画，如果在Activity中播放此动画且没有在onDestory中停止动画，即使已经无法在界面上看到动画效果，动画也会一直播放下去，并且这个时候Activity的View会被动画持有，而View有持有了Activity，最终导致Activity无法释放。这种泄漏的解决办法是在onDestory中调用animator.cancal()来停止动画。

###### 4、Handler造成的内存泄漏
大家平时开发中喜欢在Activity中直接new一个Handler的匿名内部类，在Java中，非静态的内部类或者匿名类会隐式的持有其外部类的引用，而静态的内部类则不会。这样造成匿名内部类持有一个外部类(通常是Activity)的引用(不然怎么更新ui)，但是Handler常常伴随着一个执行耗时操作的异步线程(如下载多张图片)，如果在完成耗时操作之前，Activity退出，异步线程持有handler的引用，handler持有Activity的引用，从而导致内存泄漏

###### 5、上下文对象导致的内存泄漏
* 使用application的context来替代activity相关的context。
尽量避免activity的context在自己的范围外被使用，这样会导致activity无法释放。不要让生命周期长于Activity的对象持有到Activity的引用
* 在Android中，Application Context的生命周期和应用的生命周期一样长，而不是取决于某个Activity的生命周期。
如果想保持一个长期生命的对象，并且这个对象需要一个 Context，就可以使用Application对象。可以通过调用Context.getApplicationContext()方法或者 Activity.getApplication()方法来获得Application对象。
* Drawable的对象的内部Callback持有activity的引用，当Activity finish()之后，静态成员drawable始终持有这个Activity的引用，导致内存释放不了。
* Activity内部如果有一个Context的成员变量，将导致Context引用指向的Activity对象释放不了,见上文：静态变量导致的内存泄漏

###### 6、线程造成的内存泄漏

###### 7、webview导致的内存泄漏优化
请参考：http://lipeng1667.github.io/2016/08/06/memory-optimisation-for-webview-in-android/

###### 8、其他一些导致内存泄漏的原因   
1、Bitmap对象使用完后，忘记了调用recycle()方法销毁；   
2、解析图片的时候忘记了设置采样率 详见下文    
3、[自定义View](http://www.jianshu.com/p/7468e038825a)时TypedArray使用完后忘记调用recycle()方法释放内存     
4、ListView的适配器类中没有复用convertView   
5、未采用软引用等   
* 软引用：
如果一个对象只具有软引用，那么如果内存空间足够，垃圾回收器就不会回收它；如果内存 空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。软引用可以和一个引用队 列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。
* 弱引用：
如果一个对象只具有弱引用，那么在垃圾回收器线程扫描的过程中，一旦发现了只具有弱引 用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。弱 引用也可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联 的引用队列中。
* 弱引用与软引用的根本区别在于：
只具有弱引用的对象拥有更短暂的生命周期，可能随时被回收。而只具有软引用的对象只有当内存不够的时候才被回收，在内存足够的时候，通常不被回收。

6、即时关闭InputStream/OutputStream。   
7、注册某个对象后没有注销，如广播    
8、集合对象没清理造成的内存泄漏。把大量对象的引用放入集合中，但我们不需要该对象时，记得从集合中将不需要的引用清理掉，同理，当对象不需要时，记得将对象的引用设置为null。    
9、资源文件未关闭造成内存泄漏。最常见的是文件流执行完读写操作后，忘记关闭了输入流，输出流；数据库、Content Provider操作完后Cursor忘记了close等等。

#### 如何避免内存溢出：
###### 减小对象的内存占用
避免OOM的第一步就是要尽量减少新分配出来的对象占用内存的大小，尽量使用更加轻量的对象   
1. 使用更加轻量的数据结构   
2. 避免在Android里面使用Enum   
3. 减小Bitmap对象的内存占用   
	* inSampleSize：缩放比例，在把图片载入内存之前，我们需要先计算出一个合适的缩放比例，避免不必要的大图载入。   
	* decode format：解码格式，选择ARGB_8888/RBG_565/ARGB_4444/ALPHA_8，存在很大差异。

###### 内存对象的重复利用，减少对象的重复创建，从而减少内存的分配与回收
1. 复用系统自带的资源
字符串/颜色/图片/动画/样式以及简单布局等这些都可以重复利用
2. 列表中ConvertView的复用
3. Bitmap对象的复用
在ListView与GridView等显示大量图片的控件里面需要使用LRU的机制来缓存处理好的Bitmap
4. 避免在onDraw方法里面执行对象的创建
5. 善用 StringBuilder

[关于内存优化请查看我的另一篇较详细的博客](http://www.jianshu.com/p/9fa5c242048a)

## 四、响应速度优化
###### ANR (Application Not Responding)出现的原因：
* activity如果5秒内无法响应屏幕触摸事件或者键盘输入事件；
* BroadcastRecever10秒内还未执行完操作

###### ANR解决方法：
* 尽量不要在主线程中耗时操作。运行在主线程里的任何方法都尽可能少做事情。特别是，Activity应该在它的关键生命周期方法（如onCreate()和onResume()）里尽可能少的去做创建操作。
* 应用程序应该避免在BroadcastReceiver里做耗时的操作或计算。
* 避免在Intent Receiver里启动一个Activity，因为它会创建一个新的画面，并从当前用户正在运行的程序上抢夺焦点。如果你的应用程序在响应Intent广 播时需要向用户展示什么，你应该使用Notification Manager来实现。

**响应速度的优化思想** 是避免在主线程中做耗时操作，可以把耗时操作放到子线程中执行，即采用异步处理。当程序anr时，系统会在/data/anr/目录下创建一个traces.txt，通过分析此文件从而获取造成anr的原因。导出此文件的方法：```adb pull /data/anr/traces.txt```

## 五、[列表优化](http://www.jianshu.com/p/b7741023bc6f)
1、复用convertView，减少findViewById的次数
2、避免在getView方法中做耗时操作
3、采用ViewHolder模式缓存item条目的引用
4、给listView设置滚动监听器 根据不同状态 不同处理数据 分批分页加载 根据listView的状态去操作，比如当列表快速滑动时不去开启大量的异步任务去请求图片
5、listview每个item层级结构不要太复杂
6、listview每个item中异步加载图片，并对图片加载做优化
7、listview每个item中不要创建线程
8、采用硬件加速来使ListView的滑动更流畅
[更多列表内容](http://www.jianshu.com/p/b7741023bc6f)

## 六、[线程优化](http://www.jianshu.com/p/3da543063b8c)
线程的优化思想是采用线程池，避免程序中出现大量的Thread，线程池可以重用内部的线程，从而避免了线程的创建和销毁带来的性能开销，同时线程池还能有效的控制线程的最大并发数，避免大量的线程因抢占系统资源从而导致线程阻塞的发生

## 七、[Bitmap优化](http://www.jianshu.com/p/98c88f9ceafa)
[Android中的Bitmap](http://www.jianshu.com/p/98c88f9ceafa)
[Bitmap缓存策略](http://www.jianshu.com/p/635fceca82d3)

## 八、其他的一些性能优化建议
1. 避免创建过多的对象
2. 不要过多的使用枚举，枚举占用的内存空间要比整型大
3. 常量请使用static final来修饰
4. 使用一些Android特有的数据结构，比如SparseArray和Pair等，他们都具有更好的性能
5. 适当的使用软银用和弱引用
6. 采用内存缓存和磁盘缓存
7. 尽量采用静态内部类，这样可以避免潜在的由于内部类导致的内存泄漏

避免内存溢出参考文章：http://hukai.me/android-performance-oom/
推荐文章：https://realm.io/cn/news/droidcon-farber-improving-android-app-performance/
