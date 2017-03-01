

##View的事件分发
View的事件分发在Android中很重要！！！很重要！！！很重要！！！

![](http://upload-images.jianshu.io/upload_images/1479978-8fac4ae821631a81.gif?imageMogr2/auto-orient/strip)

###1、为什么会有事件分发机制？
我们知道，android的布局结构是树形结构，这就会导致一些View可能会重叠在一起，当我们手指点击的地方在很多个布局范围之内，也就是说此时有好多个布局可以响应我们的点击事件，这个时候该让哪个view来响应我们的点击事件呢？这就是事件分发机制存在的意义。

###2、ViewGroup的事件分发涉及到哪些过程和方法？
![](http://upload-images.jianshu.io/upload_images/1479978-d2f0c0e7ba6e2cc0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**public boolean dispatchTouchEvent(MotionEvent ev)**   
	是事件分发机制中的核心，所有的事件调度都归它管   
	用来进行事件的分发，如果事件能够传递给当前View，那么此方法一定会被调用   
**public boolean onInterceptTouchEvent(MotionEvent ev)**   
	在dispatchTouchEvent中调用，用来判断是否拦截某个事件，返回结果表示是否拦截当前事件   
**public boolean onTouchEvent(MotionEvent event)**   
	在dispatchTouchEvent中调用，用来处理点击事件，返回结果表示是否消耗当前事件

###3、View中为什么会有dispatchTouchEvent方法，它存在的意义是什么？
我们知道View可以注册很多监听事件（下文有详细），比如，触摸事件，单击事件，长按事件等，而且view也有自己的onTouchEvent方法，那么这么多事件应该由谁来调度管理呢？这就是是View中dispatchTouchEvent方法存在的意义。

###4、View中为什么没有onInterceptTouchEvent事件拦截方法？
View最为事件传递的最末端，要么消费掉事件，要么不处理进行回传，根本没必要进行事件拦截

###5、用伪代码表示ViewGroup的事件分发过程并解释？
    public boolean dispatchTouchEvent(MotionEvent ev) {
        boolean consume = false;
        if (onInterceptTouchEvent(ev)) {
            consume = onTouchEvent(ev);
        } else {
            consume = child.dispatchTouchEvent(ev);
        }
        return consume;
    }
对于一个ViewGroup来说，点击事件产生后，首先会传递给它，这时她的dispatchTouchEvent会被调用，如果这个ViewGroup的onInterceptTouchEvent
方法返回true表示它要拦截当前事件，接着事件就会交给这个ViewGroup处理，即它的onTouchEvent就会被调用；如果这个这个ViewGroup的onInterceptTouchEvent
方法返回false就表示它不拦截当前事件，这时事件就会传递给子元素，接着子元素的dispatchTouchEvent方法就会被调用，如此反复直到事件最终被处理。

###6、简述事件传递的流程
* 事件都是从Activity.dispatchTouchEvent()开始传递
* 一个事件发生后，首先传递给Activity，然后一层一层往下传，从上往下调用dispatchTouchEvent方法传递事件：   
	`activity  -->  ~~ -->  ViewGroup  --> View`  
* 如果事件传递给最下层的View还没有被消费，就会按照反方向回传给Activity，从下往上调用onTouchEvent方法，最后会到Activity的onTouchEvent()函数，如果Activity也没有消费处理事件，这个事件就会被抛弃：   
	`View  -->  ViewGroup -->  ~~ -->  Activity`
* dispatchTouchEvent方法用于事件的分发，Android中所有的事件都必须经过这个方法的分发，然后决定是自身消费当前事件还是继续往下分发给子控件处理。返回true表示不继续分发，事件没有被消费。返回false则继续往下分发，如果是ViewGroup则分发给onInterceptTouchEvent进行判断是否拦截该事件。
* onTouchEvent方法用于事件的处理，返回true表示消费处理当前事件，返回false则不处理，交给子控件进行继续分发。
* onInterceptTouchEvent是ViewGroup中才有的方法，View中没有，它的作用是负责事件的拦截，返回true的时候表示拦截当前事件，不继续往下分发，交给自身的onTouchEvent进行处理。返回false则不拦截，继续往下传。这是ViewGroup特有的方法，因为ViewGroup中可能还有子View，而在Android中View中是不能再包含子View的
* 上层View既可以直接拦截该事件，自己处理，也可以先询问(分发给)子View，如果子View需要就交给子View处理，如果子View不需要还能继续交给上层View处理。既保证了事件的有序性，又非常的灵活。
* 事件由父View传递给子View，ViewGroup可以通过onInterceptTouchEvent()方法对事件拦截，停止其向子view传递
* 如果View没有对ACTION_DOWN进行消费，之后的其他事件不会传递过来，也就是说ACTION_DOWN必须返回true，之后的事件才会传递进来

###7、ViewGroup 和 View 同时注册了事件监听器(onClick等)，哪个会执行?   
事件优先给View，会被View消费掉，ViewGroup 不会响应。

###8、当俩个或多个View重叠时，事件该如何分配？
当 View 重叠时，一般会分配给显示在最上面的 View，也就是后加载的View。

###9、dispatchTouchEvent每次都会被调用吗？
是的，onInterceptTouchEvent则不会。

###10、一旦有事件传递给view，view的onTouchEvent一定会被调用吗？
View没有onInterceptTouchEvent方法，一旦有事件传递给它，他的onTouchEvent就一定会被调用。

###11、ViewGroup 默认拦截事件吗？
ViewGroup默认不拦截任何事件；看源码可以知道ViewGroup的onInterceptTouchEvent方法中只有一行代码：`return false；`

###12、事件分为几个步骤？
down事件开头，up事件结尾，中间可能会有数目不定的move事件。

##View事件的优先级
![](http://upload-images.jianshu.io/upload_images/1479978-8fac4ae821631a81.gif?imageMogr2/auto-orient/strip)

###1、基于监听的事件分发有哪些？怎么来设置监听？
我们常用的setOnClickListener、OnLongClickListener、setOnTouchListener等都是基于监听的事件处理。   
设置监听可以用如下几种方式：
 
1. 匿名内部类：

        view.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                
            }
        });
2. 内部类：

 	   	view.setOnClickListener(new MyClickListener());
 		   class MyClickListener implements View.OnClickListener
 		   {
 		       @Override
 	 	      public void onClick(View v) {

  	 	     }
 		   }
3. 外部类：

		view.setOnClickListener(new MyClickListener());
		public class MyClickListener implements View.OnClickListener {
 		   @Override
		    public void onClick(View v) {
 	       
 		   }
		}
4. Activity实现OnClickLister接口的方式

		public class TestViewActivity extends AppCompatActivity implements View.OnClickListener {
  		  MyView view;
   		 @Override
   		 protected void onCreate(Bundle savedInstanceState) {
   		     super.onCreate(savedInstanceState);
     		   setContentView(R.layout.activity_test_view);
     		   view = (MyView) findViewById(R.id.view);
     		   view.setOnClickListener(this);
    		}

   		 @Override
   		 public void onClick(View v) {
  		  }
		}
5. 在xml中绑定的方式：

		public class TestViewActivity extends AppCompatActivity{
    		MyView view;
    		@Override
    		protected void onCreate(Bundle savedInstanceState) {
     		   super.onCreate(savedInstanceState);
      		  setContentView(R.layout.activity_test_view);
      		  view = (MyView) findViewById(R.id.view);
    		}
    		public void MyClick(View view){

    		}
		}
    	<com.art.chapter_3.MyView
      	  android:id="@+id/view"
      	  android:layout_width="100dip"
      	  android:layout_height="100dip"
      	  android:background="@color/colorPrimaryDark"
        android:onClick="MyClick"/>

###2、view的onTouchEvent，OnClickListerner和OnTouchListener的onTouch方法 三者优先级如何？
代码验证：

	自定义view：
	public class MyView extends View {
  	  @Override
  	  public boolean onTouchEvent(MotionEvent event) {
     	   Log.i("--------","MyView onTouchEvent "+MyAction.getActionType(event));
    	    return super.onTouchEvent(event);
   	  }
	}

	监听：
        yelloe.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View view, MotionEvent motionEvent) {
                Log.i("--------", "touch  yelloe  " + MyAction.getActionType(motionEvent));
                return false;
            }
        });

        yelloe.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Log.i("--------", "click  yelloe  ");
            }
        });
	输出结果： 插图 优先级高低

> 优先级高低：
onTouchListener  >>>  onTouchEvent >>> setOnLongClickListener  >>>  OnClickListerner 

###3、如图有三個嵌套的控件,结构如下,其中黄色部分是一个继承于View的控件,绿色和红色都是继承于LinearLayout的控件: 插图：
代码简单如下：

	public class MyView extends View {
		@Override
		public boolean onTouchEvent(MotionEvent event) {
			Log.i("--------","MyView onTouchEvent "+MyAction.getActionType(event));
			return super.onTouchEvent(event);
		}
	}

	public class MyLinearLayoutRed extends LinearLayout {
		@Override
		public boolean onTouchEvent(MotionEvent event) {
			Log.i("--------","MyLinearLayoutRed onTouchEvent "+MyAction.getActionType(event));
			return super.onTouchEvent(event);
		}
	}

	public class MyLinearLayoutGreen extends LinearLayout {
		@Override
		public boolean onTouchEvent(MotionEvent event) {
			Log.i("--------","MyLinearLayoutRed onTouchEvent "+MyAction.getActionType(event));
			return super.onTouchEvent(event);
		}
	}

	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
		xmlns:tools="http://schemas.android.com/tools">
		<com.example.administrator.myviewevent.MyLinearLayoutRed
			android:id="@+id/red"
			android:layout_width="300dip"
			android:layout_height="300dip"
			android:background="@color/red">
			<com.example.administrator.myviewevent.MyLinearLayoutGreen
				android:id="@+id/green"
				android:layout_width="200dip"
				android:layout_height="200dip"
				android:background="@color/green">
				<com.example.administrator.myviewevent.MyView
					android:id="@+id/yellow"
					android:layout_width="130dip"
					android:layout_height="130dip"
					android:background="@color/yellow" />
			</com.example.administrator.myviewevent.MyLinearLayoutGreen>
		</com.example.administrator.myviewevent.MyLinearLayoutRed>
	</FrameLayout>


问题一：如果不在onTouchEvent方法中做任何处理，只是Log输出每一层的Touch事件类型,现在用手指按下在黄色区域并移动后抬起.请问Log输出的结果是什么?   
答：   
I/--------: MyView onTouchEvent ACTION_DOWN...   
I/--------: MyLinearLayoutGreen onTouchEvent ACTION_DOWN...   
I/--------: MyLinearLayoutRed onTouchEvent ACTION_DOWN...   

问题二：如果不在onTouchEvent方法和setOnTouchListener的onTouch方法中做任何处理，只是Log输出每一层的Touch事件类型,现在用手指按下在黄色区域并移动后抬起.请问Log输出的结果是什么?    
在Activity中增加setOnTouchListener监听   

        yelloe.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View view, MotionEvent motionEvent) {
                Log.i("--------", "touch  yelloe  " + MyAction.getActionType(motionEvent));
                return false;
            }
        });
        green.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View view, MotionEvent motionEvent) {
                Log.i("--------", "touch  green  " + MyAction.getActionType(motionEvent));
                return false;
            }
        });

        red.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View view, MotionEvent motionEvent) {
                Log.i("--------", "touch  red  " + MyAction.getActionType(motionEvent));
                return false;
            }
        });

答：   
I/--------: touch  yelloe  ACTION_DOWN...   
I/--------: MyView onTouchEvent ACTION_DOWN...   
I/--------: touch  green  ACTION_DOWN...   
I/--------: MyLinearLayoutGreen onTouchEvent ACTION_DOWN...   
I/--------: touch  red  ACTION_DOWN...   
I/--------: MyLinearLayoutRed onTouchEvent ACTION_DOWN...   

###4、setOnTouchListener中onTouch的返回值表示什么意思？   
onTouch方法返回true表示事件被消耗掉了，不会继续传递了,此时获取不到到OnClick和onLongClick事件；onTouch方法返回false表示事件没有被消耗，可以继续传递，此时，可以获取到OnClick和onLongClick事件；   
同理 onTouchEvent 和 setOnLongClickListener 方法中的返回值表示的意义一样；   

###5、setOnLongClickListener的onLongClick的返回值表示什么？
返回false，长按的话会同时执行onLongClick和onClick；如果setOnLongClickListener返回true，表示事件被消耗，不会继续传递，只执行longClick；

###6、onTouch和onTouchEvent的异同？
 * onTouch方法是View的 OnTouchListener接口中定义的方法。当一个View绑定了OnTouchLister后，当有touch事件触发时，就会调用onTouch方法。（当把手放到View上后，onTouch方法被一遍一遍地被调用）
 * onTouchEvent方法是override 的Activity的方法。重新了Activity的onTouchEvent方法后，当屏幕有touch事件时，此方法就会被调用。
 * onTouch优先于onTouchEvent执行，如果在onTouch方法中通过返回true将事件消费掉，onTouchEvent将不会再执行。
 * 相同点是它们都是在在View的dispatchTouchEvent中调用的；
 
###7、点击事件的传递过程？
Activity-Window-View。   
从上到下依次传递，当然了如果你最低的那个view onTouchEvent返回false 那就说明他不想处理 那就再往上抛，都不处理的话最终就还是让Activity自己处理了。

###8、如果某个view 处理事件的时候 没有消耗down事件 会有什么结果？
假如一个view，在down事件来的时候 他的onTouchEvent返回false， 那么这个down事件 所属的事件序列 就是他后续的move 和up 都不会给他处理了，全部都给他的父view处理。

###9、如果view 不消耗move或者up事件 会有什么结果？
那这个事件所属的事件序列就消失了，父view也不会处理的，最终都给activity 去处理了。

###10、enable是否影响view的onTouchEvent返回值？
不影响，只要clickable和longClickable有一个为真，那么onTouchEvent就返回true。

##View的滑动冲突】
![](http://upload-images.jianshu.io/upload_images/1479978-8fac4ae821631a81.gif?imageMogr2/auto-orient/strip)

###1、常见滑动冲突场景
场景1 —— 外部滑动方向与内部滑动方向不一致，比如ViewPager中包含ListView;   
场景2 —— 外部滑动方向与内部滑动方向一致，比如ScrollView中包含ListView;   
场景3 —— 上面两种情况的嵌套

###2、滑动冲突处理规则？
通过判断是水平滑动还是竖直滑动来判断到底应该谁来拦截事件；可以根据水平和竖直两个方向的距离差或速度差来做判断

###3、滑动冲突解决方式？
* 外部拦截法 —— 即点击事件先经过父容器的拦截处理，如果父容器需要此事件就拦截，不需要就不拦截，需要重写父容器的onInterceptTouchEvent方法；在onInterceptTouchEvent方法中，首先ACTION_DOWN这个事件，父容器必须返回false,即不拦截ACTION_DOWN事件，因为一旦父容器拦截了ACTION_DOWN,那么后续的ACTION_MOVE/ACTION_UP都会直接交给父容器处理；其次是ACTION_MOVE,根据需求来决定是否要拦截;最后ACTION_UP事件,这里必须要返回false,在这里没有多大意义。
* 内部拦截法 —— 所有事件都传递给子元素,如果子元素需要就消耗掉,不需要就交给父元素处理,需要子元素配合requestDisallowInterceptTouchEvent方法才能正常工作;父元素需要默认拦截除ACTION_DOWN以外的事件,这样子元素调用parent.requestDisallowInterceptTouchEvent(false)方法时，父元素才能继续拦截需要的事件。（ACTION_DOWN事件不受requestDisallowInterceptTouchEvent方法影响,所以一旦父元素拦截ACTION_DOWN事件,那么所有元素都无法传递到子元素去）。

###4、requestDisallowInterceptTouchEvent 可以在子元素中干扰父元素的事件分发吗？如果可以，是全部都可以干扰吗？
答：肯定可以，但是down事件干扰不了。

![](http://upload-images.jianshu.io/upload_images/1479978-8fac4ae821631a81.gif?imageMogr2/auto-orient/strip)

喜欢就关注我(ˇˍˇ) 
[View一问一答*View坐标及其滑动](http://www.jianshu.com/p/d212e806da32)
更多内容请关注 [ 我的专题 ](http://www.jianshu.com/collection/bcc2c1ba8378)
[Star我的Github](https://github.com/jasonLYF/)

![我的微信公众号](http://upload-images.jianshu.io/upload_images/1479978-d49921e504b51973.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)