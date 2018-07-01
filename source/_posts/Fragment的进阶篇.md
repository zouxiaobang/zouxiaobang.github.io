---
title: Fragment的进阶篇
date: 2016-09-19 22:52:02
tags: [android, Fragment]
categories: Android
---
## Fragment的回退
我们都知道，Activity有一个栈，叫Task栈，只要我们没有finish掉该Activity，点击Back就可以返回到启动它的Activity中去。
Fragment也有一个类似的栈，叫回退栈。但是它与Task栈不同的是它必须手动将fragment添加到回退栈中
``` 
mTransaction.addToBackStack(null);
```
![Fragment的回退](/img/Fragment/fragmentba.png)
栈的结构和操作我就不必多说了，当我们在最后一个fragment中点击back，就会返回与之关联的Activity。

现在我们来看一个例子
![第一个fragment](/img/Fragment/back1.png)
 点击进入第二个fragment
![第二个fragment](/img/Fragment/back2.png)
点击进入第三个fragment
![第三个fragment](/img/Fragment/back3.png) 
点击back 
![第二个fragment](/img/Fragment/back2.png) 
再点击back 
![第一个fragment](/img/Fragment/back1.png)

Activity中的代码
``` bash
public class MainActivity extends Activity {

    @Bind(R.id.fl_content)
    FrameLayout mFlContent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_back_stack);
        ButterKnife.bind(this);

        FragmentManager manager = getFragmentManager();
        FragmentTransaction transaction = manager.beginTransaction();
        transaction.add(R.id.fl_content, new BackOneFragment());
        //将fragment添加到回退栈中
//        transaction.addToBackStack(null);
        transaction.commit();
    }
}
```
其中，如果我们将第一个fragment添加到回退栈，那么在第一个fragment中点击back，会先回到该Activity中(如上面代码，如果FrameLayout中没有任何东西，则点击back会返回一张白布)，而如果不加入回退栈，那么点击back会直接退出该Activity。

fragment的xml代码就不贴出来了，里面就包含了一个EditText和一个Button
下面是第一个fragment
``` bash
public class BackOneFragment extends Fragment {
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_back_one, container, false);
        Button button = (Button) view.findViewById(R.id.btn_2second);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                FragmentManager manager = getFragmentManager();
                FragmentTransaction transaction = manager.beginTransaction();
                transaction.add(R.id.fl_content, new BackTwoFragment());
//                transaction.replace(R.id.fl_content, new BackTwoFragment());
                transaction.addToBackStack(null);
                transaction.commit();
            }
        });

        return view;
    }
}
```
使用FragmentTransaction调用add()和replace()的区别在哪里呢？
它们的区别就在于从第二个fragment回退到第一个fragment时是否将视图重绘：
![调用add()方法](/img/Fragment/back1.png)
![调用replace()](/img/Fragment/back1.png)
replace()方法的内部是先调用remove()方法，再调用add()方法，所以会先将视图销毁，再重新绘制，即调用了onDestroyView()和onCreateView();
所以尽管已经将事务保存进了回退栈，而事务也并没有被销毁，但视图还是会进行重绘。

而第二个fragment的代码和第一个大同小异，就不再贴出，如果有兴趣可以去看看我的工程代码。
[Fragment示例](https://github.com/zouxiaobang/FragmentProject)

## Fragment的通信
Fragment的通信有两种：
* Fragment与Activity的通信
* Fragment之间的通信
其中，Fragment与Fragment之间的通信一般并不提倡，而应该是由Activity来判断和显示哪一个Fragment。所以这里并不介绍第二种通信。

Fragment通过getActivity()来获取与之绑定的Activity的Context对象，而Activity可以通过Fragment实例或者findFragmentById()或findFragmentByTab()来获取Fragment对象。但它们之间一般使用interface接口来实现通信。
例子：
![调用add()方法](/img/Fragment/connect1.png)
该界面白色部分为Activity中的组件，而绿色的为Fragment中的组件，在Fragment中的EditText中输入内容后点击显示
![调用add()方法](/img/Fragment/connect2.png)
我们可以看到Activity中的TextView跟着改变，这就是在Activity中获取来着fragment中的信息。代码如下：
``` bash
/**
* Fragment的代码
*/
public class ConnectFragment extends Fragment {
    @Bind(R.id.et_message)
    EditText mEtMessage;
    @Bind(R.id.btn_show)
    Button mBtnShow;

    private OnGetMessageListener mListener;

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_connect, container, false);
        ButterKnife.bind(this, view);

        return view;
    }

    @OnClick(R.id.btn_show)
    public void onClick() {
        String msg = mEtMessage.getText().toString();
        if (!TextUtils.isEmpty(msg) && mListener != null){
            mListener.getMessage(msg);
        }
    }

    public interface OnGetMessageListener{
        void getMessage(String msg);
    }

    public void setOnGetMessageListener(OnGetMessageListener listener){
        mListener = listener;
    }


    @Override
    public void onDestroyView() {
        super.onDestroyView();
        ButterKnife.unbind(this);
    }
}


/**
* Activity的代码
*/
public class ConnectActivity extends Activity {

    private static final int MSG_GET_MSG = 0;
    @Bind(R.id.tv_change)
    TextView mTvChange;
    @Bind(R.id.fl_content)
    FrameLayout mFlContent;

    private Handler mHandler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what){
                case MSG_GET_MSG:
                    mTvChange.setText(msg.getData().getString("data"));
                    break;
            }
        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_connect);
        ButterKnife.bind(this);

        FragmentManager manager = getFragmentManager();
        FragmentTransaction transaction = manager.beginTransaction();
        ConnectFragment fragment = new ConnectFragment();
        transaction.add(R.id.fl_content, fragment);
        transaction.commit();

        fragment.setOnGetMessageListener(new ConnectFragment.OnGetMessageListener() {
            @Override
            public void getMessage(String msg) {
                Message message = new Message();
                Bundle data = new Bundle();
                data.putString("data", msg);
                message.setData(data);
                mHandler.sendMessage(message);
            }
        });
    }
}
```
在fragment类中创建一个interface接口通过该接口来发送信息，而在Activity中通过该接口获取来自fragment的信息

那么Activity怎么发送数据给Fragment呢？
在fragment类中添加public方法
``` bash
public void setTvChange(String change){
    mTvChange.setText(change);
}
```
而在Activity中通过引用来调用该方法
```
if (mFragment == null)
    mFragment = new ConnectFragment();

mFragment.setTvChange("activity已经点击了按钮");
```

[Fragment示例](https://github.com/zouxiaobang/FragmentProject)

