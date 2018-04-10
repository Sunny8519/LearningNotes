## ANR详解

#### 1. 什么是ANR?

ANR全名叫做Application Not Responding，即应用程序无响应。当系统在一定时间内无法处理某个操作时，系统会弹出“应用程序无响应”的弹窗，这就意味着App中出现了ANR异常。出现ANR异常一般会是以下几种情况导致的，所以在排除ANR异常的时候可以依据以下情况：1.Activity或者Fragment中耗时操作超过5秒；2.Broadcast Receiver的onReceive()中耗时操作超过10秒；3.Service中耗时操作超过20秒。其实上面三种情况的根源还是主线程中做了太多阻塞耗时操作，比如数据库操作，文件读写或者网络查询等。

#### 2. 如何避免ANR?

避免ANR只有一条原则：**不要在UI线程中做过多耗时操作，耗时操作应该放在另外的工作线程中来进行。**

#### 3. ANR的分析

我们可以通过获取ANR发生时产生的trace.txt文件来分析。trace.txt文件位于/data/anr/文件夹下。移动设备在没有root的情况下，该文件夹是不可见的，但是我们可以通过下面adb命令把traces.txt文件拷贝出来：`adb pull /data/anr/traces.txt .`，这样traces.txt文件会被拷贝到项目的根目录下。

以下是在**主线程**中因为sleep一段时间产生的ANR日志记录：

```java
----- pid 15030 at 2018-03-30 10:01:27 -----//ANR产生时间
Cmd line: logkit.cnmts.com //最新的ANR发生的进程名(一般应用包名)
... //省略
suspend all histogram:	Sum: 138us 99% C.I. 2us-23us Avg: 7.666us Max: 23us
DALVIK THREADS (14):
"Signal Catcher" daemon prio=5 tid=3 Runnable
  | group="system" sCount=0 dsCount=0 obj=0x12c53ca0 self=0x7a2e2a2400
  | sysTid=15036 nice=0 cgrp=default sched=0/0 handle=0x7a2d708450
  | state=R schedstat=( 21444270 75001 27 ) utm=1 stm=1 core=2 HZ=100
  | stack=0x7a2d60e000-0x7a2d610000 stackSize=1005KB
  | held mutexes= "mutator lock"(shared held)
  (no managed stack frames)

"main" prio=5 tid=1 Sleeping //这就是事发线程
  | group="main" sCount=1 dsCount=0 obj=0x7773ddd0 self=0x7a2e2a1a00
  | sysTid=15030 nice=-10 cgrp=default sched=0/0 handle=0x7a3223da98
  | state=S schedstat=( 428584375 30208 297 ) utm=35 stm=7 core=4 HZ=100
  | stack=0x7ff2851000-0x7ff2853000 stackSize=8MB
  | held mutexes=
  at java.lang.Thread.sleep!(Native method)
  - sleeping on <0x09aed269> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:379)
  - locked <0x09aed269> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:321)
  at logkit.cnmts.com.crash.CrashHandler.uncaughtException(CrashHandler.java:76) //这是事发地点
  at java.lang.ThreadGroup.uncaughtException(ThreadGroup.java:1068)
  at java.lang.ThreadGroup.uncaughtException(ThreadGroup.java:1063)
  ... //省略
```

traces.txt文件中最上面是最新产生的ANR日志记录。

如果在UI线程没有做耗时操作但还是发生ANR，那么就可能是CPU满负荷或者内存快满了，当然内存快满了更多的情况是产生OOM异常。

**CPU满负荷**时的traces信息：

```java
Process:com.anly.githubapp
...
CPU usage from 3330ms to 814ms ago:
6% 178/system_server: 3.5% user + 1.4% kernel / faults: 86 minor 20 major
4.6% 2976/com.anly.githubapp: 0.7% user + 3.7% kernel /faults: 52 minor 19 major
0.9% 252/com.android.systemui: 0.9% user + 0% kernel
...

100%TOTAL: 5.9% user + 4.1% kernel + 89% iowait //这句话表明了CPU的89%被IO操作占用了
```

如果发现这类情况，去分析方法的调用栈，一般会发现有频繁的文件读写操作或者数据库操作放到主线程来做了，因为CPU的89%都被IO操作占用了。

**内存原因：**

```java
Cmdline: android.process.acore

DALVIK THREADS:
"main"prio=5 tid=3 VMWAIT
|group="main" sCount=1 dsCount=0 s=N obj=0x40026240self=0xbda8
| sysTid=1815 nice=0 sched=0/0 cgrp=unknownhandle=-1344001376
atdalvik.system.VMRuntime.trackExternalAllocation(NativeMethod)
atandroid.graphics.Bitmap.nativeCreate(Native Method)
atandroid.graphics.Bitmap.createBitmap(Bitmap.java:468)
atandroid.view.View.buildDrawingCache(View.java:6324)
atandroid.view.View.getDrawingCache(View.java:6178) //方法调用栈，从这可以分析出ANR原因
...

MEMINFO in pid 1360 [android.process.acore] **
native dalvik other total
size: 17036 23111 N/A 40147
allocated: 16484 20675 N/A 37159
free: 296 2436 N/A 2732 //从这能看出内存所剩无几
```

但是这种情况大多数会导致OOM异常。

#### 4. ANR的处理

针对上面三种情况的解决方案分别如下：

- UI线程阻塞：开启工作线程来处理耗时任务；
- CPU满负荷：比如CPU长时间被IO占用，可以通过开启工作线程来异步执行；
- 内存不够：增大VM内存，使用largeHeap属性；排查代码中的内存泄漏等。

#### 5. 启用工作线程的方式

- 继承Thread，实现run方法；
- 实现Runnable或者Callable接口；
- AsyncTask；
- HandlerThread；
- IntentService；
- Loader，常用的有CursorLoader，用于加载数据库数据。

**注意：**当在使用Thread或者HandlerThread的时候应该把线程的优先级调低点，因为默认情况下Thread的优先级和UI线程是一样的，CPU在调度时还是有可能阻塞UI线程。

待验证：

1. Activity或者Fragment中耗时操作超过5秒，Broadcast Receiver的onReceive()中耗时操作超过10秒，Service中耗时操作超过20秒会导致ANR

参考文章：[Android App优化, 要怎么做?](https://www.jianshu.com/p/f7006ab64da7)