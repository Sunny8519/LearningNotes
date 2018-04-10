## ANR详解

#### 1. 什么是ANR?

ANR全名叫做Application Not Responding，即应用程序无响应。当系统在一定时间内无法处理某个操作时，系统会弹出“应用程序无响应”的弹窗，这就意味着App中出现了ANR异常。出现ANR异常一般会是以下几种情况导致的，所以在排除ANR异常的时候可以依据以下情况：1.Activity或者Fragment中耗时操作超过5秒；2.Broadcast Receiver的onReceive()中耗时操作超过10秒；3.Service中耗时操作超过20秒。其实上面三种情况的根源还是主线程中做了太多阻塞耗时操作，比如数据库操作，文件读写或者网络查询等。

#### 2. 如何避免ANR?

避免ANR只有一条原则：**不要在UI线程中做过多耗时操作，耗时操作应该放在另外的工作线程中来进行。**

#### 3. ANR的分析

我们可以通过获取ANR发生时产生的trace.txt文件来分析。trace.txt文件位于/data/anr/文件夹下。移动设备在没有root的情况下，该文件夹是不可见的，但是我们可以通过下面adb命令把traces.txt文件拷贝出来：`adb pull /data/anr/traces.txt .`，这样traces.txt文件会被拷贝到项目的根目录下。

以下是在主线程中因为sleep一段时间产生的ANR日志记录：

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

traces.txt文件最上面是最新产生的ANR

待验证：

1. Activity或者Fragment中耗时操作超过5秒，Broadcast Receiver的onReceive()中耗时操作超过10秒，Service中耗时操作超过20秒会导致ANR