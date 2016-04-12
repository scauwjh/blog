---
layout:     post
title:      "Android开发学习笔记（一）"
subtitle:   "布局和基本事件的学习"
date:       2015-10-19 01:08:00
author:     "Kei Wu"
header-img: "img/post-bg-06.jpg"
---

## 前言：
初衷：个人比较喜欢音乐，但是对市面上的各种音乐app都没什么好感，要么就是广告一大堆，要么就是用习惯不适合我，所以想自己动手做一个自己喜欢的音乐播放器（[github link](https://github.com/scauwjh/kmusic)），也刚好可以学习一下Android开发，一举两得。  

## 正文： 
样张：
![](https://raw.githubusercontent.com/scauwjh/kmusic/master/sample/2015-10-18.jpg)  

## AndroidMainfest.xml
AndroidMainfest.xml位于项目的根目录，描述了应用的一些基本信息和主题风格、授权声明、指定程序入口后台服务等等。  
{% highlight java %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="me.keiwu.kmusic" >
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.INTERNET" />
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/materialTheme">
        <activity android:name=".activity.MainActivity" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <service android:name=".service.MusicService"/>
    </application>
</manifest>
{% endhighlight %}
最外层标签<manifest>主要属性有package、versionCode、installLocation等等，声明一些基础的信息  
<uses-permission>声明应用所需要的一些权限，例如读写存储的权限  
<intent-filter>里面包含一些action，android.intent.action.MAIN指定应用的入口，android.intent.category.LAUNCHER指定应用是否出现在Launcher里面  
<service>用于定义后台服务，例如音乐播放器的核心播放器功能就应用到service来保持后台运行。（包含在application里面，一开始就是搞错位置导致启动失败。。）  

## res目录
res目录下主要是应用的一些静态资源  
drawable：导入的图标图片资源  
layout：应用布局文件  
values：布局用的一些静态变量，如颜色、尺寸、风格、标题等等  
...  

## 布局

### 一、LinearLayout线性布局
一般LinearLayout线性布局可以实现所有的基本的布局了，也是比较常用的布局方式之一  
LinearLayout设置了水平排列方式，水平方向的设置是无效的，如：left，right，center_horizontal（android:orientation="horizontal"）  
LinearLayout设置了垂直排列方式，垂直方向的设置是无效的，如：top，bottom，center_vertical（android:orientation="vertical"）  

### 二、FrameLayout
FramLayout布局是比较简单的一个布局对象，一个FramLayout对象会填充在当前屏幕左上角，下一个FramLayout对象会直接覆盖上一个元素，直接挡住上一个元素。  

## Activity
Android程序是运行在Activity上的，一个Activity相当于一个页面容器，运行在一个线程上，UI表现在Activity上。  
Activity需要在AndroidMainfest.xml里面声明，并设置其属性和intent-filter。  
Activity中常用的方法有：setContentView()，findViewById()，finish()，startActivity()等  
涉及生命周期的方法：onCreate(Bundle savedInstanceState)，onStart()，onRestart()，onResume()，onPause()，onStop()，onDestroy()等  
如新增一个MainActivity：  
1.在AndroidMainfest.xml声明MainActivity  
{% highlight java %}
<activity android:name=".activity.MainActivity" >
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
{% endhighlight %}  
2.编写布局文件：MainActivity.xml，如下整体采用线性布局  
{% highlight java %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    ......
    ......
</LinearLayout>
{% endhighlight %}
3.编写java代码，继承Activity，重写onCreate等方法  
{% highlight java %}
public class MainActivity extends Activity {
    // 省略...
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        initUI();
        initView();
        // bind service
        Intent intent = new Intent(this, MusicService.class);
        startService(intent);
        bindService(intent, mConnection, 0);
    }
    // 省略...
}
{% endhighlight %}  
>
## Service
Service主要用于处理一些耗时比较大的或者需要后台运行（保持运行）的逻辑。  
