本文，没有分析，完全是记录一些经验~

#### setContentView()
* 读取应用的资源数据
* 解析资源数据，展开布局
* 布局展开成为Activity的顶层视图

> 此调用花费的时间取决于布局的复杂性：资源数据越大解析越慢，而更多的类也让布局实例化变慢;所以，要想要让应用的启动时间变快，我们就要尽量减少布局展开花费的时间。


#### 选用合适的布局方式来减少实例化对象的个数和布局层级，保持布局的扁平化
* 相同效果用尽量少的布局和控件来实现；
* 相同的效果，布局尽量不要嵌套太深，默认的最深布局深度是10
* 尽量保持布局层级的扁平化，避免出现重复的嵌套布局
* 减少布局中的枝叶，如果一个布局没有子 View 或者背景，那么他可以被移除掉（况且他本身就是不可见的）来让布局更有效
* 减少父母层级，如果一个布局没有兄弟，并且他不是 ScrollView 或者根 View，并且也没有背景，那么他就可以直接被移除掉，他的孩子可以直接被移到他父母的层级下
* 在Android中单独的布局性能： FrameLayout>LinearLayout>RelativeLayout

#### 使用<include>标签重用重复的布局
###### 使用场景
* 多次使用相同的布局
* 布局有一个通用的组成部分
* 布局依赖于设备配置比如横竖屏切换

###### 使用方式
1. 将将要被重用的布局抽离出来成为一个新的布局child.xml
2. 在想要重用child.xml的地方引用
```
<include android:id="@+id/inLayout"
android:layout="@layout/children"/>
```

#### 使用<merge/>标签来合并布局
###### 使用场景
a. 布局顶结点是FrameLayout且不需要设置background或padding等属性，可以用merge代替,因为系统默认会在我们的布局外包装一层FrameLayout
b. 某布局作为子布局被其他布局include时，使用merge当作该布局的顶节点，这样在被引入时顶结点会自动被忽略，而将其子节点全部合并到主布局中

###### 使用方式
使用时直接将FrameLayout替换成<merge>标签即可

#### ViewStub懒加载
我们经常碰到的一种情况就是在某个视图中，某些view不需要在一开始就显示，只在某些特定的条件下才会显示出来，常用的做法就是把这个view在初始化的时候用INVISIBLE或者GONE进行隐藏，这样做虽然能达到我们想要的效果，但是也有一个不小的**弊端：应用中的每一个控件和布局文件都需要经过初始化，排列位置和绘制三个过程，利用 INVISIBLE只是隐藏布局，但是布局还是占居当前位置，且系统在加载布局的时候这一部分还是会绘制出来，同样花费绘制时间。**这个时候我们就需要用到ViewStub了，**ViewStub是一种非常轻量级的不可见的视图控件，它没有大小，没有绘制功能，也不参与布局，资源消耗非常低，将它放置在布局当中基本可以认为是完全不会影响性能的。ViewStub采用了推迟初始化技术，它可以推迟实例化提高性能，并且如果不触发初始化的话就不会初始化会节省这部分的内存。**
###### 使用方式
1. 在xml中定义ViewStub
```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.administrator.myapplication.MainActivity">
    <ViewStub
        android:id="@+id/mystub"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout="@layout/stub_layout" />
</RelativeLayout>
```

2. 在Activity中打开ViewStub
```
ViewStub view = (ViewStub)findViewById(R.id.mystub);
view.setVisibility(View.VISIBLE);
或者：
View stub = findViewById(R.id.mystub);
View inflatedView = stub.inflate();
```

#### 避免过渡绘制
我们可以在设备上查看过度绘制的区域：设置->开发者选项->调试GPU过渡绘制->打开；打开后设备会通过不同的颜色显示绘制程度，从最优开始：蓝，绿，淡红，红；红色说明过渡绘制严重。所以，我们在布局时候，尽量减少红色 Overdraw，使其看到更多的蓝色区域。
* 减少背景颜色的设置来减少Overdraw
* 移除掉window的默认纯色背景
`【在setContentView()之后调用getWindow().setBackgroundDrawable(null)】或者【在theme中添加 android:windowbackground="null" 】`
* 移除XML布局文件中非必需的Background
* 能不用Alpha尽量不要用Alpha，因为它会绘制俩次
* 按需显示占位背景图片
* 通过减少invalidate()方法的调用来减少onDraw()方法的调用，同时，尽可能的使用四个参数的invalidate()方法，如果是没有参数的invalidate()方法会绘制整个view

#### 其他绘制优化
* onDraw方法中不要创建新的局部对象。这是因为onDraw方法可能会被频繁的调用，这样就会在一瞬间产生大量的临时对象，这不仅会占用更多的内存而且还会导致系统更加频繁gc，降低了程序的执行效率
* onDraw方法中不要做耗时任务，也不要执行太多次的循环操作。这样会造成View的绘制不流畅
* 可以的话 使用硬件加速

#### 减少不必要的infalte
对于inflate的布局可以直接缓存，用全局变量代替局部变量，避免下次需再次inflate，比如ListView中的ViewHolder模式

#### 其他一些关于View的Tips
* 做好适配，尽量为所有分辨率创建资源，这样可以减少一些不必要的缩放
* **用SurfaceView或TextureView代替普通View**
SurfaceView或TextureView可以通过将绘图操作移动到另一个单独线程上提高性能。普通View的绘制过程都是在主线程(UI线程)中完成，如果某些绘图操作影响性能就不好优化了，这时我们可以考虑使用SurfaceView和TextureView，他们的绘图操作发生在UI线程之外的另一个线程上。因为SurfaceView在常规视图系统之外，所以无法像常规试图一样移动、缩放或旋转一个SurfaceView。TextureView是Android4.0引入的，除了与SurfaceView一样在单独线程绘制外，还可以像常规视图一样被改变。
* 通过减少invalidate()方法的调用来减少onDraw()方法的调用，同时，尽可能的使用四个参数的invalidate()方法，如果是没有参数的invalidate()方法会绘制整个view


欢迎大家更贴补充更多内容~
 
---
[view的触摸事件MotionEvent](http://www.jianshu.com/p/48c3323f334f)   
[view通过scrollTo和scrollBy滑动](http://www.jianshu.com/p/2b48551d5319)

喜欢就关注我(ˇˍˇ) 想～    
更多内容请关注 [ 我的专题 ](http://www.jianshu.com/collection/bcc2c1ba8378)   
转载请注明 [ 原文链接: ](http://www.jianshu.com/users/c1b4a5542220/latest_articles)
http://www.jianshu.com/p/ab15167c48da

![](http://upload-images.jianshu.io/upload_images/1479978-0ff1a43230b41689.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
