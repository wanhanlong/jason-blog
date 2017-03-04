前俩篇文章介绍了[ View动画和帧动画的使用 ](http://www.jianshu.com/p/17250c3b1d4a)以及[如何用View动画打造炫酷的ListView]()；这一篇我们重点来学习属性动画。

属性动画是API 11新加入的特性，相比于View动画，它更有**优势：**
* View动画的作用对象只能是View，而属性动画对作用对象进行了扩展，它可以作用于任何对象不仅仅是View，也就是说，**属性动画能实现view动画所不能实现的效果**
* 属性动画的动画效果也进行了扩展，不再像View动画那样只能支持四种简单的变换，属性动画中有ValueAnimator、ObjectAnimator和AnimatorSet概念。

###属性动画的使用：
示例一：
```
ObjectAnimator.ofFloat(view,"translationY",view.getHeight()).start();
ObjectAnimator.ofFloat(view,"translationX",view.getHeight()).start();
```
以上这俩行代码就是俩个动画效果，第一行代码的意义是：改变view对象的translationY属性，让其沿着Y轴向下平移(平移的距离就是他自己的高度)；第二行代码是让其在X轴上平移；同样的，这俩航代码也可以同时作用于view对象上，让其同时在X和Y轴上平移，类似于view动画里的动画集合。

示例二：
```
ValueAnimator valueAnimator = ObjectAnimator.ofInt(view,"backgroundColor",/*Red*/0xFFFF8080,/*Blue*/0xFF8080FF);
valueAnimator.setDuration(3000);
valueAnimator.setEvaluator(new ArgbEvaluator());
valueAnimator.setRepeatMode(ValueAnimator.REVERSE);
valueAnimator.setRepeatCount(ValueAnimator.INFINITE);
valueAnimator.start();
动画效果为:颜色渐变，无限循环
```

示例三：动画集合
```
AnimatorSet set = new AnimatorSet();
set.playTogether(
        ObjectAnimator.ofFloat(view, "rotationX", 0, 360),
        ObjectAnimator.ofFloat(view, "rotationY", 0, 180),
        ObjectAnimator.ofFloat(view, "rotation", 0, -90),
        ObjectAnimator.ofFloat(view, "translationX", 0, 90),
        ObjectAnimator.ofFloat(view, "translationY", 1, 90),
        ObjectAnimator.ofFloat(view, "scaleX", 1, 1.5f),
        ObjectAnimator.ofFloat(view, "scaleY", 1, 0.5f),
        ObjectAnimator.ofFloat(view, "alpha", 1, 0.25f, 1)
);
set.setDuration(5000);
set.start();
```
####属性动画的监听器
普通的View动画可以添加监听器，同样的属性动画也可以添加监听器：主要有俩个监听接口：
* AnimatorListener
接口定义如下：
```
public static interface AnimatorListener {
    void onAnimationStart(Animator animation);
    void onAnimationEnd(Animator animation);
    void onAnimationCancel(Animator animation);
    void onAnimationRepeat(Animator animation);
}
```
同时，系统还提供了AnimatorListenerAdapter，它是AnimatorListener 的实现类，这样就可以有选择的实现我们所需要的接口方法
使用时直接addListener（）即可。
* AnimatorUpdateListener
接口定义如下：
```
public static interface AnimatorUpdateListener {
    void onAnimationUpdate(ValueAnimator animation);
}
```
这个接口会监听整个动画过程，动画是由许多帧组成的，每播一帧，onAnimationUpdate就会被调用一次。

另外，有以下俩个方法可以移除监听：
```
void removeListener(AnimatorListener listener);
void removeAllListeners();
```

###在xml中使用属性动画
属性动画需要定义在res/animator/目录下
######先看属性动画的标签
* <objectAnimator> 对应 ObjectAnimator
* <valueAnimator>  对应 ValueAnimator 
* <set>  对应 AnimatorSet  
* <animator>比<valueAnimator>标签少了一个android:propertyName=""属性

######属性动画各个属性的含义
1. android:propertyName="string" 表示动画作用对象的属性名称
* android:valueFrom="float | int | color" 表示属性的起始值
* android:valueTo="float | int | color" 表示属性的结束值
* android:duration="int" 表示动画的时长
* android:repeatCount="int" 表示动画的重复次数，它的默认值是0，-1表示无限循环，
* android:repeatMode="restart | reverse" 表示动画的重复次数，restart 表示连续重复（动画每次都重新开始播放），reverse表示逆向重复（类似于收尾接龙，第一次从头开始，第二次倒着播放，第三次再从头开始，，依次....）
* android:valueType="intType | floatType" 表示android:propertyName所指定的属性的类型，有intType | floatType俩个选项，分别表示属性的类型为整形和浮点型，如果android:propertyName所指定的属性表示的是颜色，就不需要指定android:valueType，系统会自动对颜色类型的属性做处理
* android:startOffset="int" 表示动画的延迟时间，即动画开始后延迟N毫秒后才会真正的播放动画

###对任意属性做动画
######属性动画原理：
属性动画要求动画的作用对象提供该属性的get和set方法，属性动画根据外界传递的该属性的初始值和最终值，以动画的效果多次调用set方法，随着时间的推移，set方法所传递的值越来越接近最终值。所以，我们要想对object的abc属性做动画且让动画生效，要**满足以下条件：**
1. object必须提供abc的set方法
* 如果动画的时候没有传递初始值，object还必须提供abc的get方法，因为系统要去获取abc属性的初始值
* object的setAbc方法对属性abc所做的改变必须能够通过某种方法反映出来，如果不满足，我们也就看不到动画效果了

######改变控件的宽度
```
改变View的宽度：
ObjectAnimator.ofInt(myView, "height", 1500).setDuration(5000).start();
改变Button的宽度：
ObjectAnimator.ofInt(myTextView, "height", 1500).setDuration(5000).start();
```

分别运行上述俩航代码，发现无论是view还是textview，都没有达到我们想要的结果，且view时会提示我们“com.art W/PropertyValuesHolder: Method setWidth() with type int not found on target class class android.view.View”

通过属性动画原理我们很容易明白，ObjectAnimator.ofInt(myView, "height", 1500).setDuration(5000).start();之所以没有效果，是因为View中没有提供setWidth属性，不满足上述条件**1**；而Button中提供了setWidth和getWidth方法，      
![QQ截图20160612163458.png](http://upload-images.jianshu.io/upload_images/1479978-c05da06bfba94e0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)那为什么还是没有效果呢？细看setWidth方法，发现，这个方法设置的是view的最大和最小宽度，而不是view的宽度，所以，只满足了条件**1、2**，不满足条件**3**。

可是View及其子类是我们开发中很常用的控件，我们对它的操作也很多，这怎么办呢？针对上述问题，官方提供给另外我们**三种解决方案：**
######1. 我们自己给对象加上对应的set和get方法；当然需要权限
这种方法看似简单，但实际操作起来却也困难匆匆，一个首要的问题就是我们要有权限给属性加上set和get方法，但实际上，我们很难做到。
######2. 用一个类来包装原对象，间接的提供set和get方法
```
@Overridepublic void onClick(View v) {
    ViewWrapper wrapper = new ViewWrapper(view);
    ObjectAnimator.ofInt(wrapper, "width", 1000).setDuration(5000).start();
}
private static class ViewWrapper {
    private View mTarget;
    public ViewWrapper(View Target) {
        mTarget = Target;
    }
    public int getWidth() {
        return mTarget.getLayoutParams().width;
    }
    public void setWidth(int mWidth) {
        mTarget.getLayoutParams().width = mWidth;
        mTarget.requestLayout();
    }
}
```
######3. 采用ValueAnimator监听动画过程，自己实现属性的变化
```
本文上面关于属性监听时说过“每播一帧，onAnimationUpdate就会被调用一次。”

ValueAnimator valueAnimator = ValueAnimator.ofInt(1, 100);
valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    // 估值器
    private IntEvaluator mEvaluator = new IntEvaluator();
    @Override
    public void onAnimationUpdate(ValueAnimator animation) {
        // 获得当前动画的进度，整数 ，1~100之间
        int currentValue = (Integer) animation.getAnimatedValue();
        Log.i("=========", "currentValue=" + currentValue);
        // 获得当前进度占整个动画过程的比例，浮点型 ，0~1之间
        float fraction = animation.getAnimatedFraction();
        // 直接调用整数估值器，通过比例计算出宽度，然后设置给Button
        view.getLayoutParams().width = mEvaluator.evaluate(fraction, view.getLayoutParams().width, 1000);
        view.requestLayout();
    }
});
valueAnimator.setDuration(5000);
valueAnimator.start();
```

###属性动画的向下兼容
上文说过，属性动画是API 11新加入的特性，也就是说我们是无法在API 11以下的版本中使用属性动画的，那我们怎么向下兼容呢？
我们可以使用开源动画库[nineoldandroids](http://nineoldandroids.com/)来兼容以前的版本。

