---
title: Android 快速开发（一），封装一个 TopBarBaseActivity
tag: Android
date: 2017-04-14 14:44:28
---


## 效果


什么是快速开发嘞，看这个效果

<center>
![](/img/toolbar1.png)
</center>

然而我只用了这么几行代码：

***MainActivity.java***

```
public class MainActivity extends TopBarBaseActivity {

    @Override
    protected int getContentView() {
        return R.layout.activity_main;
    }

    @Override
    protected void init(Bundle savedInstanceState) {
        setTitle("陈序员");

        setTopLeftButton(R.drawable.ic_return_white_24dp, new OnClickListener() {
            @Override
            public void onClick() {
                Toast.makeText(MainActivity.this, "陈序员点击了左上角按钮！", Toast.LENGTH_LONG).show();
            }
        });

        setTopRightButton("Button", R.drawable.ic_mine_white_24dp, new OnClickListener() {
            @Override
            public void onClick() {
                Toast.makeText(MainActivity.this, "点击了右上角按钮！", Toast.LENGTH_LONG).show();
            }
        });
    }
}

```

***activity_main.xml 里面什么也没有！***

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

</LinearLayout>
```

其实说白了哈，就是我把 TopBar 封装在 TopBarBaseActivity 里面，然后 MainActivity 只需 继承 TopBarBaseActivity 即可。 你想一想，以后每个上面有 TopBar 的界面，只需 让你的 XXActivity extends TopBarBaseActivity，就可以随意的设置你的 TopBar 了，多爽啊！。下面跟着我一步一步的实现我们想要的 TopBarBaseActivity。

## 开始

### 依赖 appcompat-v7

新建一个项目，因为 Toolbar 是在 appcompat-v7 包下，所以确保已经依赖 appcompat-v7。如果没有请加上，例如：

```
compile 'com.android.support:appcompat-v7:25.3.0'
```

### 设置 NoActionBar 主题

由于我们使用 Toolbar 代替 ActionBar，所以先把 ActionBar 去掉，我们通过设置 Application 的 theme 来隐藏，这样项目中所有的界面的 ActionBar 就都隐藏了。  
先修改 style.xml 中的 AppTheme 继承自 Theme.AppCompat.Light.NoActionBar

```
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorAccent</item>
</style>
```

AndroidManifest.xml 文件中默认就会设置主题的

```
<application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:theme="@style/AppTheme">

    ...

</application>
```

### 创建 TopBarBaseActivity

new 一个项目之后会默认创建一个 MainActivity.java 和 activity_main.xml ，然后我们再自己新建两个文件 TopBarBaseActivity.java 和 activity_base_top_bar.xml

<center>
![](/img/toolbar2.png)
</center>


然后 TopBarBaseActivity 继承 AppCompatActivity，MainActivity 继承 TopBarBaseActivity  

```
public class TopBarBaseActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_base_top_bar);
    }
}
```

```
public class MainActivity extends TopBarBaseActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

### 把 TopBar 封装在 TopBarBaseActivity

首先我们在 activity_base_top_bar.xml 中 添加 Toolbar

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout  
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:orientation="vertical"
    android:layout_height="match_parent">

    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:background="@color/colorPrimary"
        android:layout_height="wrap_content">

    </android.support.v7.widget.Toolbar>

</LinearLayout>
```

然后在 TopBarBaseActivity 中获取并初始化设置 Toolbar

```
public class TopBarBaseActivity extends AppCompatActivity {

    Toolbar toolbar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_base_top_bar);

        toolbar = (Toolbar) findViewById(R.id.toolbar);

        setSupportActionBar(toolbar);
        //设置不显示自带的 Title
        getSupportActionBar().setDisplayShowTitleEnabled(false);
    }
}
```

到这里，MainActivity 和 TopBarBaseActivity 还是显示它们各自自己的布局，所以我们要实现让 MainActivity 的布局 显示在 TopBarBaseActivity 的布局中，我们先在 activity_base_top_bar.xml 中添加一个 FrameLayout ，这样我们就可以把 MainActivity 的布局解析到 FrameLayout 里面

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    ...
    android:layout_height="match_parent">

    <android.support.v7.widget.Toolbar
          ...
    </android.support.v7.widget.Toolbar>

    <FrameLayout
        android:id="@+id/viewContent"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    </FrameLayout>

</LinearLayout>
```

然后我们修改下 TopBarBaseActivity  

```
public abstract class TopBarBaseActivity extends AppCompatActivity {

    Toolbar toolbar;
    FrameLayout viewContent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_base_top_bar);

        toolbar = (Toolbar) findViewById(R.id.toolbar);
        viewContent = (FrameLayout) findViewById(R.id.viewContent);

        //初始化设置 Toolbar
        setSupportActionBar(toolbar);
        getSupportActionBar().setDisplayShowTitleEnabled(false);

        //将继承 TopBarBaseActivity 的布局解析到 FrameLayout 里面
        LayoutInflater.from(TopBarBaseActivity.this).inflate(getContentView(), viewContent);
        init(savedInstanceState);

    }

    protected abstract int getContentView();
    protected abstract void init(Bundle savedInstanceState);
}
```

> 注意：为了设置两个抽象方法，我们将 TopBarBaseActivity 设置成了抽象类。

接下来让我们的 MainActivity 自动实现两个父类的抽象方法、删掉 onCreate 方法，并通过 getContentView 方法返回布局 activity_main 的 id。

```
public class MainActivity extends TopBarBaseActivity {

    @Override
    protected int getContentView() {
        return R.layout.activity_main;
    }

    @Override
    protected void init(Bundle savedInstanceState) {

    }
}
```

现在你运行一下项目，我们并没有在 MainActivity 的布局中添加 ToolBar,但是运行出来的效果是 Toolbar 已经存在了。

<center>
![](/img/toolbar3.png)
</center>

### 实现一句话添加标题

接下来我们要实现在 MainActivity 里面 `setTitle("标题"); `一句话设置标题。

首先我们修改 activity_base_top_bar.xml 布局文件，在 Toolbar 中添加一个 TextView 用来显示标题


```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    ...
    android:layout_height="match_parent">

    <android.support.v7.widget.Toolbar
          ...
          android:layout_height="wrap_content">

          <TextView
            android:id="@+id/tvTitle"
            android:layout_width="wrap_content"
            style="@style/TextAppearance.AppCompat.Widget.ActionBar.Title"
            android:textColor="@android:color/white"
            android:layout_gravity="center"
            android:layout_height="wrap_content" />

    </android.support.v7.widget.Toolbar>

    <FrameLayout
        android:id="@+id/viewContent"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    </FrameLayout>

</LinearLayout>
```

再修改 TopBarBaseActivity 添加一个 setTitle 方法

```
public abstract class TopBarBaseActivity extends AppCompatActivity {

    Toolbar toolbar;
    FrameLayout viewContent;
    TextView tvTitle;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_base_top_bar);

        toolbar = (Toolbar) findViewById(R.id.toolbar);
        viewContent = (FrameLayout) findViewById(R.id.viewContent);
        tvTitle = (TextView) findViewById(R.id.tvTitle);

        //初始化设置 Toolbar
        setSupportActionBar(toolbar);
        getSupportActionBar().setDisplayShowTitleEnabled(false);

        //将继承 TopBarBaseActivity 的布局解析到 FrameLayout 里面
        LayoutInflater.from(TopBarBaseActivity.this).inflate(getContentView(), viewContent);
        init(savedInstanceState);

    }

    protected void setTitle(String title){
        if (!TextUtils.isEmpty(title)){
            tvTitle.setText(title);
        }
    }

    protected abstract int getContentView();
    protected abstract void init(Bundle savedInstanceState);
}

```

然后我们就可以在 MainActivity 中调用了 setTitle 方法了

```
public class MainActivity extends TopBarBaseActivity {

    @Override
    protected int getContentView() {
        return R.layout.activity_main;
    }

    @Override
    protected void init(Bundle savedInstanceState) {
        setTitle("陈序员");
    }
}
```

效果图：

<center>
![](/img/toolbar4.png)
</center>

### 实现一句话添加左上角按钮

接下来我们的目标是通过在 MainActivity 中 `一句话` 实现自定义 TopBar 左上角的 `图标` 和 `点击监听`  

首先修改 TopBarBaseActivity, 我们需要造一个接口并声明

```
OnClickListener onClickListenerTopLeft;

public interface OnClickListener{
    void onClick();
}
```

然后重写 方法并处理点击事件

```
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    if (item.getItemId() == android.R.id.home){
        onClickListenerTopLeft.onClick();
    }

    return true; // true 告诉系统我们自己处理了点击事件
}
```

再添加一个方法用于设置图标资源 id 和 监听器

```
protected void setTopLeftButton(int iconResId, OnClickListener onClickListener){
    toolbar.setNavigationIcon(iconResId);
    this.onClickListenerTopLeft = onClickListener;
}
```

接下来我们就可以在 MainActivity 中这样写

```
public class MainActivity extends TopBarBaseActivity {

    @Override
    protected int getContentView() {
        return R.layout.activity_main;
    }

    @Override
    protected void init(Bundle savedInstanceState) {
        setTitle("陈序员");

        setTopLeft(R.mipmap.ic_launcher_round, new OnClickListener() {
            @Override
            public void onClick() {
                Toast.makeText(MainActivity.this, "陈序员", Toast.LENGTH_LONG).show();
            }
        });
    }
}
```

运行一下试试是不是只需一句话就设置了 TopBar 左按钮的 `图标` 和 `点击事件` ！

<center>
![](/img/toolbar5.png)
</center>

### 实现一句话添加右上角按钮

先创建一个 menu 文件夹和一个 menu 资源文件

<center>
![](/img/toolbar6.png)
</center>

内容是这样的：

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/menu_1"
        android:title=""
        app:showAsAction="always" />

</menu>
```

继续修改 TopBarBaseActivity ，添加如下代码：

```

OnClickListener onClickListenerTopRight;

// icon 图标id
int menuResId;

String menuStr;

protected void setTopRightButton(String menuStr, OnClickListener onClickListener){
    this.onClickListenerTopRight = onClickListener;
    this.menuStr = menuStr;
}

protected void setTopRightButton(String menuStr, int menuResId, OnClickListener onClickListener){
    this.menuResId = menuResId;
    this.menuStr = menuStr;
    this.onClickListenerTopRight = onClickListener;
}

@Override
public boolean onCreateOptionsMenu(Menu menu) {
    if (menuResId != 0 || !TextUtils.isEmpty(menuStr)){
        getMenuInflater().inflate(R.menu.menu_activity_base_top_bar, menu);
    }
    return true;
}

@Override
public boolean onPrepareOptionsMenu(Menu menu) {
    if (menuResId != 0) {
        menu.findItem(R.id.menu_1).setIcon(menuResId);
    }
    if (!TextUtils.isEmpty(menuStr)){
        menu.findItem(R.id.menu_1).setTitle(menuStr);
    }
    return super.onPrepareOptionsMenu(menu);
}
```

现在我们再 MainActivity 调用 setTopRightButton 方法的姿势是这样的

```
public class MainActivity extends TopBarBaseActivity {

    @Override
    protected int getContentView() {
        return R.layout.activity_main;
    }

    @Override
    protected void init(Bundle savedInstanceState) {
        setTitle("陈序员");

        setTopLeftButton(R.drawable.ic_return_white_24dp, new OnClickListener() {
            @Override
            public void onClick() {
                Toast.makeText(MainActivity.this, "陈序员点击了左上角按钮！", Toast.LENGTH_LONG).show();
            }
        });

        setTopRightButton("Button", R.drawable.ic_mine_white_24dp, new OnClickListener() {
            @Override
            public void onClick() {
                Toast.makeText(MainActivity.this, "点击了右上角按钮！", Toast.LENGTH_LONG).show();
            }
        });
    }
}
```

现在运行看下效果

<center>
![](/img/toolbar7.png)
</center>

到这里我们的 TopBarBaseActivity 就封装完了，过程比较简单，重要的是这个封装的思路，其实还有很多我们常用的布局都是可以进行封装的。

## 最后
可能有些人还会有些疑问：为什么要用 ToolBar 进行封装呢，自己写一个 TopBar 不是更好吗？

是的，如果你用不惯 Toolbar 你就可以根据自己的习惯写一个扩展性更好的 TopBar。但是我觉得 Toolbar 其实也有一些优质的品德的：

1. 官方提供，符合标准，界面统一。
2. 自带点击效果。
3. 自带点击热区。

接下来，我可能还会继续封装一些常用的 BaseActivity 。


***GitHub 地址***

> https://github.com/ifmvo/MatthewBases

***本文出自 陈序员 的博客，转载必须注明出处。***
> https://blog.ifmvo.cn/2017/04/14/package-topbarbaseactivity-by-toolbar/

<center>
![](/img/blog/huba.png)
<p><strong>陈序员</strong></p>
<p>https://blog.ifmvo.cn</p>
</center>
