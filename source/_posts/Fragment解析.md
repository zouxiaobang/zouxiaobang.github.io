---
title: Fragment解析
date: 2016-09-19 09:55:25
tags: [android,Fragment]
categories: Android
---
很久前就想写Fragment和ViewPager配合使用的博客了，但总是忘了，现在想起来，但觉得还是要先写写Fragment，再写与ViewPager的搭配。
## Fragment的介绍
### 1、Fragment的特点
在面对不同大小的手机，我们总是很苦恼，总是一显示，就乱七八糟，除非你copy了一份代码，然后又重写改了一下尺寸。
这就是一个特别麻烦的问题，所以Android出现了Fragment这个组件来解决这个问题。
Fragment与Activity的关系总是微妙的。Fragment类似于一个ViewGroup，一个Activity可以包含几个Fragment，可以对Fragment进行增删。

### 2、Fragment的生命周期
先看两张图(来自官网)
![Fragment的生命周期](/img/Fragment/life1.png)	![Fragment与Activity的对比](/img/Fragment/life2.png)

相比之下，Fragment所回调的方法更多，它的生命周期被分割得更细。除去与Actiity相同的方法，我们看看这些方法是什么作用：
* onAttach(Activity)：与Activity进行关联的时候回调
* onCreateView(LayoutInflater，ViewGroup，Bundle)：开始创建视图的时候回调
* onActivityCreate(Bundle)：与之关联的Activity调用onCreate的时候回调
----------------------
* DestroyView()：当该Fragment的视图被移除时回调
* onDetach()：与Activity的关联被取消时回调

其中，我们一般重写的onCreateView()方法，而除了该方法，其他方法重写时都需要调用父类该方法，即super.on***()

[Fragment示例](https://github.com/zouxiaobang/FragmentProject)

## Fragment的使用
Fragment的使用都需要以下两步：
* 该Fragment的layout的xml文件
* 继承自Fragment的具体类(继承的是android.app.Fragment)

### 1、静态调用Fragment
![效果图](/img/Fragment/fragmentdisplay.png)
其中分为两个Fragment，一个为TitleFragment，一个为ContentFragment。
很简单，先写出两个Fragment的布局:
``` bash
<!-- fragment_title.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:background="#00796b">
    <Button
        android:id="@+id/btn_left_menu"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="==="/>
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:text="Fragment的静态演示"
        android:textColor="#ffffff"/>
</LinearLayout>

<!-- fragment_content.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center">
    <TextView
        android:id="@+id/tv_content"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Fragment"/>
</LinearLayout>
```
ok，写完两个布局之后再来写它们的继承类
``` bash
//标题的Fragment
public class TitleFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_title, container, false);
        Button btnLeftMenu = (Button) view.findViewById(R.id.btn_left_menu);
        btnLeftMenu.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                //TODO what are you want to do
                Toast.makeText(getActivity(), "to do what you want to do ", Toast.LENGTH_SHORT).show();
            }
        });

        return view;
    }
}

//内容的Fragment
public class ContentFragment extends Fragment {
    private TextView mTvContent;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_content, container, false);
        mTvContent = (TextView) view.findViewById(R.id.tv_content);

        return view;
    }
}
```
这样，就算创建好了Fragment，剩下的只是引用了。我们在Activity中来使用它们，即在Activity的布局文件中调用它们
``` bash
<!-- activity_main.xml -->
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <fragment
        android:id="@+id/fragment_title"
        android:name="com.example.zouxiaobang.fragmentproject.fragment.TitleFragment"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        tools:layout="@layout/fragment_title"/>
    <fragment
        android:id="@+id/fragment_content"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:name="com.example.zouxiaobang.fragmentproject.fragment.ContentFragment"
        tools:layout="@layout/fragment_content"/>
</LinearLayout>
```
其中的tools:layout="@layout/fragment_title"可以不用，这个只是为了在编写代码的时候可以提前预览。
而Activity类中不需要进行任何的修改就可以使用了。

从上面的代码中可以看到，Activity类十分简洁，因为所有的逻辑判断都可以在Fragment中完成。如果你觉得不同的Fragment之间的控件进行通信是个问题，那么你可以interface来进行通信或使用Otto框架，这个就不在这里进行详讲了。
这是静态的调用方法，那么动态调用呢？

### 2、动态调用Fragment
很多app都有这么一个设计，在底部有一行按钮，当我们点击哪一个按钮的时候，就进入一个界面。这个就使用到了Fragment，而且是动态改变。
在学习动态加载时，需要先了解动态Fragment的两个类
* FragmentManager
* FragmentTransaction:Fragment的处理事务
```
FragmentManager fm = getFragmentManager();
FragmentTransaction transaction = fm.beginTransaction(); 
```
对Fragment进行增删改都在Transaction中进行处理。
FragmentTransaction的常用方法
* transaction.add(int, Fragment)：将Fragment添加到Activity中
* transaction.remove(Fragment)：将Fragment从Activity中移除
* transaction.replace(int, Fragment)：使用Fragment替换int所代表的ViewGroup
* transaction.hide(Fragment)：将该Fragment隐藏
* transaction.show(Fragment)：显示该Fragment
* transaction.commit()：提交事务

其中最主要的是replace()和commit()方法。

现在来看上面的那个例子：
![效果图](/img/Fragment/fragmentdy.png)
``` bash
<?xml version="1.0" encoding="utf-8"?>
<!-- activity_main.xml -->
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/activity_dynamic"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <include
        android:id="@+id/buttons"
        layout="@layout/buttons"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"/>
    <FrameLayout
        android:id="@+id/fl_content"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_above="@id/buttons"/>
</RelativeLayout>
```
include中包含了3个按钮，这个我就不给出来了。
FrameLayout是一个ViewGroup，用于存放或替代fragment。
接下来看看Activity中的代码
``` bash
public class MainActivity extends Activity {

    @Bind(R.id.btn_chat)
    Button mBtnChat;
    @Bind(R.id.btn_friend)
    Button mBtnFriend;
    @Bind(R.id.btn_mine)
    Button mBtnMine;
    @Bind(R.id.fl_content)
    FrameLayout mFlContent;

    private ChatFragment mChatFragment;
    private FriendFragment mFriendFragment;
    private MineFragment mMineFragment;
    private FragmentTransaction mTransaction;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);

        //设置默认的页面
        setDefaultFragment();
    }

    private void setDefaultFragment() {
        mChatFragment = new ChatFragment();

        FragmentManager manager = getFragmentManager();
        mTransaction = manager.beginTransaction();
        mTransaction.replace(R.id.fl_content, mChatFragment);

        mTransaction.commit();
    }

    @OnClick({R.id.btn_chat, R.id.btn_friend, R.id.btn_mine})
    public void onClick(View view) {
        FragmentManager manager = getFragmentManager();
        mTransaction = manager.beginTransaction();
        switch (view.getId()) {
            case R.id.btn_chat:
                if (mChatFragment == null)
                    mChatFragment = new ChatFragment();
                mTransaction.replace(R.id.fl_content, mChatFragment);
                break;
            case R.id.btn_friend:
                if (mFriendFragment == null)
                    mFriendFragment = new FriendFragment();
                mTransaction.replace(R.id.fl_content, mFriendFragment);
                break;
            case R.id.btn_mine:
                if (mMineFragment == null)
                    mMineFragment = new MineFragment();
                mTransaction.replace(R.id.fl_content, mMineFragment);
                break;
        }
        mTransaction.commit();
    }
}
```
其中有几点是需要注意的
1 给出一个默认的fragment
2 在FragmentTransaction调用commit()提交了事务之后，该事务就会消失，所以要再次使用它，则需要重新开启。

这是Fragment的基本用法，而Fragment远远不止如此，如果要运用好Fragment，还需要继续看下去
[Fragment的进阶](https://zouxiaobang.github.io/2016/09/19/Fragment的进阶篇/)











