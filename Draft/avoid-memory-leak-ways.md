---
title: Andorid 常见的内存泄露及解决办法
date: 2017-04-13
tag: Android
---

## 单例造成的内存泄漏

当调用 getInstance 时，如果传入的 context 是 Activity 的 context。只要这个单例没有被释放，那么这个 Activity 也不会被释放一直到进程退出才会释放。


```
public class CommUtil {
    private static CommUtil instance;
    private Context context;
    private CommUtil(Context context){
        this.context = context;
    }

    public static CommUtil getInstance(Context mcontext){
        if(instance == null){
            instance = new CommUtil(mcontext);
        }
        return instance;
    }
}
```

> 解决方案：能使用Application的Context就不要使用Activity的Content，Application的生命周期伴随着整个进程的周期

---


## 非静态内部类创建静态实例造成的内存泄漏

```
private static TestResource mResource = null;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        if(mManager == null){
            mManager = new TestResource();
        }

    }
    class TestResource {

    }
}
```


> 解决方案：将非静态内部类修改为静态内部类。（静态内部类不会隐式持有外部类）


---


## Handler造成的内存泄漏

mHandler是Handler的非静态匿名内部类的实例，所以它持有外部  类Activity的引用，我们知道消息队列是在一个Looper线程中不断轮询处理消息，那么当这个Activity退出时消息队列中还有未处理的消息或者正在处理消息，而消息队列中的Message持有mHandler实例的引用，mHandler又持有Activity的引用，所以导致该Activity的内存资源无法及时回收，引发内存泄漏。

```
private MyHandler mHandler = new MyHandler(this);
    private TextView mTextView ;
    private static class MyHandler extends Handler {
        private WeakReference<Context> reference;
        public MyHandler(Context context) {
            reference = new WeakReference<>(context);
        }
        @Override
        public void handleMessage(Message msg) {
            MainActivity activity = (MainActivity) reference.get();
            if(activity != null){
                activity.mTextView.setText("");
            }
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mTextView = (TextView)findViewById(R.id.textview);
        loadData();
    }

    private void loadData() {

        Message message = Message.obtain();
        mHandler.sendMessage(message);
    }
```


> 解决方案：创建一个静态Handler内部类，然后对Handler持有的对象使用弱引用，这样在回收时也可以回收Handler持有的对象，这样虽然避免了Activity泄漏，不过Looper线程的消息队列中还是可能会有待处理的消息，所以我们在Activity的Destroy时或者Stop时应该移除消息队列中的消息


```
private MyHandler mHandler = new MyHandler(this);
    private TextView mTextView ;
    private static class MyHandler extends Handler {
        private WeakReference<Context> reference;
        public MyHandler(Context context) {
            reference = new WeakReference<>(context);
        }
        @Override
        public void handleMessage(Message msg) {
            MainActivity activity = (MainActivity) reference.get();
            if(activity != null){
                activity.mTextView.setText("");
            }
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mTextView = (TextView)findViewById(R.id.textview);
        loadData();
    }

    private void loadData() {
        //...request
        Message message = Message.obtain();
        mHandler.sendMessage(message);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mHandler.removeCallbacksAndMessages(null);
    }
}
```

---


## 线程造成的内存泄漏

异步任务和Runnable都是一个匿名内部类，因此它们对当前Activity都有一个隐式引用。如果Activity在销毁之前，任务还未完成， 那么将导致Activity的内存资源无法回收，造成内存泄漏。

```
new AsyncTask<Void, Void, Void>() {
    @Override
    protected Void doInBackground(Void... params) {
        SystemClock.sleep(10000);
        return null;
    }
}.execute();


new Thread(new Runnable() {
    @Override
    public void run() {
        SystemClock.sleep(10000);
    }
}).start();
```

> 解决方案：使用 静态内部类，避免了Activity的内存资源泄漏，当然在Activity销毁时候也应该取消相应的任务AsyncTask::cancel()，避免任务在后台执行浪费资源

```
static class MyAsyncTask extends AsyncTask<Void, Void, Void> {
        private WeakReference<Context> weakReference;

        public MyAsyncTask(Context context) {
            weakReference = new WeakReference<>(context);
        }

        @Override
        protected Void doInBackground(Void... params) {
            SystemClock.sleep(10000);
            return null;
        }

        @Override
        protected void onPostExecute(Void aVoid) {
            super.onPostExecute(aVoid);
            MainActivity activity = (MainActivity) weakReference.get();
            if (activity != null) {
                //...
            }
        }
    }
    static class MyRunnable implements Runnable{
        @Override
        public void run() {
            SystemClock.sleep(10000);
        }
    }
//——————
    new Thread(new MyRunnable()).start();
    new MyAsyncTask(this).execute();
```

---


## 资源未关闭造成的内存泄漏
对于使用了BraodcastReceiver，ContentObserver，File，Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销，否则这些资源将不会被回收，造成内存泄漏.

> 解决方案：在Activity销毁时及时关闭或者注销

---


## 使用了静态的Activity和View

```
static view;

void setStaticView() {
    view = findViewById(R.id.sv_button);
}

View svButton = findViewById(R.id.sv_button);
svButton.setOnClickListener(new View.OnClickListener(){
    @Override
    public void onClick(View v) {
        setStaticView();
        nextActivity();
    }
});


static Activity activity;

void setStaticActivity() {
    activity = this;
}

View saButton = findViewById(R.id.sa_button);
saButton.setOnClickListener(new View.OnClickListener(){
    @Override
    public void onClick(View v) {
        setStaticActivity();
        nextActivity();
    }
});
```

> 解决方案：应该及时将静态的应用 置为null，而且一般不建议将View及Activity设置为静态

---


## 注册了系统的服务，但onDestory未注销

```
SensorManager sensorManager = getSystemService(SENSOR_SERVICE);
Sensor sensor = sensorManager.getDefaultSensor(Sensor.TYPE_ALL);
sensorManager.registerListener(this,sensor,SensorManager.SENSOR_DELAY_FASTEST);
```
> 解决方案：不需要的时候及时移除监听

```
sensorManager.unregisterListener(listener);
```

---


## 不需要用的监听未移除会发生内存泄露

```
//add监听，放到集合里面
tv.getViewTreeObserver().addOnWindowFocusChangeListener(
  new ViewTreeObserver.OnWindowFocusChangeListener() {
    @Override
    public void onWindowFocusChanged(boolean b) {
        //监听view的加载，view加载出来的时候，计算他的宽高等。
    }
});
```

> 解决方案：使用完后，一定要移除这个监听

```
tv.getViewTreeObserver()
.removeOnWindowFocusChangeListener(this);
```
```
//监听执行完回收对象，不用考虑内存泄漏
tv.setOnClickListener();
//add监听，放到集合里面，需要考虑内存泄漏
tv.getViewTreeObserver()
.addOnWindowFocusChangeListener(this);
```

---



参考文章：

https://www.jianshu.com/p/402225fce4b2

如果一不小心，还是让内存泄露了，怎么分析是在哪里泄露了呢？请看：

https://www.jianshu.com/p/2c9fc4e871a4
