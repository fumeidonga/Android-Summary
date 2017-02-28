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
