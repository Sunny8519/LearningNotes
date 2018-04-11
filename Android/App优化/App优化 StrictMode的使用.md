## StrictMode的使用

```java
//ThreadPolicy:线程策略
StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
                .detectCustomSlowCalls() //开启
                .detectDiskReads() //开启对磁盘读取操作的监控
                .detectDiskWrites() //开启对磁盘写入操作的监控
                .detectNetwork() //开启对网络操作的监控
                .penaltyDialog() //触发违例后弹窗提醒
                .penaltyLog() //触发违例后打印到logcat
                .penaltyDropBox() //将违例信息记录到dropbox系统日志目录中(/data/system/dropbox)
                .penaltyFlashScreen() //触发违例后屏幕闪屏，有些设备可能没有这个功能
                .penaltyDeath() //触发违例后crash掉当前应用程序
                .penaltyDeathOnNetwork() //当网络操作违例时crash掉当前应用程序
                .permitDiskReads() //关闭对磁盘读取操作的监控
                .build());
//VmPolicy:虚拟机策略
StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
                .detectActivityLeaks() //检查Activity内存泄漏情况
                .detectLeakedClosableObjects() //检查资源有没有正确关闭
                .detectLeakedRegistrationObjects() //检查BroadcastReceiver或者ServiceConnect注册类对象有没有被正确释放
                .detectLeakedSqlLiteObjects() //检查SQLiteCursor或者其他SQLite对象是否被正确关闭
                .setClassInstanceLimit(CanvasPaintLearningActivity.class, 5) //设置某个类同时处于内存中的实例上限，可用于检测内存泄漏
                .penaltyLog()
                .build());
```

StrictMode一般在Application的onCreate()方法中初始化，可以用来监测应用的全局违例情况，也可以用在Activity的onCreate()方法中，具体的用法如上面所示代码，当然如果我们不想分的如上面那般细，可以使用`detectAll()`方法来监测所有情况。

使用StrictMode要注意在debug模式下使用，在release版本中应该把这项功能禁止掉。

StrictMode不能监控JNI层的磁盘IO和网络请求。

总结：ThreadPolicy可以说是对UI线程的监控，比如在UI线程中有耗时操作，ThreadPolicy就会提醒(以日志或者弹窗的形式)；VmPolicy可以说是对内存溢出的监控，比如资源没有释放。

参考文章：

[Android严苛模式StrictMode使用详解](https://blog.csdn.net/mynameishuangshuai/article/details/51742375)

[Android性能调优利器StrictMode](http://www.androidchina.net/4358.html)

[Android性能调优利器StrictMode](https://droidyue.com/blog/2015/09/26/android-tuning-tool-strictmode/)