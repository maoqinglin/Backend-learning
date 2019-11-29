## 1、Android中Handler源码详解

**1）Looper.prepare**

创建了Looper对象，在构造函数中创建了 MessageQueue

**然后通过ThreadLocal 将Looper对象跟当前的线程进行绑定**

```java
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}

private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```

**2）new Handler()**

通过Looper类中的 ThreadLocal 从主线程中获取到了 Looper 对象

**然后通过Looper对象获取到了 MessageQueue的引用**

```java
public Handler(Callback callback, boolean async) {
    .......

    mLooper = Looper.myLooper();
    ......
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

**3）handler.sendMsg()**

找到MessageQueue对象

然后 msg.target = this，将 handler 对象赋值给 msg 的一个成员变量

最后把 msg 添加到 MessageQueue中

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

**4）Looper.loop()**

找到MessageQueue

然后开启死循环遍历 MessageQueue池

当获取到 msg 的时候，通过 msg.target找到发送这个 msg 的handler

然后调用 handler 的dispatcheMessage

```java
public static void loop() {
    final Looper me = myLooper();

    final MessageQueue queue = me.mQueue;
	......
    for (;;) {
        Message msg = queue.next(); // might block
		......
        try {
            msg.target.dispatchMessage(msg);
            dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }
    }
}
```



应用：如何让主线程给子线程发消息

```java
public Handler mHandler;
public Looper subLooper;
class LooperThread extends Thread {
    
    public void run() {
    // 创建Looper对象，Looper中创建MessageQueue，通过ThreadLocal绑定子线程和Looper
        Looper.prepare();

        // 1、创建handler对象，然后从当前线程中获取Looper对象，然后获取到MessageQueue对象
        mHandler = new Handler() {
            public void handleMessage(Message msg) {
                // process incoming messages here
            }
        };

        subLooper = Looper.myLooper(); // 用于activity退出时释放资源
        Looper.loop();
    }
}

// 主线程拿到子线程的 mHandler 可以直接发消息

// 退出子线程loop死循环，避免内存泄露
protected void onDestory(){
    super.onDestory();
    if(myLooper != null){
        myLooper.quit();
        myLooper = null;
    }
}

```



## 2、Fragment

1）new Fragment 不会走声明周期

2）replace和add区别，两者都不会影响声明周期

replace是把容器中其它所有Fragment都清除了，再创建新的Fragment；

add只是添加新的Fragment，通过show与hide方法控制fragment的显示与隐藏节省资源，效率高

```kotlin
class MainActivity : FragmentActivity(), View.OnClickListener {

    var fragmentA = FragmentA();
    var fragmentB = FragmentB();
    var fragmentC = FragmentC();
    val fragmentList = mutableListOf<Fragment>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        findViewById<Button>(R.id.btnA).setOnClickListener(this)
        findViewById<Button>(R.id.btnB).setOnClickListener(this)
        findViewById<Button>(R.id.btnC).setOnClickListener(this)


        var beginTransaction = supportFragmentManager.beginTransaction()
        beginTransaction.add(R.id.fl, fragmentA, "FragmentA")
            .add(R.id.fl, fragmentB, "FragmentB")
            .add(R.id.fl, fragmentC, "FragmentC")
            .hide(fragmentB)
            .hide(fragmentC)
            .commit()

        addToBackStack(fragmentA)
    }

    private fun addToBackStack(fragment: Fragment) {
        if (fragmentList.contains(fragment)) {
            // 如果存在fragment先移除
            fragmentList.remove(fragment)
            // 再将fragment添加到最后
            fragmentList.add(fragment)
        } else {
            fragmentList.add(fragment)
        }
    }

    override fun onBackPressed() {
//        super.onBackPressed()  不能调用父类的处理后退栈逻辑
        if (fragmentList.size > 1) {
            // 移除最后的fragment
            fragmentList.removeAt(fragmentList.size - 1)
            // 显示（移除后的）最后的fragment
            showFragment(fragmentList.get(fragmentList.size - 1))
        } else {
            // 如果当前栈中没有Fragment或只有一个Fragment，退出activity
            finish()
        }
    }

    private fun showFragment(fragment: Fragment) {
        var transaction = supportFragmentManager.beginTransaction()
        transaction.hide(fragmentA).hide(fragmentB).hide(fragmentC)
            .show(fragment)
            .commit()
    }


    override fun onClick(v: View?) {
        var transaction = supportFragmentManager.beginTransaction()
        transaction.hide(fragmentA).hide(fragmentB).hide(fragmentC) // 先把所有的fragment隐藏
        when (v?.id) {
            R.id.btnA -> {
                transaction.show(fragmentA)
                addToBackStack(fragmentA)
            }
            R.id.btnB -> {
                transaction.show(fragmentB)
                addToBackStack(fragmentB)
            }
            R.id.btnC -> {
                transaction.show(fragmentC)
                addToBackStack(fragmentC)
            }
        }
        transaction.commit()
    }
}
```

## 3、Parcelable的使用

Parcelable的底层是Binder；**如果是进程间通信，底层使用了内核的共享内存**，一个进程写数据，另一个进程读数据

```kotlin
data class User(var name: String, var age: Int) : Parcelable {

    override fun describeContents(): Int {
        return 0
    }

    // 序列化，写的顺序与读的顺序必须一致
    override fun writeToParcel(dest: Parcel?, flags: Int) {
        dest?.writeString(name)
        dest?.writeInt(age)
    }

    // 反序列化
    companion object CREATOR : Parcelable.Creator<User> {
        override fun createFromParcel(parcel: Parcel): User {
            var user = User(parcel.readString(), parcel.readInt())
            return user
        }

        override fun newArray(size: Int): Array<User?> {
            return arrayOfNulls(size)
        }
    }
}

tv.setOnClickListener {
    var intent = Intent(context, SecondActivity::class.java)
    var user = User("xiaolin",20)

    intent.putExtra("user",user)
    startActivity(intent)
}

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    var user = intent.getParcelableExtra<User>("user")
    Log.i("lmq", user.toString())
}
```

## 4、Activity数据恢复

```kotlin
// 保存数据
override fun onSaveInstanceState(outState: Bundle?) {
    super.onSaveInstanceState(outState)
    Log.i("lmq","保存状态")
    outState?.putString("state","saveState")
}

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)// 测试Activity退至后台后，内存不足时，Activity数据恢复
    var reStoreStr = savedInstanceState?.get("state").toString()
    Log.i("lmq",reStoreStr)
}

// 操作EditText问题：
findViewById<TextView>(R.id.stateEt).text 
= Editable.Factory.getInstance().newEditable(str)
```

