## App优化 总结

### 一. 内存优化

#### 1. 节制使用Service

用Service来执行后台任务，应该在需要的时候才开启。因为启动一个Service，系统会倾向于将这个Service依赖的进程保留，导致系统在LRUCache中缓存的进程数减少，这样可能会导致切换别的应用程序时消耗更多的性能来创建进程。我们可以使用IntentService，在后台任务执行完之后能够自动停止。

#### 2. 界面不可见时释放内存

当用户离开了应用程序，应用程序的界面已经不可见了，我们可以把跟界面相关的资源释放。重写Activity的onTrimMemory(int)方法，在该方法中监听TRIM_MEMORY_UI_HIDDEN这个级别，一旦触发该级别说明用户离开了该页面，我们就可以做一些相关的资源释放操作。

TRIM_MEMORY_UI_HIDDEN是接口ComponentCallbacks2中的常量，该接口中还有其他类似的常量，可在不同的情况下监听这些常量。

#### 3. 避免加载不需要的分辨率的Bitmap

压缩图片。[Bitmap压缩策略](http://sunny8519.top/2018/03/31/Bitmap%E5%8E%8B%E7%BC%A9%E7%AD%96%E7%95%A5/)

#### 4. 使用优化过的数据集合

在Android中我们可以使用诸如SparseArray、SparseBooleanArray、LongSparseArray等经过优化的集合。

#### 5. 开发者应该知晓内存开支情况

- 使用枚举会比静态常量消耗两倍以上的内存；
- 任何一个Java类，包括匿名类、内部类，都要占用大约500字节的内存空间；
- 任何一个类的实例会占用12~16字节的内存，尽量避免频繁创建对象实例；

### 二. 代码优化

#### 1. 避免在循环条件中使用复杂的表达式

复杂的表达式能从循环中抽取出来的尽量抽取出来，比如：

```java
public class Test{
  public static void main(String[] args){
    for(int i = 0;i < list.size(); i++){
      ...
    }
  }
}

//改进
public class Test{
  public static void main(String[] args){
    int size = list.size();
    for(int i = 0;i < size; i++){
      ...
    }
  }
}
//或者
public class Test{
  public static void main(String[] args){
    for(int i = 0,size = list.size();i < size; i++){
      ...
    }
  }
}
```

#### 2. 类中的getter和setter方法如果能使用final修饰尽量使用final修饰

#### 3. 将try/catch块移出循环

#### 4. boolean值不要用==判断

比如：

```java
public boolean foo(String arg){
  return "a".equals(arg) == true;
}
//改进
public boolean foo(String arg){
  return "a".equals(arg); //没必要多此一举
}
```

#### 5. 尽量使用条件操作符代替if/else

#### 6. 尽量不要在循环体重实例化变量

#### 7. 尽量使用局部变量

#### 8. 尽量使用final修饰符

#### 9. 采用需要时再创建策略

参考书籍：

《Java程序性能优化》