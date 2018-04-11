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





参考文章：

[Android严苛模式StrictMode使用详解](https://blog.csdn.net/mynameishuangshuai/article/details/51742375)

[Android性能调优利器StrictMode](http://www.androidchina.net/4358.html)

[Android性能调优利器StrictMode](https://droidyue.com/blog/2015/09/26/android-tuning-tool-strictmode/)