###1. 按返回键不销毁Activity
<pre>
<code>
 //
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if(keyCode == KeyEvent.KEYCODE_BACK){
            //1    
            //moveTaskToBack(false);仅当前Activity为task根时，将activity退到后台而非finish();
            moveTaskToBack(true);

            //2   
            //按下Back键的回调函数，转成Home键的效果
            Intent intent = new Intent();
            intent.setAction(Intent.ACTION_MAIN);
            intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            intent.addCategory(Intent.CATEGORY_HOME);
            startActivity(intent);
            return true;
        }
        return super.onKeyDown(keyCode, event);
    }
</code></pre>   
###2. Activity启动过程，原理
首先要了解[Binder](#binder)机制 
<pre>
<code>
 
</code></pre>  

###3. 什么是Service？启动过程？原理
首先要了解[Binder](#binder)机制
<pre>
<code>
 
</code></pre>  
###4. Service 弹出Dialog Toast  
**对于toast**，相对来说比较简单，因为toast可以接受Application的context，可以直接show，也可以写在一个线程里面
<pre>
<code>
    @Override
    public IBinder onBind(Intent intent) {
        UtilsLog.d();

        Handler handler = new Handler(Looper.getMainLooper());
        handler.post(new Runnable() {
            public void run() {
                Toast.makeText(AppService.this, "onBind", Toast.LENGTH_LONG).show();
            }
        });
        return stub;
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        UtilsLog.d();

        Toast.makeText(this, "onStartCommand", Toast.LENGTH_LONG).show();
    }
</code></pre>  
Service里面直接弹出Dialog会抛出如下异常  
<pre>    
Caused by: android.view.WindowManager$BadTokenException: Unable to add window -- token null is not for an application</pre>   
但是，**对于Dialog**，因为Dialog只接受Activity的Context，而如果需要在Service中弹出，需要在Servcie中显示需要把dialog设置   
成一个系统的dialog，即全局 性质的提示框   
dialog.getWindow().setType((WindowManager.LayoutParams.TYPE_SYSTEM_ALERT));   
系统的Dialog需要在清单文件中添加权限，否则不会显示出来   
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
<pre><code>
    public void showDialog1() {
        UtilsLog.d();
        final AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("test");
        builder.setIcon(R.mipmap.icon);
        builder.setMessage("test");
        builder.setPositiveButton("ok", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {

            }
        });

        builder.setNegativeButton("cancel", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {

            }
        });
        Handler handler = new Handler(Looper.getMainLooper());
        handler.post(new Runnable() {
            public void run() {

                dialog = builder.create();
                dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_DIALOG);
                dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT);
                dialog.show();
            }
        });
    }
</code></pre>
需要注意的是，如果用TYPE_SYSTEM_ALERT，某些手机对底层进行了修改(小米，魅族之类)，系统会默认会拒绝该权限。   
解决：通过将type设定为TYPE_TOAST，就可以绕过检查，这时候需要改动TYPE_SYSTEM_ALERT为TYPE_TOAST   
**另外对于6.0以上的，因为有权限的限制所以还需要对权限有一个判断**
<pre><code>
 if(Build.VERSION.SDK_INT > Build.VERSION_CODES.M && !Settings.canDrawOverlays(mActivity)){
    final Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:" + mActivity.getPackageName()));
    startActivityForResult(intent,808);
 }

    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        try {
            mIappAidlInterface.showDialog();
        } catch (RemoteException e) {
            e.printStackTrace();
        }

    }
</code></pre>

###5. Local Service & Remote Service   
Local Service 本地服务，就是一个单纯的Service   
Remote Service 远程服务 [AIDL](#AIDL) [Binder](#binder)

<pre>
<code>
 
</code></pre>  

###6. linux基础命令

<pre>
<code>
 
</code></pre>  

###7. Handler原理
为什么要有handler，简单来说就是因为不能再主线程以外的地方更新ui，所以handler就出现了。

消息队列MessageQueue Looper是何时创建的？   
1. 对于主线程，即UIThread,查看ActivityThread的main方法，主线程的Looper和MessageQueue就是在此时创建的。   
2. 当我们之间new Handler对象时，
<pre><code>
 Handler mHandler = new Handler(new Handler.Callback() {
    @Override
    public boolean handleMessage(Message msg) {
        return false;
    }
});
//请看Handler的构造方法
public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }
		//创建looper
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
		//创建queue
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
从代码中可以看到，一个Handler里面包含 looper 跟 queue 引用
</code></pre>
3. new 一个Thread时，我们可以参考HandlerThread代码   
<pre><code>
@Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }
</code></pre> 

Android 是消息驱动的，我们得搞懂Handler、 Looper、 Message、 MessageQueue之间的关系，知道他们都是什么，我们来分析一下：
那么 ，首先，我们得有一个消息 Message，   
其次，有了消息Message后，得把消息存起来吧，于是又了消息队列MessageQueue   
再次，当消息都在队列里面，那么我们得处理，要处理就得一个一个拿出来吧，那么该怎么拿出来呢，于是我们右创建了一个消息循环Looper，用于循环取出消息   
最后，我们有了消息，有了消息队列，有了可以取出消息的Looper，那么我们的消
             息该怎么放入消息队列呢，又如何处理呢？当然就是消息处理Handler了
总之，它们之间的关系大体如下图：
![](http://i.imgur.com/67CD8ei.png)   
在使用Handler前，我们必须明白一点：
android没有全局的消息队列，消息队列是和某个线程关联在一起的，每个线程有且只有一个消息队列MessageQueue   
**如何发送消息？**   
1. 要发送消息，首先要获取一个Message对象，不建议直接new Message,在Message类中有一个静态方法obtain，可以减少内存占用   
2. 有了Message后，可以直接用Handler的post、sendMessage等方法   
**消息如何返回到Handler处理？** 
查看Looper.loop();源码，有个无限循环方法一直取message   
<pre><code>
public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;
        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();
        for (;;) {  //无限循环
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }
            // This must be in a local variable, in case a UI event sets the logger
            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }
            msg.target.dispatchMessage(msg);
            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }
            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
</code></pre>
**for (;;) {  //无限循环 为啥不会ANR**   
主线程的looper开启无敌模式，不断的读取消息列表，看是否有消息需要处理   
最开始Android的入口ActivityThread里面的main方法，里面有一个巨大的Handler，然后会创建一个主线程的looper对象，这也是为什么直接在主线程拿Handler就有Looper的原因，在其他线程是要自己Looper.prepare()的。其实整个Android就是在一个Looper的loop循环的，整个Androidi的一切都是以Handler机制进行的，即只要有代码执行都是通过Handler来执行的，而所谓ANR便是🈯️Looper.loop没有得到及时处理，一旦没有消息，Linux的epoll机制则会通过管道写文件描述符的方式来对主线程进行唤醒与沉睡，android里调用了linux层的代码实现在适当时会睡眠主线程
###8. 动画原理

####View Animation（Tween Animation）渐变动画/补间动画
先来说说这个渐变的动画，View animation只能应用于View对象，而且只支持一部分属性，它只是改变了View对象绘制的位置，而没有改   变View对象本身   
无论是用纯java代码构建Animation对象，还是通过xml文件定义Animation，其实最终的结果都是   
<pre><code>Animation a = new AlphaAnimation(); //透明度
Animation b = new ScaleAnimation(); //缩放
Animation c = new RotateAnimation(); //旋转
Animation d = new TranslateAnimation(); //位移
而我们使用的时候，一般是用这样的形式：
View.startAnimation(a);
</code></pre>
要了解动画的原理，首先得知道[View的绘制流程](#drawable)   
当view调用startAnimation，我们查看源码，首先就是set一个animation赋值，然后调用invalidate进行view的刷新重绘
<pre>   public void startAnimation(Animation animation) {
        animation.setStartTime(Animation.START_ON_FIRST_FRAME);
        setAnimation(animation);
        invalidateParentCaches();
        invalidate(true);
    }

    public void setAnimation(Animation animation) {
        mCurrentAnimation = animation;

        if (animation != null) {
            // If the screen is off assume the animation start time is now instead of
            // the next frame we draw. Keeping the START_ON_FIRST_FRAME start time
            // would cause the animation to start when the screen turns back on
            if (mAttachInfo != null && mAttachInfo.mDisplayState == Display.STATE_OFF
                    && animation.getStartTime() == Animation.START_ON_FIRST_FRAME) {
                animation.setStartTime(AnimationUtils.currentAnimationTimeMillis());
            }
            animation.reset();
        }
    }</pre>
invalidate调用后经过一系列的判断只重绘需要绘制的view，接下来就到了view的绘制流程中了   
<pre><code>
    boolean draw(Canvas canvas, ViewGroup parent, long drawingTime) {
        ......
        //这里获取的是前面set的animation
        final Animation a = getAnimation();
        if (a != null) {
            //当view需要动画时，
            more = drawAnimation(parent, drawingTime, a, scalingRequired);
            ......
        } else {
           ......
        }</code></pre>  
在view的drawAnimation中，处理动画的时间、差值器，然后调用Animation.applyTransformation，动画的执行都在Animation的实现类中

<pre>
<code>
 
</code></pre>  
###9. 做了什么项目，有哪些问题比较深刻，学习到了什么

<pre>
<code>
 
</code></pre>  
###10. 技术上的亮点

<pre>
<code>
 
</code></pre>  
###11. 单链表的操作   
单链表节点本身有个值，然后还有一个next属性，就是这样一个一个的节点连接起来的
<pre>
<code>
先来定义一个节点类   
public class Node{
    public int value;
    public Node next;
    public void setValue(...){...}
    public void setNext(...){...}
    ....
}
Node head = new Node(-1);
Node tempNode = head;
/**
* 首先我们要有一个链表
*/
for(int i = 0; i < 3; i++) {
    Node node = new Node(i);
    tempNode.next = node;
    tempNode = node;
}
再来一个方法，将链表反序  
public void test(Node node){
     Node result = null;
     Node head = null;
     Node next = node;
     while(next != null){
         result = next;
	 next.next = head;
	 head = next;
	 next = result;
     }
}
</code></pre>  

###12. 进程与线程的区别
（1）调度：线程作为调度和分配的基本单位，进程作为拥有资源的基本单位
（2）并发性：不仅进程之间可以并发执行，同一个进程的多个线程之间也可并发执行
（3）拥有资源：进程是拥有资源的一个独立单位，线程不拥有系统资源，但可以访问隶属于进程的资源.
（4）系统开销：在创建或撤消进程时，由于系统都要为之分配和回收资源，导致系统的开销明显大于创建或撤消线程时的开销。

###13. 递归与for循环是否可以转换 ，递归的弊端   
递归跟for是可以转换的
<pre><code>
for(int i = 0; i < 10; i++){
    for(int j = 0; j < 10; j++){
        System.out.println(""+i+j);
    }
}
public void print(String result, int depth){
    for(int i = 0; i < 10; i++) {
       result += i;
       if(depth == 1){
          System.out.println(result);
       } else {
           print(result, depth -1);
       }
       result = result.subString(0, result.length() -1);
       
    }
}
</code></pre>   
举个栗子   
<pre>
<code>
 public int sum(int i) {
     if(i > 0){
         return sum(i -1) + i;
     } else {
         return 0;
     }
 }
</code></pre>  
这是一个求和的递归方法，比如i=10000比较大，循环的调用方法，要额外的增加方法栈的开销，消耗内存容易出错
<pre>Exception in thread "main" java.lang.StackOverflowError</pre>

###14. 抽象类跟接口   
简单来说，抽象类是自下而上，将一些类共有的特性抽象出来，比如人可以抽象出性别，年龄，身高等等，   
然后每个人的行为是不一样的，比如说话，唱歌，职业等这些就可以用接口来定义   
通常情况下，混用貌似没多大的区别，但是在架构层，就可以是整体结构更清晰
<pre>
<code>
public abstract class Person{
     public abstract String sex();
     public abstract String age();
     public abstract String height();
}
public interface Action{
     void speak();
     void song();
     void work();
}
</code></pre>  
抽象类方式中，抽象类可以拥有任意范围的成员数据，同时也可以拥有自己的非抽象方法，但是接口方式中，它仅能够有静态、不能修改的成员数据（但是我们一般是不会在接口中使用成员数据），同时它所有的方法都必须是抽象的。在某种程度上来说，接口是抽象类的特殊化。   
      对子类而言，它只能继承一个抽象类（这是java为了数据安全而考虑的），但是却可以实现多个接口   
###15. String StringBuffer   
String的实现中就用到了StringBuffer StringBuilder,要尽量避免使用String a = ""＋"";这种使用+号来拼接字符，因为这样会额外的创建string stringbuffer等对象   
如果操作少量数据，用string，如果单线程操作大量数据用stringbuilder，多线程操作大量数据用stringbuffer   
从安全上来说，stringbuffer是线程安全的，它的大部分方法都是用了线程锁synchronized来描述。(信扩奶子的- _-!)   
stringbuffer & stringbuilder实现上都差不多，可以看源码

stringbuilder一般用在方法内部，应为是线程不安全的，用完后就可以gc掉了
stringbuffer 一般用在全局变量上，因为不知道这个变量会不会再线程里面

<pre>
<code>
 
</code></pre>  
###16. handler运行在哪个线程   
消息处理的本质是，开启一个无限循环的线程，不断的查询自己的消息队列是否有需要处理的消息
<pre>
<code>
 public class TestRunnableHandler implements Runnable {

    public TextView mTextView;
    public Handler mHandler;

    public TestRunnableHandler(TextView mTextView) {
        this.mTextView = mTextView;
    }

    @Override
    public void run() {

        Looper.prepare();
        //这个线程里面的handler是不能更新ui的
        mHandler = new Handler(){
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                UtilsLog.d(Thread.currentThread().getName());
                //mTextView.setText(Thread.currentThread().getName());
            }
        };
        UtilsLog.d("Looper.myLooper() " + Looper.myLooper());
        UtilsLog.d("mHandler.getLooper() " + mHandler.getLooper());
        mHandler.sendEmptyMessageDelayed(0, 7000);
        Looper.loop();
    }
}
public class TestThreadHandler extends Thread {

    public TextView mTextView;
    public Handler mHandler;

    public TestThreadHandler(String name, TextView mTextView) {
        super(name);
        this.mTextView = mTextView;
    }

    //这是一个线程，这里面创建的handler 引用的是main线程的looper
    @Override
    public void run() {
        super.run();
        //Looper.prepare();
        mHandler = new Handler(){
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                UtilsLog.d(Thread.currentThread().getName());
                mTextView.setText(Thread.currentThread().getName());
            }
        };
        UtilsLog.d("Looper.myLooper() " + Looper.myLooper());
        UtilsLog.d("mHandler.getLooper() " + mHandler.getLooper());
        mHandler.sendEmptyMessageDelayed(0, 1000);
        //Looper.loop();
    }

    public Handler getmHandler() {
        return mHandler;
    }
}
public void initHandlerThread() {
		UtilsLog.d();
		mHandlerThread = new HandlerThread("my thread");
		mHandlerThread.start();

		Looper looper = mHandlerThread.getLooper();
		// 2 这个handler也是不能更新ui的，因为用的是HandlerThread，
		mHandler = new Handler(looper, new Handler.Callback() {
			// 该接口的实现就是处理异步耗时任务的，因此该方法执行在子线程中
			@Override
			public boolean handleMessage(Message msg) {
                UtilsLog.d(Thread.currentThread().getName());
				return false;
			}
		}) {
			@Override
			public void handleMessage(Message msg) {
				super.handleMessage(msg);
                UtilsLog.e(Thread.currentThread().getName());
				//mTextView.setText(Thread.currentThread().getName());
			}

		};

		UtilsLog.d("Looper.myLooper() " + Looper.myLooper());
		UtilsLog.d("mHandler.getLooper() " + mHandler.getLooper());
		mTestThreadHandler.run();
		new Thread(new TestRunnableHandler(mTextView)).start();

		mHandler2 = new Handler(){
			@Override
			public void handleMessage(Message msg) {
				super.handleMessage(msg);
				UtilsLog.d(Thread.currentThread().getName());
				mTextView.setText(Thread.currentThread().getName());
			}
		};
		mHandler3 = new Handler(){
			@Override
			public void handleMessage(Message msg) {
				super.handleMessage(msg);
				UtilsLog.d(Thread.currentThread().getName());
			}
		};

		UtilsLog.d("mHandler2 " + mHandler2);
		UtilsLog.d("mHandler3 " + mHandler3);
		UtilsLog.d("mHandler2.getLooper() " + mHandler2.getLooper());
		UtilsLog.d("mHandler3.getLooper() " + mHandler3.getLooper());

	}
</code></pre>  
###17. 内存溢出

<pre>
<code>
 
</code></pre>  
###18. 内存优化

<pre>
<code>
 
</code></pre>  
###19. Java的runtime机制

<pre>
<code>
 
</code></pre>  
###20. 6.0新特性

<pre>
<code>
 
</code></pre>  
###21. web app交互方式有哪些，promt

<pre>
<code>
 
</code></pre>  
###22. webview如何启动其他app

<pre>
<code>
 
</code></pre>  
###23. webview防止白屏

<pre>
<code>
 
</code></pre>  
###24. Service保活

<pre>
<code>
 
</code></pre>  

###25. Binder
<h4 id="binder">Binder</h4>  　 
在Linux里面进程间是相互隔离的，而Android是基于Linux开发，也充分利用隔离性，那么进程间是怎么通信的(IPC),
在Linux里面比较常见的几种IPC    

* Signals 信号量   
* Pipes 管道   
* Sockets 套接字      
* Message Queue 消息队列   
* Shared Memory 共享内存   
 
但是这些都不适合Android移动设备，所以有了Binder   
Binder本质是要解决多线程通信，如下面这样 c/s通信   
![](http://i.imgur.com/OFkcyru.png)   
那么有了binder后就是这样，同时binder应该支持多并发，这就是Binder最基础的架构
![](http://i.imgur.com/w7RlSAc.png)   
方法的跨进程调用是收到Linux限制的，那么Binder是如何跨进程调用的，解决方案就是将Binder放在Kernel层，内核共享层，Binder提供的功能就是
类似于DNS(将一个网址域名跟ip地址映射起来,我们在浏览器里面输入域名而不需要输入ip地址)，将各个进程的地址跟kernel中的地址进行关联。
然而，我们还需要知道这个域名，域名都是统一在ServiceManager统一管理,例如我们获取系统服务的方式
<pre><code>
// 获取电话服务
TelephonyManager tm = (TelephonyManager) ctx.getSystemService(Context.TELEPHONY_SERVICE);
    这个TelephonyManager 里面管理着ITelephony.aidl 等接口
    然后所有的操作都是通过这个远程服务接口来链接
// 获取网络连接的服务
ConnectivityManager cm = (ConnectivityManager) ctx.getSystemService(Context.CONNECTIVITY_SERVICE);
// 获取窗口服务
WindowManager wm = (WindowManager) ctx.getSystemService(Context.WINDOW_SERVICE);
</code></pre>
**在Android中对于AIDL的使用其实就很简单了，我们抛去原理，只要获取到AIDL的这个远程接口即可**,如下图，首先注册，然后调用
![](http://i.imgur.com/vBGo5FN.png)
![](http://i.imgur.com/UCdpmUG.png)   
**原理**
![](http://i.imgur.com/yWquxiW.png)
如图所示，对Binder的访问，使用的是代理的模式，

* IBinder是一个接口，它代表了一种跨进程传输的能力；只要实现了这个接口，就能将这个对象进行跨进程传递；
这是驱动底层支持的；在跨进程数据流经驱动的时候，驱动会识别IBinder类型的数据，从而自动完成不同进程Binder本地对象以及Binder代理对象的转换。
* IBinder负责数据传输，那么client与server端的调用契约（这里不用接口避免混淆）呢？
这里的IInterface代表的就是远程server对象具有什么能力。具体来说，就是aidl里面的接口。
* Java层的Binder类，代表的其实就是Binder本地对象。BinderProxy类是Binder类的一个内部类，
它代表远程进程的Binder对象的本地代理；这两个类都继承自IBinder, 因而都具有跨进程传输的能力；
实际上，在跨越进程的时候，Binder驱动会自动完成这两个对象的转换。
* 在使用AIDL的时候，编译工具会给我们生成一个Stub的静态内部类；这个类继承了Binder, 说明它是一个Binder本地对象，
它实现了IInterface接口，表明它具有远程Server承诺给Client的能力；Stub是一个抽象类，
具体的IInterface的相关实现需要我们手动完成，这里使用了策略模式。

###26. Service Thread###
1. Thread优先级低于Service，Service属于一个系统组件，当内存不足是优先kill掉Thread   
2. Service由ServiceManager管理，运行在主线程中，而Thread只是一个子线程，比如在Activity里面启动一个Thread，当Activity关掉后，就无法控制Thread了   
3. Service可以跨进程调用   
 
如果是长时间运行且不需要ui交互的，则用service，同样是在后台运行，不需要交互的情况下，如果只是完成某个任务，之后就不需要运行，而且可能是多个任务，需需要长时间运行的情况下使用Thread;     
如果任务占用CPU时间多，资源大的情况下，要使用Thread   
###27. <h3 id="AIDL">AIDL</h3>
####什么是AIDL, 为什么要用AIDL####
>为了在不同的进程中进行通讯，获取数据，交互数据等，我们查看源码可以看到大量的AIDL文件   
core\java\android\os\IPermissionController.aidl   
\core\java\android\os\IPowerManager.aidl   
telecomm\java\com\android\internal\telecom\ITelecomService.aidl 等   
比如常见的有在自己的应用程序中读取手机联系人的信息，这就涉及到 IPC 了。因为自己的应用程序是一个进程，   
通讯录也是一个进程，只不过获取通讯录的数据信息是通过 Content Provider 的方式来实现的。   
多个客户端，多个线程并发的情况下要使用 AIDL  

####AIDL工作原理####
当创建一个IAppAidlInterface.aidl文件后，SDK工具会将aidl文件一个个的编译成继承 android.os.IInterface的java文件   
查看编译后的 IAppAidlInterface.java 文件   
<pre><code>
public interface IInterface{
    public IBinder asBinder();
}
</code></pre>    

IAppAidlInterface.java文件包含两部分
public static abstract class Stub 跟 在aidl文件中定义的方法
bindService时创建binder对象

#### AIDL 远程服务 进程间通信 的使用####
&emsp;2.2.1 定义AIDL接口
       通过AIDL文件定义服务(Service)向客户端(Client)提供的接口，我们需要在对应的目录下添加一个后缀为.aidl的文件(注意，不是.java),
       IAppAidlInterface.aidl
       注：如果服务端与客户端不在同一App上，需要在客户端、服务端两侧都建立该aidl文件。

&emsp;2.2.2 创建本地服务 AppService.java ， 在Service中创立对应的stub 对象
       // 实现接口中暴露给客户端的Stub--Stub继承自Binder，它实现了IBinder接口
       IAppAidlInterface.Stub
       并且在onBinder() 方法中返回它

&emsp;2.2.3 调用远程服务
       在客户端中建立与Remote Service的连接，获取Stub，然后调用Remote Service提供的方法来获取对应数据
       IAppAidlInterface
       ServiceConnection
       ServiceAliveTestFragment.java

#### 不用AIDL ,使用Binder直接通讯
&emsp;我们将IAppAidlInterface.java拆开来写比如，我们不使用AIDL来实现IPC通讯，而直接用binder来写   
1.  定义一个继承android.os.IInterface的接口 IRemoteInterface   
2.  定义一个继承IBinderd的类RemoteInterfaceImpl并实现上面的接口IRemoteInterface   
3.  RemoteInterfaceImpl 中重写onTransact,在这个方法里面**读取参数，然后写入结果**   
    <pre><code>
         @Override
    protected boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
        if(code == INTERFACE_TRANSACTION){
//            reply.writeString(getInterfaceDescriptor());
            reply.writeString(DESCRIPTOR);
            return true;
        } else if(code == TRANSACTION_log){
            String message ;
            //首先读入参数
            message = data.readString();
            java.lang.String _result = this.log(message);
            reply.writeNoException();
            //将结果写入到reply中
            reply.writeString(_result);
            return true;
        }
        return super.onTransact(code, data, reply, flags);
    }
    </code></pre>
4.  RemoteInterfaceImpl内部写一个内部代理类， 该代理类代理的就是IRemoteInterface远程接口
<pre>private static class Proxy implements IRemoteInterface</pre>
该代理类要做的就是实现IRemoteInterface远程接口，在远程接口里面在这个方法里面**写入参数，然后读取结果**，并将远程Binder作为参数传入，
<pre><code>
    private static class Proxy implements IRemoteInterface {

        /**
         * @param message
         * @return
         * @throws RemoteException
         */
        @Override
        public String log(String message) throws RemoteException {
            android.os.Parcel _data = android.os.Parcel.obtain();
            android.os.Parcel _reply = android.os.Parcel.obtain();
            java.lang.String _result;
            try {
                _data.writeInterfaceToken(DESCRIPTOR);
                //首先将参数message写入到_data中去
                _data.writeString(message);
                //同时在 远程binder 调用结束后，得到返回的 _reply
                mRemote.transact(RemoteInterfaceImpl.TRANSACTION_log, _data, _reply, 0);
                _reply.readException();
                //在没有异常的情况下，返回_reply.readString()的结果,也就是读取值
                _result = _reply.readString();
            } finally {
                _reply.recycle();
                _data.recycle();
            }
            return _result;
        }
        private android.os.IBinder mRemote;

        Proxy(android.os.IBinder remote) {
            mRemote = remote;
        }

        @Override
        public android.os.IBinder asBinder() {
            return mRemote;
        }

        public java.lang.String getInterfaceDescriptor() {
            return DESCRIPTOR;
        }

    }
</code></pre>

<h3 id="drawable">28. View的绘制流程</h3>
从大的方面来说大体上分成三步来走，measure，layout，draw      
measure(测量) 要测量出view多宽，多高   
layout(布局)  view要画在哪个位置   
draw(画)      怎么画view   
我们从Activity SetContentView 说起，setContentView 调用LayoutInflater.inflate()对应的是PhoneWindow 的DecorView， 这是一个   
FrameLayout，也就是我们常说的根布局，一个DecorView里面又包含两个布局，一个是SystemUI的布局(状态栏)，一个是contentview   
View 有三种测量规格 MeasureSpec， 精确值，最大值，未指定，view的测量值是由view的MeasureSpec跟view的layoutParams共同决定      
performMeasure开始，调用view的measure，首先获取到屏幕的宽高，赋值给 measureSpecWidth、measureSpecHeight，如果view本身是个viewGroup，则调用子view的measure，并将上面的参数传入，并传入view的MeasrureSpec(即测量规格)
