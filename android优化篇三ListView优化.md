#### ListView怎么和ScrollView兼容？
我们知道，有些时候我们需要在ListView外层嵌套一层ScrollView，代码如下：
```
    <ScrollView
        android:id="@+id/scrollview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <ListView
            android:id="@+id/listview"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"></ListView>
    </ScrollView>
```
只要稍微有点经验的人都知道这是会出现什么问题，没错，就是“Listview不能显示正常的条目，只显示一条或二条”，这是怎么回事呢？**这是因为：由于listView在scrollView中无法正确计算它的大小, 故只显示一行。**
当目前为止，我知道的针对这一问题的**解决办法有：**
###### 1. 方法一：重写ListView, 覆盖onMeasure()方法
```
activity_list_view_scroll_view_test.xml:
<merge xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.art.demo.ListViewScrollViewTestActivity">
    <ScrollView
        android:id="@+id/scrollview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <com.art.demo.WrapperListView
            android:id="@+id/listview"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>
    </ScrollView>
</merge>

WrapperListView.java:
public class WrapperListView extends ListView {
    public WrapperListView(Context context) {
        super(context);
    }
    public WrapperListView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
    public WrapperListView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
    public WrapperListView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
    }
    /**
     * 重写该方法，达到使ListView适应ScrollView的效果
     */
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);
        super.onMeasure(widthMeasureSpec, expandSpec);
    }
}

ListViewScrollViewTestActivity.java:
public class ListViewScrollViewTestActivity extends AppCompatActivity {

    private ScrollView scrollView;
    private WrapperListView listView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_list_view_scroll_view_test);
        scrollView = (ScrollView) findViewById(R.id.scrollView);
        listView = (WrapperListView) findViewById(R.id.listview);
        initListVeiw();
    }

    private void initListVeiw() {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 20; i++) {
            list.add("第 " + i + " 条");
        }
        listView.setAdapter(new ArrayAdapter<String>(this,
                android.R.layout.simple_list_item_1, list));
    }
}

另外，哪位大神可以告诉我在代码块(```)中，怎么给某一行加粗，或者做一些其他明显标记？？？？？？？？？？？？？？
```
###### 2. 方法二：动态设置listview的高度，不需要重写ListView
只需要在setAdapter之后调用如下方法即可：
```
public void setListViewHeightBasedOnChildren(ListView listView) {
        // 获取ListView对应的Adapter
        ListAdapter listAdapter = listView.getAdapter();
        if (listAdapter == null) {
            return;
        }
        int totalHeight = 0;
        for (int i = 0, len = listAdapter.getCount(); i < len; i++) {
            // listAdapter.getCount()返回数据项的数目
            View listItem = listAdapter.getView(i, null, listView);
            // 计算子项View 的宽高
            listItem.measure(0, 0);
            // 统计所有子项的总高度
            totalHeight += listItem.getMeasuredHeight();
        }
        ViewGroup.LayoutParams params = listView.getLayoutParams();
        params.height = totalHeight + (listView.getDividerHeight() * (listAdapter.getCount() - 1));
        // listView.getDividerHeight()获取子项间分隔符占用的高度
        // params.height最后得到整个ListView完整显示需要的高度
        listView.setLayoutParams(params);
    }
```
另外，这时，这时最好给ListView之外嵌套一层LinearLayout，不然有时候这种方法会失效，如下：
```
<merge xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.art.demo.ListViewScrollViewTestActivity">
    <ScrollView
        android:id="@+id/scrollview"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent">

            <ListView
                android:id="@+id/listview"
                android:layout_width="fill_parent"
                android:layout_height="match_parent"
                android:background="#FFF4F4F4"
                android:dividerHeight="0.0dip"
                android:fadingEdge="vertical" />
        </LinearLayout>
    </ScrollView>
</merge>
```
###### 3. 方法三：在xml文件中，直接将Listview的高度写死
可以确定的是：这种方式可以改变ListView的高度，但是，还有一个严重的问题就是listview的数据是可变动的，除非你能正确的写出listview的高度，否则这种方式就是个鸡肋。
如下：
```
<ScrollView
        android:id="@+id/scrollview"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent">
            <ListView
                android:id="@+id/listview"
                android:layout_width="fill_parent"
                android:layout_height="300dip"
                android:background="#FFF4F4F4"
                android:dividerHeight="0.0dip"
                android:fadingEdge="vertical" />
        </LinearLayout>
    </ScrollView>
```
###### 4. 方法四：
某些情况下，其实我们可以完全避免ScrollView嵌套Listview，比如使用listview的addHeader() 函数来实现预期效果或者利用布局的特性达到预期效果，当然，具体怎么用，只有在开发中慢慢琢磨，慢慢总结了.

至此，关于“ListView怎么和ScrollView兼容”这个问题就算是回答完了，如果有不明白的地方可以问我，同样，那里有错误也欢迎大家指出，真的不胜感激。

 ------
 
接下来要说的就是！！！！！
#### listview怎么优化？
关于Listview的优化，只要面试过的人，我相信都对这个题很熟悉，不管有没有人问过你这个题，我想你自己也一定准备过，否则，嘿嘿！！！！！而且网上也一搜一大把这里就简单提几个主要的:
* 1、复用convertView，对convetView进行判空，当convertView不为空时重复使用，为空则初始化，从而减少了很多不必要的View的创建、减少findViewById的次数，  
* 2、避免在getView方法中做耗时操作
* 3、采用ViewHolder模式缓存item条目的引用
* 4、给listView设置滚动监听器 根据不同状态 不同处理数据 分批分页加载 根据listView的状态去操作，比如当列表快速滑动时不去开启大量的异步任务去请求图片
* 5、listview每个item层级结构不要太复杂
* 6、listview每个item中异步加载图片，并对图片加载做优化，（关于Listview分页加载和图片异步加载思路请看接下来的文章内容）
* 7、listview每个item中不要创建线程
* 8、尽量能保证 Adapter 的 hasStableIds() 返回 true 这样在 notifyDataSetChanged() 的时候，如果item内容并没有变化，ListView 将不会重新绘制这个 View，达到优化的目的
* 9、在一些场景中，ScollView内会包含多个ListView，可以把listview的高度写死固定下来。 由于ScollView在快速滑动过程中需要大量计算每一个listview的高度，阻塞了UI线程导致卡顿现象出现，如果我们每一个item的高度都是均匀的，可以通过计算把listview的高度确定下来，避免卡顿现象出现
* 10、使用 RecycleView 代替listview： 每个item内容的变动，listview都需要去调用notifyDataSetChanged来更新全部的item，太浪费性能了。RecycleView可以实现当个item的局部刷新，并且引入了增加和删除的动态效果，在性能上和定制上都有很大的改善
* 11、ListView 中元素避免半透明： 半透明绘制需要大量乘法计算，在滑动时不停重绘会造成大量的计算，在比较差的机子上会比较卡。 在设计上能不半透明就不不半透明。实在要弄就把在滑动的时候把半透明设置成不透明，滑动完再重新设置成半透明。

下面就是关于Listview的一些相关拓展
###### 1. 打开套有 ListVew的 ScrollView的页面布局 默认 起始位置不是最顶部？
解决办法有两种：
 方法一：把套在里面的ListVew 不让获取焦点即可。listview.setFocusable(false);注意：在xml布局里面设置android：focusable=“false”不生效
方法二：myScrollView.smoothScrollTo(0,0);

###### 2. 上拉加载和下拉刷新怎么实现？
实现OnScrollListener 接口重写onScrollStateChanged 和onScroll方法，
使用onscroll方法实现”滑动“后处理检查是否还有新的记录，如果有，调用 addFooterView，添加记录到adapter, adapter调notifyDataSetChanged 更新数据;如果没有记录了，把自定义的mFooterView去掉。使用onScrollStateChanged可以检测是否滚到最后一行且停止滚动然后执行加载

###### 3. listview失去焦点怎么处理？
在listview子布局里面写，可以解决焦点失去的问题
android:descendantFocusability="blocksDescendants"

###### 4. ListView图片异步加载实现思路？
1.先从内存缓存中获取图片显示（内存缓冲） 
2.获取不到的话从SD卡里获取（SD卡缓冲，，从SD卡获取图片是放在子线程里执行的，否则快速滑屏的话会不够流畅） 
3.都获取不到的话从网络下载图片并保存到SD卡同时加入内存并显示（视情况看是否要显示）

###### 5. 你知道Listview里有Button就点不动了你知道吗？
原因是button强制获取了item的焦点，只要设置button的focusable为false即可。

###### 6. 如何自定义一个Adapter（有兴趣的可以看一下，大家不呀扔我鸡蛋）
继承自BaseAdapter实现里面的方法，listView在开始绘制的时候，系统首先调用getCount（）函数，根据他的返回值得到listView的长度，然后根据这个长度，调用getView（）逐一绘制每一行。如果你的getCount（）返回值是0的话，列表将不显示同样return 1，就只显示一行。系统显示列表时，首先实例化一个适配器（这里将实例化自定义的适配器）。当手动完成适配时，必 须手动映射数据，这需要重写getView（）方法。系统在绘制列表的每一行的时候将调用此方法。getView()有三个参数，position表示将显示的是第几行，covertView是从布局文件中inflate来的 布局。我们用LayoutInflater的方法将定义好的main.xml文件提取成View实例用来显示。
然后 将xml文件中的各个组件实例化（简单的findViewById()方法）。这样便可以将数据对应到各个组件上了。但是按钮为了响应点击事件，需要为它添加点击监听器，这样就能捕获点击事件。至此一个自定 义的listView就完成了，现在让我们回过头从新审视这个过程。系统要绘制ListView了，
他首先获得 要绘制的这个列表的长度，然后开始绘制第一行，怎么绘制呢？
调用getView()函数。在这个函数里面 首先获得一个View（实际上是一个ViewGroup），然后再实例并设置各个组件，显示之。好了，绘制完这一行了。那 再绘制下一行，直到绘完为止。在实际的运行过程中会发现listView的每一行没有焦点了，这是因为Button抢夺了listView的焦点，只要布局文件中将Button设置为没有焦点就OK了。

###### 7. listview分页加载的步骤？
通常实现分页加载有两种方式，一种是在ListView底部设置一个按钮，用户点击即加载。另一种是当用户滑动到底部时自动加载。
在ListView底部设置一个按钮，用户点击即加载实现思路:
```
        // 加上底部View，注意要放在setAdapter方法前
        ListView.addFooterView(moreView);
        bt.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                pg.setVisibility(View.VISIBLE);// 将进度条可见
                bt.setVisibility(View.GONE);// 按钮不可见
                handler.postDelayed(new Runnable() {
                    @Override
                    public void run() {
                        loadMoreDate();// 加载更多数据
                        bt.setVisibility(View.VISIBLE);
                        pg.setVisibility(View.GONE);
                        mSimpleAdapter.notifyDataSetChanged();// 通知listView刷新数据
                    }
                }, 2000);
            }
        });
```
当用户滑动到底部时自动加载实现思路: 
实现OnScrollListener 接口重写onScrollStateChanged 和onScroll方法，使用onscroll方法实现”滑动“后处理检查是否还有新的记录，如果有,添加记录到adapter, adapter调用 notifyDataSetChanged 更新数据;如果没有记录了，则不再加载数据。使用onScrollStateChanged可以检测是否滚到最后一行且停止滚动然后执行加载.

###### 8. ViewHolder内部类非得要声明成static的呢？
这不是Android的优化，而是Java提倡的优化，
如果声明成员类不要求访问外围实例，就要始终把static修饰符放在它的声明中，使它成为静态成员类，而不是非静态成员类。
因为非静态成员类的实例会包含一个额外的指向外围对象的引用，保存这份引用要消耗时间和空间，并且导致外围类实例符合垃圾回收时仍然被保留。如果没有外围实例的情况下，也需要分配实例，就不能使用非静态成员类，因为非静态成员类的实例必须要有一个外围实例。

###### 9. [Listview每个item有特效进入视图](http://www.jianshu.com/p/6567b6dbb372)
###### 10. [ScrollView、ListView剖析 - 上下拉伸回弹阻尼效果](http://www.jianshu.com/p/834e522d02dc)
###### 11. [自定义控件-下拉刷新和上拉加载的listView](http://www.jianshu.com/p/cf4a77727d68)

###### 12、ListView 如何显示多种类型的Item
ListView 显示的每个条目都是通过 baseAdapter 的 getView(int position, View convertView, ViewGroup parent)来展示的,理 论上我们完全可以让每个条目都是不同类型的 view。 比如:从服务器拿回一个标识为 id=1,那么当 id=1 的时候,我们就加载类 型一的条目,当 id=2 的时候,加载类型二的条目。常见布局在资讯类客户端中 可以经常看到。 此之外 adapter 还提供了 getViewTypeCount()和 getItemViewType(int position)两个方法。在 getView 方法中我们可以根据不 同的 viewtype 加载不同的布局文件。

###### 13、在ListView中设置Selector为null会报空指针？ 
mListView.setSelector(null);//空指针 
试试下面这个： 
mListView.setSelector(new ColorDrawable(Color.TRANSPARENT));

###### 14、ListView图片错位的问题是如何产生的？怎么解决？
图片错位问题的本质源于我们的 listview 使用了缓存 convertView， 假设一种场景， 一个 listview一屏显示九个 item，那么在拉出第十个 item 的时候，事实上该 item 是重复使用了第一个 item，也就是说在第一个 item 从网络中下载图片并最终要显示的时候，其实该 item 已经不在当前显示区域内了，此时显示的后果将可能在第十个 item 上输出图像，这就导致了图片错位的问题。所以解决之道在于可见则显示，不可见则不显示。 
如下：
每次getView能给对象一个标识，在异步加载完成时比较标识与当前行item的标识是否一致，一致则显示，否则不做处理即可， 如下
```
// 给 ImageView 设置一个 tag
holder.img.setTag(imgUrl);
// 预设一个图片
holder.img.setImageResource(R.drawable.ic_launcher);

// 通过 tag 来防止图片错位
if (imageView.getTag() != null &&imageView.getTag().equals(imageUrl)) {
imageView.setImageBitmap(result);
}
```

###### 15、ListView实现Item局部刷新？
```
private void updateView(int itemIndex) {
          //得到第一个可显示控件的位置，
          int visiblePosition = mListView.getFirstVisiblePosition();
          //只有当要更新的view在可见的位置时才更新，不可见时，跳过不更新
          if (itemIndex - visiblePosition >= 0) {
              //得到要更新的item的view
            View view = mListView.getChildAt(itemIndex - visiblePosition);
              //调用adapter更新界面
             mAdapter.updateView(view, itemIndex);
        }
     }
```
###### 16、ListView 中如何优化图片
图片的优化策略比较多。
* [处理图片的方式](http://www.jianshu.com/p/98c88f9ceafa)：
如果 ListView 中自定义的 Item 中有涉及到大量图片的，一定要对图片进行细心的处理，因为图片占的内存是 ListView 项中最头疼的，处理图片的方法大致有以下几种：
	* 不要直接拿路径就去循环 BitmapFactory.decodeFile;使用 Options 保存图片大小、不要加载图片到内存去。
	* 对图片一定要经过边界压缩尤其是比较大的图片，如果你的图片是后台服务器处理好的那就
不需要了
	* 在 ListView 中取图片时也不要直接拿个路径去取图片，而是以 WeakReference（使用WeakReference 代替强引用。比如可以使用 WeakReference mContextRef）、SoftReference、WeakHashMap 等的来存储图片信息。
	* 在 getView 中做图片转换时，产生的中间变量一定及时释放
* 异步加载图片基本思想：   
1）、 先从内存缓存中获取图片显示（内存缓冲）   
2）、获取不到的话从 SD 卡里获取（SD 卡缓冲）    
3）、都获取不到的话从网络下载图片并保存到 SD 卡同时加入内存并显示（视情况看是否要显示）   
原理：   
优化一：先从内存中加载，没有则开启线程从 SD 卡或网络中获取，这里注意从 SD 卡获取图片是放在子线程里执行的，否则快速滑屏的话会不够流畅。   
优化二：于此同时，在 adapter 里有个 busy 变量，表示 listview 是否处于滑动状态，如果是滑动状态则仅从内存中获取图片，没有的话无需再开启线程去外存或网络获取图片。   
优化三：ImageLoader 里的线程使用了线程池，从而避免了过多线程频繁创建和销毁，如果每次总是 new 一个线程去执行这是非常不可取的，好一点的用的 AsyncTask 类，其实内部也是用到了线程池。在从网络获取图片时，先是将其保存到 sd 卡，然后再加载到内存，这么做的好处是在加载到内存时可以做个压缩处理，以减少图片所占内存。

***

更多内容请关注 [ 我的专题 ](http://www.jianshu.com/collection/bcc2c1ba8378)

![](http://upload-images.jianshu.io/upload_images/1479978-0ff1a43230b41689.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

