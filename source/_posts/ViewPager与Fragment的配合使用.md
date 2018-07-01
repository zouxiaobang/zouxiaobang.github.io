---
title: ViewPager与Fragment的配合使用
date: 2016-09-20 22:10:51
tags: [android, ViewPager, Fragment]
categories: Android
---
## ViewPager
很多时候我们都需要使用到一种界面，就是微信那种可以左移右移的界面，如果通过Scroller机制去自定义这么一个组件，那也不是不可以，就是特别麻烦，特别是各种滑动冲突，点击事件的截取，都能让你头两个大。幸好，Android提供了这么个组件来解决这个难题。
ViewPager的官方文档第一句话就说了ViewPager的左右，就是移来移去。
![官方文档](/img/viewPager/viewpager1.png)
第一段告诉我们这个组件的作用，当然使用它还得实现一个Adapter来控制ViewPager的展示。
那么怎么使用比较好呢，下面就说了，通常是跟Fragment配合使用，那么Adapter当然就是使用与Fragment相关的最好了，官方介绍了两个接口，FragmentPagerAdapter和FramentStatePagerAdapter。我们这里以FragmentPagerAdapter举例子。

先给一个项目图片
![效果图1](/img/viewPager/viewpager2.png)
![效果图2](/img/viewPager/viewpager3.png)

特色：
* 定制性强，修改ViewPager中内容和标题都不需要改动Adapter类
* 标题可随手指移动随时移动。

[项目代码](https://github.com/zouxiaobang/ViewPagerEx)

## 代码解析
### 1、main的xml文件
``` bash
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <include
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        layout="@layout/tab_top"/>

    <android.support.v4.view.ViewPager
        android:id="@+id/vp_content"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
```
将ViewPager作为一个组件添加进去
其中的tab_top.xml文件就不给出了

### 2、Fragment类
总共4个Fragment类，这里只举一个
``` bash
public class ChatFragment extends Fragment{
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_chat, container, false);


        return view;
    }
}
```

### 3、FragmentAdapter类
``` bash
public class FragmentAdapter extends FragmentPagerAdapter implements ViewPager.OnPageChangeListener {
    private List<Fragment> mFragments;
    private ViewPager mPager;
    private OnScrolledListener mListener;
    private int currentIndex;

    public FragmentAdapter(FragmentActivity activity, ViewPager pager) {
        super(activity.getSupportFragmentManager());

        mFragments = ((MainActivity)activity).getFragments();
        mPager = pager;
        mPager.setAdapter(this);
        mPager.setOnPageChangeListener(this);


        DisplayMetrics displayMetrics = new DisplayMetrics();
        activity.getWindowManager().getDefaultDisplay().getMetrics(displayMetrics);
    }

    @Override
    public Fragment getItem(int position) {
        return mFragments.get(position);
    }

    @Override
    public int getCount() {
        return mFragments.size();
    }

    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {

        if (currentIndex == position){
            if (mListener != null){
                mListener.toRight(currentIndex, positionOffset);
            }
        } else if(currentIndex > position){
            if (mListener != null){
                mListener.toLeft(currentIndex, positionOffset);
            }
        }
    }

    @Override
    public void onPageSelected(int position) {
        currentIndex = position;
    }

    @Override
    public void onPageScrollStateChanged(int state) {

    }


    interface OnScrolledListener{
        void toRight(int current, float offset);
        void toLeft(int current, float offset);
    }

    public void setOnScrolledListener(OnScrolledListener listener){
        mListener = listener;
    }
}
```
该Adapter继承了FragmentPagerAdapter，且实现了OnPageChangeListener来解决滑动事件，为了简化滑动事件，该Adapter重新提供了一个接口给外面的Activity调用。

### 4、Activity类
``` bash
public class MainActivity extends FragmentActivity {

    @Bind(R.id.tv_chat)
    TextView mTvChat;
    @Bind(R.id.tv_friend)
    TextView mTvFriend;
    @Bind(R.id.tv_say)
    TextView mTvSay;
    @Bind(R.id.tv_mine)
    TextView mTvMine;
    @Bind(R.id.vp_content)
    ViewPager mVpContent;
    @Bind(R.id.iv_tab_bottom)
    ImageView mIvTabBottom;

    private ChatFragment mChatFragment;
    private FriendFragment mFriendFragment;
    private SayFragment mSayFragment;
    private MineFragment mMineFragment;
    private int mWidth;
    private List<Fragment> mFragments = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);

        init();
    }

    private void init() {
        mChatFragment = new ChatFragment();
        mFriendFragment = new FriendFragment();
        mSayFragment = new SayFragment();
        mMineFragment = new MineFragment();
        mFragments.add(mChatFragment);
        mFragments.add(mFriendFragment);
        mFragments.add(mSayFragment);
        mFragments.add(mMineFragment);

        DisplayMetrics displayMetrics = new DisplayMetrics();
        getWindowManager().getDefaultDisplay().getMetrics(displayMetrics);
        mWidth = displayMetrics.widthPixels;
        final LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams) mIvTabBottom
                .getLayoutParams();
        lp.width = mWidth / 4;

        FragmentAdapter adapter = new FragmentAdapter(this, mVpContent);
        adapter.setOnScrolledListener(new FragmentAdapter.OnScrolledListener() {
            @Override
            public void toRight(int current, float offset) {
                lp.leftMargin = (int) (offset
                        * (mWidth / 4) + current
                        * (mWidth / 4));
                mIvTabBottom.setLayoutParams(lp);
            }

            @Override
            public void toLeft(int current, float offset) {
                lp.leftMargin = (int) (-(1 - offset)
                        * (mWidth / 4) + current
                        * (mWidth / 4));
                mIvTabBottom.setLayoutParams(lp);
            }
        });

        mIvTabBottom.setLayoutParams(lp);
    }

    public List<Fragment> getFragments() {
        return mFragments;
    }

    @OnClick({R.id.tv_chat, R.id.tv_friend, R.id.tv_say, R.id.tv_mine})
    public void onClick(View view) {
        int index = 0;
        switch (view.getId()) {
            case R.id.tv_chat:
                index = 0;
                break;
            case R.id.tv_friend:
                index = 1;
                break;
            case R.id.tv_say:
                index = 2;
                break;
            case R.id.tv_mine:
                index = 3;
                break;
        }
        mVpContent.setCurrentItem(index);
        LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams) mIvTabBottom.getLayoutParams();
        lp.leftMargin = index * (mWidth/4);
        mIvTabBottom.setLayoutParams(lp);
    }
}
```

其中需要注意的是，与ViewPager一起使用的Fragment也需要时V4包
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentActivity;
import android.support.v4.app.FragmentPagerAdapter;
import android.support.v4.view.ViewPager;

如果需要添加页面，只需要修改init()方法里的Fragment就可以。






