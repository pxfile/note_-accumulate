---
layout:     post
title:      BroadcastReceiver与LocalBroadcastManager应用及区别
subtitle:   BroadcastReceiver与LocalBroadcastManager应用及区别
date:       2018-03-07
author:     pxf
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Android
---

BroadcastReceiver与LocalBroadcastManager应用及区别
===


android中有两种广播机制，一种是BroadcastReceiver，另一种是LocalBroadcastManager。现在来简单介绍一下两者。

## 一、应用场景

   1、BroadcastReceiver用于应用之间的传递消息；

   2、而LocalBroadcastManager用于应用内部传递消息，比broadcastReceiver更加高效。

## 二、安全

   1、BroadcastReceiver使用的Content API，所以本质上它是跨应用的，所以在使用它时必须要考虑到不要被别的应用滥用；

   2、LocalBroadcastManager不需要考虑安全问题，因为它只在应用内部有效。

## BroadcastReceiver的使用
```
private MyBroadcastReceiver myBroadcastReceiver; 
//实例化广播 
myBroadcastReceiver=new MyBroadcastReceiver();  
//注册广播  
registerReceiver(myBroadcastReceiver,new IntentFilter(ACTION));
//发送广播
Intent intent = new Intent();  
intent.setAction(ACTION);  
intent.putExtra(NAME,"Andy");
sendBroadcast(intent);  
//广播接收器
public static class MyBroadcastReceiver extends BroadcastReceiver  
    {  
        @Override  
        public void onReceive(Context context, Intent intent) {  
            String name=intent.getStringExtra(NAME);          Toast.makeText(context,name+":"+flag,Toast.LENGTH_SHORT).show();  
            ++flag;  
        }  
    }  
//销毁  
    @Override  
    protected void onDestroy() {  
        super.onDestroy();
		unregisterReceiver(myBroadcastReceiver);   
    }  
```
## LocalBroadcastManager的使用
```
private LocalBroadcastManager mLocalBroadcastManager;  
private BroadcastReceiver mBroadcastReceiver;  
//实例化
mLocalBroadcastManager=LocalBroadcastManager.getInstance(this); 
//过滤器
IntentFilter intentFilter=new IntentFilter();  
intentFilter.addAction(ACTION_STARTED);  
//广播接收器
mBroadcastReceiver=new BroadcastReceiver() {  
	@Override  
        public void onReceive(Context context, Intent intent) {  
            if(intent.getAction().equals(ACTION_STARTED))  {  
                    tv.setText("STARTED");  
                } 
            }  
        };     
//注册广播
mLocalBroadcastManager.registerReceiver(mBroadcastReceiver,intentFilter); 
//发送广播
mLocalBroadcastManager.sendBroadcast(new Intent(ACTION_STARTED)); 
//销毁
@Override  
protected void onDestroy() {  
    super.onDestroy();  
    mLocalBroadcastManager.unregisterReceiver(mBroadcastReceiver);  
}   
```

