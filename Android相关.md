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
Remote Service 远程服务 AIDL BINDER

<pre>
<code>
 
</code></pre>  

##面试相关
###6. linux基础命令

<pre>
<code>
 
</code></pre>  

###7. Handler原理

<pre>
<code>
 
</code></pre>  

###8. 动画原理

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

<pre>
<code>
 
</code></pre>  

###12. 进程与线程的区别

<pre>
<code>
 
</code></pre>  
###13. 递归与for循环是否可以转换 ，递归的弊端

<pre>
<code>
 
</code></pre>  
###14. 抽象类跟接口

<pre>
<code>
 
</code></pre>  
###15. string stringbuffer

<pre>
<code>
 
</code></pre>  
###16. handler运行在哪个线程

<pre>
<code>
 
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

###25. Binder & AIDL
<span id="binder">Binder</span>  　 
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
