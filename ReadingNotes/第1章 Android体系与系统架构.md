## 第1章 体系与系统架构

### 1. Android系统架构

Android系统架构从上往底层说大致可以分为四层：

- 应用层，即我们的app，这是应用开发人员接触最多的一层；
- Framework层，应用程序中使用的四大组件属于Framework层，它是在Framework层定义，然后在应用层使用，另外就是我们应用层中使用的api都是Framework层，有时调用native层的代码也是通过Framework层的API来调用；
- 运行时库层，包含C/C++库，为Framework层提供底层功能支撑；
- Linux内核，Linux内核包含了Android系统的核心服务，包括硬件驱动，进程管理，安全系统等。

### 2. Dalvik和ART

Dalvik：每个app都会分配一个独立的Dalvik虚拟机实例来保证多个app之间相互不受干扰。Dalvik的特点就是运行时编译，Android应用程序打包完之后是.dex可执行文件，在应用程序运行时需要靠Dalvik把这些.dex可执行文件解释成机器能识别的二进制代码。

ART：在Android 5.X之后，ART模式已经取代了Dalvik，ART的特点就是在应用程序安装的时候通过预编译把.dex文件翻译成二进制文件存放于移动设备中，这样在应用程序运行时就不用编译了，因此它使得应用程序运行的速度更快，缺点是应用程序安装时需要耗费一定的时间，并且应用程序安装后的文件包更大。

### 3. 应用程序上下文对象

应用程序中的上下文共有三类：Application，Activity，Service，在创建Application，Activity，Service的时候都会创建相应的Context实现。

### 4.Android系统源代码目录和系统目录

Makefile机制：比如像Android这样的大型工程，它的源文件有很多，不同的功能，模块按照类型放置在不同的目录中，这些模块通常会有一个叫Makefile的文件来进行管理，Makefile文件定义了一系列的规则来指定模块和文件需要编译以及这些文件该按照怎样的顺序编译，甚至还可以定义编译规则，打包规则等。

在Android源代码目录中，每个最小功能单位的目录下都会有一个Makefile文件，这样每一级向上，通过一个个的Makefile文件把整个源代码联系在一起。

连接上手机，输入`adb shell`，然后再输入`ls`后我们能看到手机根目录里的一些文件，这些文件中我们比较关心的时system和data：

- /system/app/：存放系统App

- /system/bin/：存放Linux自带组件

- /system/build.prop：记录系统属性信息

- /system/fonts/：存放系统字体，手机在root后可下载TTF格式的字体替换原字体以达到修改系统字体的效果

- /system/framework/：系统核心文件，框架层

- /system/lib/：存放几乎所有的共享库(.so)文件

- /system/media/：存放系统提示音，系统铃声等，比如/system/media/audio/alarms目录中存放的是闹铃提醒，notification目录中存放的是短信或提示音，ringtones目录存放来电铃声，ui目录存放界面音效

- /system/usr/：保存用户的配置文件，如键盘布局，共享，时区文件等

- /data/app/：data目录包含了用户大部分的数据信息，/data/app/目录中包含了用户安装的app或者升级的app

  ```
  //这是/data/app/目录下某个应用包下的文件
  base.apk
  lib
  oat
  split_lib_dependencies_apk.apk
  split_lib_slice_0_apk.apk
  split_lib_slice_1_apk.apk
  split_lib_slice_2_apk.apk
  split_lib_slice_3_apk.apk
  split_lib_slice_4_apk.apk
  split_lib_slice_5_apk.apk
  split_lib_slice_6_apk.apk
  split_lib_slice_7_apk.apk
  split_lib_slice_8_apk.apk
  split_lib_slice_9_apk.apk
  ```

  - /data/data/：这个目录是开发者访问最多的目录，该目录下包含了app的数据信息，文件信息，数据库信息等，以包名的形式区分各个应用
  - /data/system/：包含手机的各项系统信息
  - /data/misc/：保存了大部分的WIFI，VPN信息

