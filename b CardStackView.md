最近小猿在改进之前写过的学校教务系统APP的UI界面的时候，发现了一个有趣的Android UI——卡片式折叠交互CardStackView，该View是在GitHub上找到的，但是该View的主人没有告诉我如何使用，小猿研究了半天，在此，将其简单的使用步骤阐述一下：

**CardViewStack的GitHub地址**：https://github.com/loopeer/CardStackView

> 先上个**源博主**效果图：

![这里写图片描述](http://img.blog.csdn.net/20170717112218406?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2h1X2xhbmNl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)![这里写图片描述](http://img.blog.csdn.net/20170717112250561?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2h1X2xhbmNl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

> 使用步骤：

**1.在Android studio中dependencies里添加依赖**

```
dependencies {
    compile 'com.loopeer.library:cardstack:1.0.2'
}
```
**2.自定义一个单个卡片的item：**

```
LinearLayout
    android:id="@+id/linear_list_card_item"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    app:stackHeaderHeight="100dp">

    <FrameLayout
        android:id="@+id/frame_list_card_item"
        android:layout_width="match_parent"
        android:layout_height="80dp"
        android:background="@drawable/course_item_1">

        <TextView
            android:id="@+id/text_list_card_title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_margin="20dp"
            android:textColor="#fff"
            android:textSize="24sp"
            android:textStyle="bold"
            tools:text="12"/>
    </FrameLayout>

    <LinearLayout
        android:id="@+id/container_list_content"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:animateLayoutChanges="true">

        <android.support.v7.widget.RecyclerView
            android:id="@+id/recycler"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

        </android.support.v7.widget.RecyclerView>
    </LinearLayout>

</LinearLayout>
```

**3.在主布局里使用CardStackView：**

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#f0f0f0">

    <com.loopeer.cardstack.CardStackView
        android:id="@+id/stackview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:clipToPadding="false"
        android:padding="6dp">
        
    </com.loopeer.cardstack.CardStackView>
</LinearLayout>
```
**4.写一个TestStackAdapter（GitHub博主上给出了个Adapter的demo，里面有卡片的三种样式：**

 1. ColorItemViewHolder ：显示卡片正常样式 
 2. ColorItemLargeHeaderViewHolder ：卡片的头比正常显示的大 
 3. ColorItemWithNoHeaderViewHolder ：不显示卡片的头，只显示内容）
 
**TestStackAdapter代码有点长，但是理解起来不难，因为它非常像RecyclerView.Adapter<>，因为它是继承自CardStackView.Adapter<>,我将部分主要代码讲一下：**

1.onCreateView（）：加载卡片的item布局（三种样式可以加载）：

```
@Override
    protected CardStackView.ViewHolder onCreateView(ViewGroup parent, int viewType) {
        View view;
        switch (viewType) {
            case R.layout.list_card_item_larger_header:
                view = getLayoutInflater().inflate(R.layout.list_card_item_larger_header, parent, false);
                return new ColorItemLargeHeaderViewHolder(view);
            case R.layout.list_card_item_with_no_header:
                view = getLayoutInflater().inflate(R.layout.list_card_item_with_no_header, parent, false);
                return new ColorItemWithNoHeaderViewHolder(view);
            default:
                view = getLayoutInflater().inflate(R.layout.list_card_item, parent, false);
                return new ColorItemViewHolder(view);
        }
    }
```
2.getItemViewType（）：返回每个item的布局样式，在这个Adapter里，都返回了统一的样式：

```
@Override
    public int getItemViewType(int position) {
        if (position == 6) {//TODO TEST LARGER ITEM
            return R.layout.list_card_item_larger_header;
        } else if (position == 10) {
            return R.layout.list_card_item_with_no_header;
        }else {
            return R.layout.list_card_item;
        }
    }
```
3.onItemExpand(boolean b)：在这里判断卡片是否被点击，true就将卡片展开：

```
 @Override
        public void onItemExpand(boolean b) {
            mContainerContent.setVisibility(b ? View.VISIBLE : View.GONE);
        }
```
4.onBind(Integer data, int position)：根据item的position来加载卡片内的内容：

```
public void onBind(Integer data, int position) {
            mLayout.getBackground().setColorFilter(ContextCompat.getColor(getContext(), data), PorterDuff.Mode.SRC_IN);
            mTextTitle.setText(String.valueOf(position));
        }
```
5.bindView(Integer data, int position, CardStackView.ViewHolder holder)：调用onBind（）来加载布局：

```
@Override
    public void bindView(Integer data, int position, CardStackView.ViewHolder holder) {
        if (holder instanceof ColorItemLargeHeaderViewHolder) {
            ColorItemLargeHeaderViewHolder h = (ColorItemLargeHeaderViewHolder) holder;
            h.onBind(data, position);
        }
        if (holder instanceof ColorItemWithNoHeaderViewHolder) {
            ColorItemWithNoHeaderViewHolder h = (ColorItemWithNoHeaderViewHolder) holder;
            h.onBind(data, position);
        }
        if (holder instanceof ColorItemViewHolder) {
            ColorItemViewHolder h = (ColorItemViewHolder) holder;
            h.onBind(data, position);
        }
    }
```
**5.在MainActivity中将CardStackView初始化，因为我是在Fragment中写的，在此，附上我的Fragment中的代码：**

```
public class ProjectFragment extends Fragment implements CardStackView.ItemExpendListener{

    private CardStackView cardStackView;
    private TestStackAdapter testStackAdapter;

    private static Integer[] item = new Integer[]{R.color.team1,R.color.team2,R.color.team3,
            R.color.team4,R.color.team5,R.color.team6,R.color.team7,R.color.team8};

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.project_fragment,container,false);
        initWight(view);
        return view;
    }

    private void initWight(View view) {
        cardStackView = (CardStackView) view.findViewById(R.id.stackview);
        testStackAdapter = new TestStackAdapter(getContext());
        cardStackView.setAdapter(testStackAdapter);
        cardStackView.setItemExpendListener(this);

        handler.postDelayed(new Runnable() {
            @Override
            public void run() {
                testStackAdapter.updateData(Arrays.asList(item));
            }
        },200);
    }

    @Override
    public void onItemExpend(boolean expend) {

    }
}
```
1.先初始化一组颜色的数组，因为TestStackAdapter中根据传入的颜色数组来将卡片的片头附上颜色：

```
private static Integer[] item = new Integer[]{R.color.team1,R.color.team2,R.color.team3,
            R.color.team4,R.color.team5,R.color.team6,R.color.team7,R.color.team8};
```
2.利用handler来进行延时更新卡片内的内容，先将CardStackView实例化，在利用.updateData()来进行CardStackView的内容更新：

```
handler.postDelayed(new Runnable() {
            @Override
            public void run() {
                testStackAdapter.updateData(Arrays.asList(item));
            }
        },200);
```
3.CardStackView的item展开监听事件（通过implement CardStackView.ItemExpendListener）：

```
cardStackView.setItemExpendListener(this);
```

```
@Override
    public void onItemExpend(boolean expend) {

    }
```
CardStackView的简单使用就到此了，如果对此有兴趣的话，可以自行研究一下源码，也可自己写个CardStackAdapter，因为源代码的TestStackAdapter是继承自CardStackView.Adapter<>的

这是小猿写的效果图：
![这里写图片描述](http://img.blog.csdn.net/20170717112743564?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2h1X2xhbmNl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) ![这里写图片描述](http://img.blog.csdn.net/20170717113049310?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2h1X2xhbmNl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

PS：如果小猿写的文章有些不妥之处的话，欢迎指出需要更改的地方O(∩_∩)O~~