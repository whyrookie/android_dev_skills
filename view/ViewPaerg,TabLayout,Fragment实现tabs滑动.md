##Demo实现:
　　前段时间公司的项目遇到了使用　**ViewPager**,**TabLayout** 和 **Fragment**实现一个多个tab之间的滑动,这样的效果在大部分的app中都有，因为有了5.0以后的TabLayout控件，实现这样的效果简单多了。下面是demo的效果图：

![](https://github.com/whyrookie/android_dev_skills/blob/master/images/564843735-587990a40a4ab_articlex.gif)

接下来就是代码了，先是Fragenmnt的xml文件：

```xml

<?xml version="1.0" encoding="utf-8"?>
   <LinearLayoutxmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:gravity="center"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <TextView
            android:gravity="center"
            android:id="@+id/tv_content"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:text="hello world"
            />
    </LinearLayout>
```

　为了简单就写了个TextView,然后是Fragment的代码：

```java

public class TestFragment extends Fragment {

       private static final String TAG ="TestFragment";
       private  int mIndex;
       @BindView(R.id.tv_content)
       TextView mContentTv;
       private View mTestView;

       public TestFragment(){}

       @Nullable
       @Override
       public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
           mTestView = inflater.inflate(R.layout.fragment_test, container, false);
           ButterKnife.bind(this, mTestView);
           return mTestView;

       }

       @Override
       public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
           super.onViewCreated(view, savedInstanceState);
           mContentTv.setText("TestFragment" + mIndex);
       }

       @Override
       public void onCreate(@Nullable Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           mIndex = getArguments().getInt("index");//根据不同的额index显示不同的内容

       }

       @Override
       public void onStart() {
           super.onStart();
           Log.d(TAG,"onStart()" + mIndex);
       }

       @Override
       public void onPause() {
           super.onPause();
           Log.d(TAG,"onPause()" + mIndex);
       }

       @Override
       public void onResume() {
           super.onResume();
           Log.d(TAG,"onResume()" + mIndex);
       }
   }

```


那几个生命周期方法是为了待会在Tab之间切换时跟踪一下Fragment的生命周期方法。代码里为了偷懒使用了butterknife插件=_=
　　然后是Adapter的代码：

```java

public class ViewPagerAdapter extends FragmentPagerAdapter {

        private List<String> mTabTitles ;
        private static final int FRAGMENT_COUNT = 6;

        public ViewPagerAdapter(FragmentManager fragmentManager, List<BaseFragment> fragments,List<String> tabTitles) {
            super(fragmentManager);
            mTabTitles = tabTitles;
        }

        @Override
        public Fragment getItem(int position) {
            TestFragment testFragment = new TestFragment();
            Bundle bundle = new Bundle();
            bundle.putInt("index",position);//传入不同的index
            testFragment.setArguments(bundle);
            return testFragment;
        }

        @Override
        public Object instantiateItem(ViewGroup container, int position) {
            return super.instantiateItem(container, position);
        }

        @Override
        public int getCount() {
            return FRAGMENT_COUNT;
        }

        //设置Tab的标题
        @Override
        public CharSequence getPageTitle(int position) {
            return mTabTitles.get(position);
        }
    }
```


　好像也没什么注释的，接下来就是Activity的代码了，把这些组合起来使用:

```xml

<?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:orientation="vertical"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:id="@+id/activity_main"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context="com.example.why.demo.MainActivity">

        <android.support.design.widget.TabLayout
            android:id="@+id/tab_layout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:tabIndicatorColor="@color/colorAccent"
            app:tabIndicatorHeight="2dp"
            app:tabMode="fixed">

        </android.support.design.widget.TabLayout>
        <android.support.v4.view.ViewPager
            android:id="@+id/view_pager"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            >
        </android.support.v4.view.ViewPager>
    </LinearLayout>
```

TabLayout的属性app:tabMode还有一直模式scrollable，可以自行尝试。代码部分:

```java

public class MainActivity extends AppCompatActivity {

       @BindView(R.id.tab_layout)
       TabLayout mTabLayout;
       @BindView(R.id.view_pager)
       ViewPager mViewPager;
       private List<String> mTabTitles = new ArrayList<>();

       private List<BaseFragment> mFragments = new ArrayList<>();
       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.activity_main);
           ButterKnife.bind(this);
           initTab();
           mViewPager.setAdapter(new ViewPagerAdapter(getSupportFragmentManager(),mFragments,mTabTitles));

           mTabLayout.setupWithViewPager(mViewPager);//注意：这行代码不能少,使TabLayout和ViewPager绑定
       }

       //设置Tab的title
       private void initTab() {
           for (int i = 0; i < 6; i++) {
               mTabTitles.add("Tab" + i);
           }
       }
   }

```

这里的代码就只是将ViewPager和TabLayout结合,并设置了Adapter。到这里效果就出来了，当然这里没有对TabLayout的讲解，TabLayout出来也挺久了，除了官网的介绍外，也有很多博客对其进行了讲解,这里也就不介绍了。
##Fragment生命周期跟踪：
　　写到这里效果是有了，但是由于ViewPager的预加载机制，导致我当时想实现几个页面同步更新同步(就是几个tab用相同的内容，比如在tab1删除了一些内容，希望在tab２也发生改变了，由于预加载的原因Fragment主要生命周期的方法不会再调用了所以无法在Fragment的生存周期方法中实现改变)更新内容时遇到了点麻烦。先说下ViewPager预加载，每次加载三个，比如现在是ta2那么ta1和tab3也被加载了(当然也可以取消预加载)，比如我在Acivity通过mViewPager.setCurrentItem(3),demo初始界面:

![](https://github.com/whyrookie/android_dev_skills/blob/master/images/558863520-5879ba4b88cb2_articlex.gif)

日志:

![](https://github.com/whyrookie/android_dev_skills/blob/master/images/1405728091-5879a13d1ae60_articlex.png)

可以看到tab2,tab3,tab4的生命周期都被调用了。如果我们从左往右滑动，日志如下：

![](https://github.com/whyrookie/android_dev_skills/blob/master/images/3255273114-5879a20c39e5b_articlex.png)

现在在tab4,tab2调用了onPause(),tab5被加载了，tab3已经加载过了。如果我们一开始是从右往左滑动，那么日志如下：

![](https://github.com/whyrookie/android_dev_skills/blob/master/images/2638532950-5879a2cb726fd_articlex.png)

现在在tab2,tab4调用了onPause(),tab1被加载了。
##不同Tab的同步更新:
　　回到前面的问题，如何同步更新不同tab的内容?在网上找到了一个方法，已经找不到到那篇csdn博客了，表示感谢，解决了我的问题。
　　那篇博客里的解决方法是给每个fragment打上tag,使用是FragmentPagerAdapter内部的 String makeFragmentName(int viewId, long id)方法，这里我们可以直接复制放在自己定义的ViewPagerAdapter里给fragment打tag,这个动作放在instantiateItem(ViewGroup container, int position)中，然后要同步更新的时候调用FragmentManager.findFragmentByTag()获取对应的Fragment就可以调用相应的方法了，这里是打上tag后的代码:

```java

public class ViewPagerAdapter extends FragmentPagerAdapter {

    private List<String> mTabTitles ;
    private static final int FRAGMENT_COUNT = 6;

    private List<String> mFragmentTags = new ArrayList<>();

    private FragmentManager mFragmentManager;

    public ViewPagerAdapter(FragmentManager fragmentManager, List<BaseFragment> fragments,List<String> tabTitles) {
        super(fragmentManager);
        mTabTitles = tabTitles;
        this.mFragmentManager = fragmentManager;
    }

    @Override
    public Fragment getItem(int position) {
        TestFragment testFragment = new TestFragment();
        Bundle bundle = new Bundle();
        bundle.putInt("index",position);//传入不同的index
        testFragment.setArguments(bundle);
        return testFragment;
    }

    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        mFragmentTags.add(makeFragmentName(container.getId(),position));//设置tag
        return super.instantiateItem(container, position);
    }

    @Override
    public int getCount() {
        return FRAGMENT_COUNT;
    }

    //设置Tab的标题
    @Override
    public CharSequence getPageTitle(int position) {
        return mTabTitles.get(position);
    }

    //这个方法是FragmentPagerAdapter内部方法，其内部就是根据这个给fragment打tag的，我们直接使用可以获取正确的tag。
    private static String makeFragmentName(int viewId, long id) {
        return "android:switcher:" + viewId + ":" + id;
    }
    //这个方法可以在Fragment的代码中调用
    public void synchronizedFragment() {
        for (int i = 0; i < mFragmentTags.size();i++) {
            TestFragment testFragment = (TestFragment) mFragmentManager.findFragmentByTag(mFragmentTags.get(i));

            //接下来就可以调用testFragment相应的更新方法了
            testFragment.update();
        }

    }
}
```
就这些了，有些内容参考过别人的文章就不一一贴出来了（很久以前看的），很多内容网上都有，这里只是做个总结。
