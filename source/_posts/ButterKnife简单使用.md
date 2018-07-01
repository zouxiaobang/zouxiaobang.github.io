---
title: ButterKnife简单使用
date: 2016-09-14 21:50:06
tags: [android,注解]
categories: Android
---
## 注入 ButterKnife
现在的ButterKnife已经升级到8，所以我还是介绍v8吧，虽然以前一直使用的是v6

### ButterKnife的基本使用
注意ButterKnife的v7以上版本的注册都是使用bind()
```
ButterKnife.bind(this);
```
记得bind操作必须在setContentView之后。
并在onDestroy()方法中进行解绑
```
super.onDestroy();
ButterKnife.unbind(this);
```


#### 1、代替findViewById()
``` bash
TextView mTvTest = (TextView)findViewById(R.id.tv_test);
==>
@Bind(R.id.tv_test)
TextView mTvTest;
```
使用@Bind()时的对象必须至少是包权限的，即不能使用private。当然也不能是静态static的。

#### 2、代替ViewHolder中的findViewById
你会不会被列表的ViewHolder中的各种组件初始化烦到爆炸，反正我会。而ButterKnife正好解决了这个问题
``` bash
	@Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder;
        if (convertView == null) {
            convertView = mInflater.inflate(R.layout.person_item_layout, null);
            holder = new ViewHolder(convertView);
            convertView.setTag(holder);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
        Person person = getItem(position);
        if (null != person) {
            holder.name.setText(person.getName());
            holder.age.setText(String.valueOf(person.getAge()));
            holder.location.setText(person.getLocation());
            holder.work.setText(person.getWork());
        }
 
        return convertView;
    }
 
    static class ViewHolder {
        @Bind(R.id.person_name)
        TextView name;
        @Bind(R.id.person_age)
        TextView age;
        @Bind(R.id.person_location)
        TextView location;
        @Bind(R.id.person_work)
        TextView work;
 
        public ViewHolder(View view) {
            ButterKnife.inject(this, view);
        }
    }
```
这里的代码就有点区别了，哪里的区别呢，给一份原来的代码就可以很清楚的看出来了
``` bash
	@Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder;
        if (convertView == null) {
            holder = new ViewHolder();
            convertView = mInflater.inflate(R.layout.person_item_layout, null);
            holder.name = (TextView)findViewById(R.id.person_name);
            holder.age = (TextView)findViewById(R.id.person_age);
            holder.location = (TextView)findViewById(R.id.person_location);
            holder.work = (TextView)findViewById(R.id.person_work);
            convertView.setTag(holder);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
        Person person = getItem(position);
        if (null != person) {
            holder.name.setText(person.getName());
            holder.age.setText(String.valueOf(person.getAge()));
            holder.location.setText(person.getLocation());
            holder.work.setText(person.getWork());
        }
 
        return convertView;
    }
 
    static class ViewHolder {
        TextView name;
        TextView age;
        TextView location;
        TextView work;
    }
```
少了中间那一大堆的findViewById，是不是很爽

#### 3、代替setOnClickListener()方法
点击事件，要么在类实现View.OnClickListener，要么直接使用组件的setOnCLickListener()方法
但这都太麻烦，看起来代码也很臃肿不好看，所以看看这个
``` bash
@OnClick(R.id.btn_send)
void send(){
	//TODO
}
```
而参数，是重写onClicked()方法中的所有参数，这是可选的，例如
``` bash
@OnClick(R.id.btn_send)
void send(View view){
	//TODO
}
```

#### 4、资源的绑定
这个功能就很强大了，平常我们都需要使用
```
	getResources().getDrawable(R.drawable.a);
```
之类的代码去获取，如果忘记这段代码呢？没关系，使用ButterKnife
```
@BindString(R.string.name)
String name;
@BindColor(R.color.green)
int color;
@BindDrawable(R.drawable.pic)
Drawable picture;
```

#### 5、对注入的快捷键
(只提供mac下的)
在该布局id上 ==> command+N ==> Generate ==> Generate ButterKnife Injections ==> 选择要注入的组件
当然，这个就需要androidstudio安装插件了
搜索Android Butter Zelezny，安装 ==> 重启


