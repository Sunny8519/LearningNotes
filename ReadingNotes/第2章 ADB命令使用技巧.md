## 第2章 ADB命令使用技巧

#### 1. ADB基础

ADB英文全称为Android Debug Bridge。

ADB工具位于SDK中的platform-tools目录下，如果我们要使用ADB命令需要先进入到该目录下，或者把platform-tools路径添加到系统环境变量中，这样在控制台可以直接使用。

#### 2. ADB常用命令

##### 查看系统盘符

```
D:\xg-android-pad>adb shell df
Filesystem               Size     Used     Free   Blksize
/mnvm2:0                 6.5M     1.1M     5.4M   1024
/modem_log               7.8M   600.0K     7.2M   4096
/dev                     1.4G    88.0K     1.4G   4096
/sys/fs/cgroup           1.4G    12.0K     1.4G   4096
/mnt/asec                1.4G     0.0K     1.4G   4096
/mnt/obb                 1.4G     0.0K     1.4G   4096
/cache                 248.0M   464.0K   247.5M   4096
/system                  2.4G     1.9G   495.1M   4096
/data                   10.4G     7.1G     3.3G   4096
/cust                  495.9M   111.5M   384.5M   4096
/splash2: Permission denied
/3rdmodem               59.0M     4.8M    54.2M   1024
/3rdmodemnvm: Permission denied
/3rdmodemnvmbkp: Permission denied
/data/sec_storage: Permission denied
/mnt/shell/emulated     10.4G     7.1G     3.3G   4096
```

##### 查看Log

```
//打印所有Log
D:\xg-android-pad>adb shell
shell@HWMozart:/ $ logcat 
```

这里面打印出来的日志和Android studio的Logcat中的是一样的，建议直接使用Android studio中的Logcat，里面的日志会根据等级以不同的颜色显示。

##### 输出所有已安装的应用

```
D:\xg-android-pad>adb shell pm list packages -f
```

##### 模拟按键输入

```
D:\xg-android-pad>adb shell input keyevent 3
最后面的数字是要执行的Keyevent的code

常用Code
input keyevent 82 -- menu
input keyevent 3 -- home 
input keyevent 19 -- up
input keyevent 20 -- down
input keyevent 21 -- left
input keyevent 22 -- right
input keyevent 66 -- enter
input keyevent 4 -- back
```

##### 模拟滑动输入

```
D:\xg-android-pad>adb shell input touchscreen swipe 18 665 200 665
```

上面这行命令模拟的是手指向右滑动

##### 查看运行状态

```
D:\xg-android-pad>adb shell dumpsys activity [activities/services...]
或者
D:\xg-android-pad>adb shell
shell@HWMozart:/ $ dumpsys activity [activities/services...]
```

上面这两种方式都可以查看运行状态，比如现在想要查看app中Activity的运行状态以及任务栈情况，可以输入以下命令：

```
D:\xg-android-pad>adb shell
shell@HWMozart:/ $ dumpsys activity activities [| grep "应用程序包名"]
```

中括号中的命令是可有可无的，它的作用主要是把指定应用程序中的activity筛选出来。

##### 录制屏幕

```
D:\xg-android-pad>adb shell screenrecord /sdcard/demo.mp4
```

后面的路径是移动设备上的存储路径

##### 重启移动设备

```
D:\xg-android-pad>adb reboot
```

