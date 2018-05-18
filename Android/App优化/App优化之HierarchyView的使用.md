## App优化之HierarchyView

### 1. Android Studio中的使用步骤

开启HierarchyView需要如下条件：

- 4.0及以下，没有root，需要使用[开源工程ViewServer](https://github.com/romainguy/ViewServer)
- 4.0及以下，有root，无需其他额外的配置
- 4.1及以上，需要在PC端设置ANDROID_HVPROTO环境变量

```java
ANDROID_HVPROTO = ddm
```

在Android Studio中通过以下步骤即可打开HierarchyView：

![android_monitor.png](https://upload-images.jianshu.io/upload_images/5231076-b7e44188612dcc19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

开启这个“Android Device Monitor”之前需要把项目在手机上运行起来。

然后再点击下面两个按钮中的一个就进入到HierarchyView页面了：

![HierarchyView1.png](https://upload-images.jianshu.io/upload_images/5231076-b3fc6d5a2c542400.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是第一次进入的时候可能什么都没有，我们可以点击下面的按钮来生成Tree View：

![HierarchyView2.png](https://upload-images.jianshu.io/upload_images/5231076-064da8e874eeb13a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是生成的Tree View可能是没有性能指标的(也就是没有红黄绿点)，我们可以先选中一个控件，然后点击“Profile Node"来描述一下节点就可以生成性能指标了：

![HierarchyView3.png](https://upload-images.jianshu.io/upload_images/5231076-f69646e5195d81a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这三个有颜色的点从左到右表示的是Measure,Layout和Draw的性能：

- 绿色，表示该View的此项性能比该View Tree中超过50%的View都要快
- 黄色，表示该View的此项性能比该View Tree中超过50%的View都要慢
- 红色，表示该View的此项性能是View Tree中最慢的

下面列出一些红点可能代表的性能问题：

- Measure红点：可能是布局中嵌套RelativeLayout，或者嵌套的LinearLayout都使用了weight属性；
- Layout红点：可能是这个布局以下的层级嵌套太深；
- Draw红点：可能是自定义View的绘制有问题，比如复杂的计算等。