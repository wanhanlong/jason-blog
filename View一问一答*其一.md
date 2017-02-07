***
View坐标
***
###1、View的坐标参数 主要有哪些？分别有什么注意的要点？
**答：**   
几个主要坐标参数是：   
1）Left，Right，top,Bottom；它们表示的并非是距离屏幕左上方的绝对值，而是表示 view 和 他的父控件的相对坐标值，并且代表View的初始坐标，在绘制完毕后就不会再改变 。   
2）X和Y 表示的是View左上角相对于父控件的坐标值，即实时相对坐标。   
3）TranslationX,TranslationY 这2个值 默认都为0，表示的是相对于父控件的左上角的偏移量。   
它们之间的换算关系是：   
x=left+translationX , y=top+translationY；   
width = right-left;   
height = bottom-top;   
Left = getLeft();// 获取 left，right，top，bottom

###2、View中的几个重要方法？
**答：**   
1）public final void measure(int widthMeasureSpec, int heightMeasureSpec)

* measure 过程决定了View的宽高，Measure完成后可以通过getMeasureWidth和getMeasureHeight方法获取到view的测量后的宽高，在几乎所有的情况下都会等于最终view的宽高
* measure()方法接收两个参数，widthMeasureSpec和heightMeasureSpec，这两个值分别用于确定视图的宽度和高度的规格和大小。   

2）protected void onLayout(boolean changed, int left, int top, int right, int bottom)

* layout 过程决定了View的四个顶点的坐标和实际的View的宽高，完成以后可以通过getTop,getBottom,getLeft,getRight来获取View的四个顶点位置，并通过getWidth,getHeight获取View的最终宽高

3）protected void onDraw(Canvas canvas)   

draw过程则决定了View的显示，完成draw后view会显示在屏幕上

* 绘制背景(background.draw(Canvas))
* 绘制自己
    protected void onDraw(Canvas canvas)
    onDraw绘制自己，新建一个paint 在canvas上绘制自己的图形
* 绘制children (dispatchDraw)
    dispatchDraw会遍历调用所有子元素的draw方法
* 绘制装饰（onDrawScrollBars）

4）public boolean isEnabled() 当前视图是否可用。   
可以调用setEnable()方法来改变视图的可用状态，传入true表示可用，传入false表示不可用。
它们之间最大的区别在于，不可用的视图是无法响应onTouch事件的。

5）public boolean isFocused()  当前视图是否获得焦点   
通常情况下有两种方法可以让视图获得焦点，即通过键盘的上下左右键切换视图，以及调用requestFocus()方法。而现在的Android手机几乎都没有键盘了，因此基本上只可以使用requestFocus()这个办法来让视图获得焦点了。
而requestFocus()方法也不能保证一定可以让视图获得焦点，它会有一个布尔值的返回值，如果返回true说明获得焦点成功，返回false说明获得焦点失败。一般只有视图在focusable和focusable in touch mode同时成立的情况下才能成功获取焦点，比如说EditText。

6）public void offsetTopAndBottom(int offset)及 public void offsetLeftAndRight(int offset)   
offsetTopAndBottom直接改变的是top, bottom, 相当于在parent中上下平移View的位置；   
offsetLeftAndRight直接改变的是left, right, 相当于在parent中左右平移View的位置；   
View的边界直接发生了变化，又因为View和他的子View的相对位置没变，所以他的子View的边界也跟着变化了。

###3、获取View的位置坐标失败问题怎么处理？
如果我们要获取一个view的位置坐标，可以直接子findviewbyid之后获取，也可以在view的点击事件中获取，当然也可以在其他地方获取。如果我们直接在finedviewbyid之后获取有时候就会获取失败，获取到的值是0；   
分析原因可能有以下三种情况： 1）view的宽高本身就是0；2）View的visibility属性为gone；3）视图还未绘制完成，当然未绘制完成也表现在不同的方面，比如，activity所代表的界面还没显示出来没有添加到WindowPhone的DecorView上；要获取的view没有被添加到DecorView上。   
我们主要要说的是第三种情况，可以怎么解决呢？   
**答：**   
1）在View的事件回调里获取；这时候该view已经被显示即被添加到DecorView上 如点击事件 触摸事件 焦点事件等
```
        view1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
              // doing
            }
        });
```
2）在activity被显示出来时即添加到了DecorView上时获取宽和高如 onWindowFocusChanged() 回调方法

    @Override
    public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);
        // doing
    }
        boolean measure;
        @Override
        public void onWindowFocusChanged(boolean hasFocus) {
            super.onWindowFocusChanged(hasFocus);
            if (hasFocus && !measure) {
                measure = true;
                int width = new TextView(this).getMeasuredWidth();
                int height = new TextView(this).getMeasuredHeight();
            }
        }

3）在onResume方法最后开线程300毫秒左右后获取宽和高 因为onResume执行完后300毫秒后 界面就显示出来了

    @Override
    protected void onResume() {
        super.onResume();
        view1.postDelayed(new Runnable() {
            @Override
            public void run() {
                // doing
            }
        }, 300);
    }

4）在onCreate()或onResume()等方法中需要获取宽高时使用getViewTreeObserver().addOnGlobalLayoutListener()
来为view添加回调在回调里获得宽度或者高度获取完后让view删除该回调

        view1.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {
                // doing
            }
        });

5）view.measure(int widthMeasureSpec,int heightMeasureSpec)
手动对view进行measure来得到view的宽高

***
View滑动
***
###1、让view滑动总共有几种方式，分别要注意什么？都适用于那些场景？
1）通过view本身提供的scrollTo和scrollBy来滑动；但是只能滑动view的内容 不可以滑动view本身。比如textview 调用这2个方法  滑动的就是显示出来的字的内容。[详情查看](http://www.jianshu.com/p/2b48551d5319)   
2）用动画来滑动View；

 * 补间动画平移view，是对view的影像的操作，它不能真正改变view的位置参数
 * 属性动画平移view需要兼容3.0以下版本
 * 利用补间动画将view平移

		TranslateAnimation ta = new TranslateAnimation(0, 500, 0, 200);
 		ta.setDuration(2000);
 		ta.setFillAfter(true);
		 view.startAnimation(ta);

 * 利用属性动画平移view
 * 
		ObjectAnimator.ofFloat(view,"translationX",0,500).setDuration(2000).start();  
		ObjectAnimator.ofFloat(view,"translationY",0,500).setDuration(2000).start();  


3）通过改变view的LayoutParams使得view重新布局从而实现滑动   
比如把一个Button向右平移100PX有下面俩种思路：

    * 将Button的LayoutParams里的marginLeft参数值增加100px；
    * 在Button的左边放一默认宽度为0的View，当我们想想右平移Button时，重置View的宽度为100px即可，这里利用了布局占位挤压的特性。
		ViewGroup.MarginLayoutParams params = (ViewGroup.MarginLayoutParams) view.getLayoutParams();
		params.leftMargin += 500;

###2、三种滑动方式的比较
* scrollTo和scrollBy这种方式使用简单操作方便，但是它只能滑动view的内容，并不能直接滑动view本身
* 补间动画滑动，是对view影像的操作，它同样不能真正改变view的位置参数
* 属性动画滑动可以改变view的位置参数，但是需要兼容3.0以下版本
* LayoutParams这种方式，使用起来稍有麻烦，使用于有交互的view

###3、使用动画来实现view的滑动 有什么后果？
答：实际上view动画 是对view的表面ui 也就是给用户呈现出的视觉效果 来做的移动，动画本身并不能移动view的真正位置。属性动画除外。动画播放结束以后，view最终还是会回到自己的位置的，。当然了你可以设置fillafter 属性 来让动画播放结束以后 view表象停留在 变化以后的位置。所以这会带来一个很严重的后果。比如你的button在屏幕的左边，你现在用个动画 并且设置了fillafter属性让他去了右边。你会发现 点击右边的button 没有click事件触发，但是点击左边的 却可以触发，原因就是右边的button 只是view的表象，真正的button 还在左边没有动过。你一定要这么做的话 可以提前在右边button移动后的位置放一个新的button，当你动画执行结束以后  把右边的enable 左边的让他gone就可以了。

###4、view的滑动渐变效果有哪些方法？
三种，   
第一种是scroller 也是使用最多的。问题7里有解释。   
第二种就是动画，动画我就不多说了，不属于本文范畴。   
第三种也是我们经常使用的就是用handler ，每隔一个时间间隔 来更新view的状态。

###5、Scroller是干嘛的？
Scroller就是用于 让view有滑动渐变效果的。

###6、onTouchEvent和GestureDetector 在什么时候用哪个比较好？
只有滑动需求的时候 就用前者，如果有双击等这种行为的时候 就用后者。

喜欢就关注我(ˇˍˇ) 
更多内容请关注 [ 我的专题 ](http://www.jianshu.com/collection/bcc2c1ba8378)
