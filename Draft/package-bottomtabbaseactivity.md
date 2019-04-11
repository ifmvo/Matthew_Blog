---
title: Android 快速开发（二），封装一个 BottomTabBaseActivity
date: 2017-05-03 14:41:08
tags: Android
---

## 前言
上一篇我们详细介绍了 [Android 快速开发（一），封装一个 TopBarBaseActivity](/2017/04/14/package-topbarbaseactivity-by-toolbar/) 效果是显而易见的。接下来我们进行 Android 快速开发（二）。


## 效果

什么是快速开发嘞，看这个效果



<center>
![](/img/bottom_tab/bottomtabview1.gif)    ![](/img/bottom_tab/bottomtabview2.gif)
</center>


然而实现 BottomTab + ViewPager 的效果我只用了这么几行代码：


```
public class TestBottomTabBaseActivity extends BottomTabBaseActivity {

    @Override
    protected List<BottomTabView.TabItemView> getTabViews() {
        List<BottomTabView.TabItemView> tabItemViews = new ArrayList<>();
        tabItemViews.add(new BottomTabView.TabItemView(this, "标题1", R.color.colorPrimary,
                R.color.colorAccent, R.mipmap.ic_launcher, R.mipmap.ic_launcher_round));
        tabItemViews.add(new BottomTabView.TabItemView(this, "标题2", R.color.colorPrimary,
                R.color.colorAccent, R.mipmap.ic_launcher, R.mipmap.ic_launcher_round));
        tabItemViews.add(new BottomTabView.TabItemView(this, "标题3", R.color.colorPrimary,
                R.color.colorAccent, R.mipmap.ic_launcher, R.mipmap.ic_launcher_round));
        tabItemViews.add(new BottomTabView.TabItemView(this, "标题4", R.color.colorPrimary,
                R.color.colorAccent, R.mipmap.ic_launcher, R.mipmap.ic_launcher_round));
        return tabItemViews;
    }

    @Override
    protected List<Fragment> getFragments() {
        List<Fragment> fragments = new ArrayList<>();
        fragments.add(new TabFragment1());
        fragments.add(new TabFragment2());
        fragments.add(new TabFragment3());
        fragments.add(new TabFragment4());
        return fragments;
    }

}
```

添加 CenterView 也很简单，只需重写 getCenterView 方法并返回你想添加的 View。

```
@Override
    protected View getCenterView() {
        ImageView centerView = new ImageView(this);
        centerView.setImageResource(R.mipmap.ic_launcher_round);

        centerView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //Toast 一个 "centerView 点击了";
            }
        });
        return centerView;
    }
```

为什么会这么简单呢，因为我把重复性的代码都封装在 `父类` BottomTabBaseActivity 里面，这样的话，我们想实现 BottomTab + ViewPager 的效果只需实现三个方法，这样就可以大大提高我们的效率。爽YY~~~ 接下来我们一起实现这个 BottomTabBaseActivity。

## BottomTabView

首先我介绍一下我自定义一个 [BottomTabView](https://github.com/ifmvo/BottomTabView)，用于实现 Tab 效果

<center>
![](/img/bottom_tab/bottomtabview4.gif)
</center>

- 它提供两个 setTabItemViews 方法，用于初始化 TabItem 的数据和 CenterView
```
setTabItemViews(List<TabItemView> tabItemViews);
setTabItemViews(List<TabItemView> tabItemViews, View centerView);
```

- 提供 setOnTabItemSelectListener 方法，用于设置 TabItem 的选择监听，可以实现 ViewPager 的页面切换或者 Fragment 切换。
```
bottomTabView.setOnTabItemSelectListener(new BottomTabView.OnTabItemSelectListener() {
    @Override
    public void onTabItemSelect(int position) {
        //viewPager.setCurrentItem(position, true);
    }
});  
```

- 还提供 setOnSecondSelectListener ，当 TabItemView 已经被选择并再次点击时，会出发此监听，用于实现某些 App 二次点击 TabItemView 刷新功能。

```
bottomTabView.setOnSecondSelectListener(new BottomTabView.OnSecondSelectListener() {
    @Override
    public void onSecondSelect(int position) {
        //refresh();
    }
});
```

你可以直接粘贴项目中这两个文件即可  
`BottomTabView.java`
`view_tab_item.xml`


***GitHub 地址***
> https://github.com/ifmvo/BottomTabView

## 开始
下面利用 BottomTabView 实现我们想要的 BottomTabBaseActivity

### 搭建整体布局
我们新建两个文件 BottomTabBaseActivity.java 和 activity_base_bottom_tab.xml，并把 BottomTabView.java 和 view_tab_item.xml 复制到我们的项目中

<center>
![](/img/bottom_tab/bottomtabview3.png)
</center>

activity_base_bottom_tab.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.v4.view.ViewPager
        android:id="@+id/viewPager"
        android:layout_weight="1"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

    <View
        android:layout_width="match_parent"
        android:background="#c7c7c7"
        android:layout_height="0.1dp"/>

    <cn.ifmvo.bottomtab.BottomTabView
        android:id="@+id/bottomTabView"
        android:gravity="center"
        android:layout_width="match_parent"
        android:layout_height="50dp"/>

</LinearLayout>
```

BottomTabBaseActivity 设置成抽象类，并添加三个方法
```
public abstract class BottomTabBaseActivity extends AppCompatActivity{

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_base_bottom_tab);

    }
}

protected abstract List<BottomTabView.TabItemView> getTabViews();

protected abstract List<Fragment> getFragments();

protected View getCenterView(){
    return null;
}
```

我们为什么写两个 abstract 类型的方法呢，我们的思路是这样的：  

- 可变的东西（底部菜单的 Item、Fragment）放在子类去实现
- 父类 BottomTabBaseActivity 实现不变的、重复性的代码

所以声明两个抽象方法 getTabViews、getFragments ，并在子类去实现，至于 getCenterView 方法可以自行定制，如果需要 CenterView 在子类实现方法即可。


MainActivity 继承自 BottomTabBaseActivity，只需实现两个方法，其他的什么也不需要做。

```
public class MainActivity extends BottomTabBaseActivity {

    @Override
    protected List<BottomTabView.TabItemView> getTabViews() {
        return null;
    }

    @Override
    protected List<Fragment> getFragments() {
        return null;
    }
}
```

到这里，整体的布局已经搭建完毕，下面我们来实现功能.  

### 实现底部 Tab

在 BottomTabBaseActivity 中添加：
```
if (getCenterView() == null){
    bottomTabView.setTabItemViews(getTabViews());
}else {
    bottomTabView.setTabItemViews(getTabViews(), getCenterView());
}
```

在 MainActivity 中实现 getTabView 方法

```
@Override
protected List<BottomTabView.TabItemView> getTabViews() {
    List<BottomTabView.TabItemView> tabItemViews = new ArrayList<>();
    tabItemViews.add(new BottomTabView.TabItemView(this, "标题1", R.color.colorPrimary,
            R.color.colorAccent, R.mipmap.ic_launcher, R.mipmap.ic_launcher_round));
    tabItemViews.add(new BottomTabView.TabItemView(this, "标题2", R.color.colorPrimary,
            R.color.colorAccent, R.mipmap.ic_launcher, R.mipmap.ic_launcher_round));
    tabItemViews.add(new BottomTabView.TabItemView(this, "标题3", R.color.colorPrimary,
            R.color.colorAccent, R.mipmap.ic_launcher, R.mipmap.ic_launcher_round));
    tabItemViews.add(new BottomTabView.TabItemView(this, "标题4", R.color.colorPrimary,
            R.color.colorAccent, R.mipmap.ic_launcher, R.mipmap.ic_launcher_round));
    return tabItemViews;
}
```

TabItemView 构造方法的参数含义
```
/**
 * @param title 标题
 * @param colorDef 标题默认的颜色
 * @param colorPress 标题被选中时的颜色
 * @param iconResDef 默认的图标
 * @param iconResPress 被选中时的图标
 */
public TabItemView(Context context, String title, int colorDef, int colorPress, int iconResDef, int iconResPress);
```

现在你可以运行一下你的 MainActivity，是不是已经实现了 底部 Tab 的效果。

<center>
![](/img/bottom_tab/bottomtabview5.gif)
</center>

### 连接 BottomTabView 和 ViewPager

我们继续。接下来我们将 BottomTabView 和 ViewPager 进行连接，并切换 ViewPager 的页面，以显示不同的 Fragment。  

声明、初始化一个 Adapter，并设置到 ViewPager 里面
```
FragmentPagerAdapter adapter;

adapter = new FragmentPagerAdapter(getSupportFragmentManager()) {
    @Override
    public Fragment getItem(int position) {
        return getFragments().get(position);
    }

    @Override
    public int getCount() {
        return getFragments().size();
    }
};

viewPager.setAdapter(adapter);
```

利用 setOnTabItemSelectListener 方法设置 BottomTabView 的监听器，来切换 ViewPager
```
bottomTabView.setOnTabItemSelectListener(new BottomTabView.OnTabItemSelectListener() {
    @Override
    public void onTabItemSelect(int position) {
        viewPager.setCurrentItem(position, true);
    }
});
```

用 ViewPager 的 addOnPageChangeListener 方法，使 ViewPager 切换的时候，也同步切换 BottomTabView 的选中位置。
```
viewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {}

    @Override
    public void onPageSelected(int position) {
        bottomTabView.updatePosition(position);
    }

    @Override
    public void onPageScrollStateChanged(int state) {}
});
```

最后一步：实现 getFragments 方法 返回一个 List<Fragment>
```
@Override
protected List<Fragment> getFragments() {
    List<Fragment> fragments = new ArrayList<>();
    fragments.add(new TabFragment1());
    fragments.add(new TabFragment2());
    fragments.add(new TabFragment3());
    fragments.add(new TabFragment4());
    return fragments;
}
```

当然 TabFragment 需要你自己去实现。 现在你再运行一下你的 MainActivity，是不是已经实现了。  

<center>
![](/img/bottom_tab/bottomtabview1.gif)
</center>

你想一想，以后每个 BottomTab + ViewPager 的界面，只需 让你的 XXActivity extends BottomTabBaseActivity，再实现两个方法就可以了，多爽啊！  

### 添加 CenterView

如果你想添加一个 CenterView 的话，你可以实现 getCenterView

```
@Override
protected View getCenterView() {
    ImageView centerView = new ImageView(this);
    centerView.setImageResource(R.mipmap.ic_launcher_round);
    LinearLayout.LayoutParams layoutParams = new LinearLayout.LayoutParams(200, 200);
    layoutParams.leftMargin = 60;
    layoutParams.rightMargin = 60;
    layoutParams.bottomMargin = 0;
    centerView.setLayoutParams(layoutParams);
    centerView.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {

        }
    });
    return centerView;
}
```

或者这样写比较灵活：

```
@Override
protected View getCenterView() {
    View centerView = LayoutInflater.from(this).inflate(R.layout.centerview, null);
    centerView.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {

        }
    });
    return centerView;
}
```

<center>
![](/img/bottom_tab/bottomtabview7.png)
</center>

如果你想让 CenterView 的上边界暴露在 BottomTabView 的外面的话，只需修改两处代码：

- 将 BottomTabView 内部的 View 设置成 底部对齐
- 在最外层布局添加属性并设置成 false

> 前提是你要保证 BottomTabView 内部的 CenterView 足够大。

***activity_base_bottom_tab.xml***
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    ...
    android:clipChildren="false"
    ...>

	...

    <cn.ifmvo.bottomtab.BottomTabView
        ...
        android:gravity="bottom"
        .../>

</LinearLayout>
```
<center>
![](/img/bottom_tab/bottomtabview8.png)
</center>

## Prefect !~~


***GitHub 地址***

> https://github.com/ifmvo/MatthewBases

***本文出自 陈序员 的博客，转载必须注明出处。***
> https://blog.ifmvo.cn/2017/05/03/package-bottomtabbaseactivity/

<center>
![](/img/blog/huba.png)
<p><strong>陈序员</strong></p>
<p>https://blog.ifmvo.cn</p>
</center>
