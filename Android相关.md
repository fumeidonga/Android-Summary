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
