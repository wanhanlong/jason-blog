######动画的分类
* View动画
view动画通过对场景里的对象不断做图像变换（平移、缩放、旋转、透明度）从而产生动画效果，他是一种渐进式动画，并且支持自定义。
* 帧动画
帧动画是通过顺序播放一系列图像从而产生动画效果，可以简单理解为图片切换动画，如果[图片过大就会导致OOM]()。
* 属性动画
属性动画是通过动态改变对象的属性从而达到动画效果，他是API 11的新特性。

***

####View动画
view动画的作用对象是View，他支持四种动画效果，分别为：*平移动画、缩放动画、旋转动画 和 透明度动画*

###### 几个重要属性：
* view动画默认*执行一次*完成之后，view会瞬间回到初始位置；
* ta.setDuration(1000);// 设置动画时间
* ta.setFillAfter(true); 可以是动画执行完一次后控件停在当前位置
* ta.setRepeatCount(Animation.INFINITE); 可以设置动画循环次数，Animation.INFINITE为无限循环
* ta.setRepeatMode(Animation.REVERSE); 可以设置动画的重复模式，Animation.REVERSE为反方向执行，Animation.RESTART为重新开始


###### 平移动画

```
构造函数：
TranslateAnimation(float fromXDelta, float toXDelta, float fromYDelta, float toYDelta)

代码如下:
TranslateAnimation ta = new TranslateAnimation(0, 200, 0, 200);
ta.setDuration(1000);
ta.setFillAfter(true);
ta.setRepeatCount(Animation.INFINITE);
ta.setRepeatMode(Animation.RESTART);
view.startAnimation(ta);
```

######缩放动画
```
构造函数：
ScaleAnimation(float fromX, float toX, float fromY, float toY)
ScaleAnimation(float fromX, float toX, float fromY, float toY, float pivotX, float pivotY) 
ScaleAnimation(float fromX, float toX, float fromY, float toY, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)

frmoX/fromX 表示X/Y轴上的起始值
toX/toY 表示X/Y 轴上的结束值
pivotX/pivotY 表示缩放轴点X/Y坐标，会影响缩放效果（轴点就是View的中心点位置）
```
* 从左上顶点缩放
```
默认轴点是左上：
ScaleAnimation sa=new ScaleAnimation(0,2,0,2);
sa.setDuration(1000);
view.startAnimation(sa);
```
* 从中心缩放
```
ScaleAnimation sa = new ScaleAnimation(0, 2, 0, 2, RotateAnimation.RELATIVE_TO_SELF, 0.5f, RotateAnimation.RELATIVE_TO_SELF, 0.5f);
sa.setDuration(1000);
view.startAnimation(sa);
```

######旋转动画
```
构造函数：
RotateAnimation(float fromDegrees, float toDegrees, float pivotX, float pivotY)
RotateAnimation(float fromDegrees, float toDegrees, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)
fromDegrees 起始角度
toDegrees 旋转角度
pivotX/pivotY 旋转中心X/Y坐标

普通旋转：
RotateAnimation ra=new RotateAnimation(0,360,0,0);
ra.setDuration(1000);
view.startAnimation(ra);
中心旋转：
RotateAnimation ra=new RotateAnimation(0,360,RotateAnimation.RELATIVE_TO_SELF,0.5f,RotateAnimation.RELATIVE_TO_SELF,0.5f);
ra.setDuration(1000);
view.startAnimation(ra);
```

###### 透明度动画
```
 AlphaAnimation(float fromAlpha, float toAlpha)// 参数为起始透明度为结束透明度
AlphaAnimation aa=new AlphaAnimation(0,1);
// 0表示完全透明，1表示完全不透明
aa.setDuration(5000);
view.startAnimation(aa);
```

###### 动画集合
上边的动画都是单一的动画效果，Android还提供了一种动画集合，也就是说，几种单一动画组合在一起的动画效果。用到的类为：AnimationSet ；
```
构造函数：
AnimationSet(Context context, AttributeSet attrs)
AnimationSet(boolean shareInterpolator)
代码示例：
AnimationSet as = new AnimationSet(true);
TranslateAnimation ta = new TranslateAnimation(0, 200, 0, 200);
as.addAnimation(ta);
AlphaAnimation aa = new AlphaAnimation(0, 1);
as.addAnimation(aa);
RotateAnimation ra=new RotateAnimation(0,360,RotateAnimation.RELATIVE_TO_SELF,0.5f,RotateAnimation.RELATIVE_TO_SELF,0.5f);
as.addAnimation(ra);
as.setDuration(6000);
as.addAnimation(ta);
view.startAnimation(as);
```

######动画回调接口
在Android中可以通过Animation的setAnimationListener方法给view动画添加回调接口，如下:
```
TranslateAnimation ta = new TranslateAnimation(0, 200, 0, 200);
ta.setDuration(2000);
ta.setRepeatCount(Animation.INFINITE);
ta.setRepeatMode(Animation.RESTART);
view.startAnimation(ta);
ta.setAnimationListener(new Animation.AnimationListener() { 
   @Override
    public void onAnimationStart(Animation animation) {    }
    @Override
    public void onAnimationEnd(Animation animation) {    }
    @Override
    public void onAnimationRepeat(Animation animation) {    }
});
onAnimationStart 当动画开始执行
onAnimationEnd 当动画执行结束
onAnimationRepeat 当动画重复
尤其要注意onAnimationEnd，不是指每一次执行完，而是指整个动画过程执行完回调，
这样说可能还不明白，如上代码，如果设置了ta.setRepeatCount(Animation.INFINITE);时，
动画会无限循环，这个时后，回调过程是：onAnimationStart -->onAnimationRepeat -->onAnimationStart -->onAnimationRepeat -->onAnimationStart ······-->onAnimationStart -->onAnimationRepeat ,因为动画会无限循环，所以onAnimationEnd永远不会被执行到
```
================================================
以上的这些View动画效果都是通过Java代码来实现的，Android还可以在xml中来设置View动画效果，这俩种方式的效果是等价的。同样，xml中既可以设置单一动画，也可以设置动画集合。
```
单一动画：
res/anim/animtest.xml:
<?xml version="1.0" encoding="utf-8"?>
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromXDelta="0"
    android:fromYDelta="0"
    android:toXDelta="200"
    android:toYDelta="200"
    android:duration="2000">
</translate>
Java代码：
Animation animation = AnimationUtils.loadAnimation(TestAnimActivity.this, R.anim.animtest);
view.startAnimation(animation);
--------------------------------------
动画集合：
res/anim/animtest.xml:
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:duration="2000"
        android:fromXDelta="0"
        android:fromYDelta="0"
        android:toXDelta="200"
        android:toYDelta="200">
    </translate>
    <alpha
        android:fromAlpha="0"
        android:toAlpha="1"
        android:duration="2000"/>
</set>
注意为集合中的每个子动画设置动画时间android:duration="2000"，这个属性是集合中每个子动画都必须要设置的，，否则没有设置的那个子动画会瞬间执行完，而设置了这个属性的子动画会按我们设定的时间执行。
java代码同上
```
**xml中设置View动画的一些重要属性**
* <set>标签对应AnimationSet类
* android：interpolator
表示动画集合采用的插值器，它影响动画的速度，通过它可以来设置非匀速动画。
* android：shareInterpolator
表示集合中动画是否和集合共享同一个插值器，如果集合不指定插值器，子动画就需要单独指定插值器或使用系统默认的


================================================

#### 帧动画
帧动画是顺序播放一组预先定义好的图片来达到动画效果，它主要用到AnimationDrawable类，它的使用也很简单：
```
xml:
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="false">
    <item android:drawable="@drawable/image1" android:duration="500"/>
    <item android:drawable="@drawable/image2" android:duration="500"/>
    <item android:drawable="@drawable/image3" android:duration="500"/>
</animation-list>
Java代码：
view.setBackgroundResource(R.anim.animtest);
AnimationDrawable drawable = (AnimationDrawable)view.getBackground();
drawable.start();
```


[【200多种Android动画效果的强悍框架】](http://finalshares.com/read-651)
[【仿简书、淘宝等等App的View弹出效果】](http://www.jianshu.com/p/a697d2a38b3c)
