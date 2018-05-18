### 1. App启动方式

App启动分为三种：冷启动，热启动，温启动

冷启动是指App没有启动过或者App进程被系统killed，这样系统中是不存在该App进程的，这时启动App就是冷启动。因此冷启动需要经历App启动流程的全过程(创建App进程，加载相关资源，启动main thread等等)。

热启动是指App进程处于后台，启动App时只是把后台的App进程置为前台进程，比如通过home键退出App，然后通过点击App图标重新进入App。

温启动是介于冷启动和热启动之间，一般会在以下两种情况下发生：

- 用户通过back键退出了App，然后又启动了App，App进程可能还在运行，但是Activity需要重新创建；
- 用户退出App后，系统由于内存紧张把App killed了，那么App的进程和activity都会被销毁，这时重新启动App，系统就需要重新创建App进程以及恢复activity。

### 2. 阻塞App快速启动的地方

- Application的onCreate()方法：有时，我们应用程序中集成了很多第三方服务，这些服务都要求在Application的onCreate()中初始化，如果这些第三方服务的初始化过程很缓慢，就会导致我们的App启动缓慢，从而出现白屏或者黑屏(白屏还是黑屏依据App主题)。
- 首屏Activity的渲染：如果首屏Activity界面复杂，那么App启动进入首屏Activity时渲染界面的用时就会较长，也会出现白屏或者黑屏。

针对Application的onCreate()方法耗时的优化方案是把一些第三方服务或者其他一些耗时操作放入工作线程中执行，比如开启一个IntentService。如果有些操作非得放在主线程中执行的话，我们可以添加一个SplashActivity页面为启动屏，把这些操作放入SplashActivity，该页面没有setContentView()，仅提供一个具有背景的Theme，这样就能减少SplashActivity的界面渲染时间了。

比如：

```java
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 底层白色 -->
    <item android:drawable="@color/white" />
    
    <!-- 顶层Logo居中 -->
    <item>
        <bitmap
            android:gravity="center"
            android:src="@drawable/ic_github" />
    </item>
</layer-list>

//自定义主题
<style name="SplashTheme" parent="AppTheme">
    <item name="android:windowBackground">@drawable/logo_splash</item>
</style>

public class LogoSplashActivity extends BaseActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        // 注意, 这里并没有setContentView, 单纯只是用来跳转到相应的Activity.
        // 目的是减少首屏渲染
        
        if (AppPref.isFirstRunning(this)) {
            IntroduceActivity.launch(this);
        }
        else {
            MainActivity.launch(this);
        }
        finish();
    }
}

<activity
  android:name=".ui.module.main.LogoSplashActivity"
  android:screenOrientation="portrait"
  android:theme="@style/SplashTheme">
  <intent-filter>
      <action android:name="android.intent.action.MAIN"/>
      <category android:name="android.intent.category.LAUNCHER"/>
  </intent-filter>
</activity>
```

