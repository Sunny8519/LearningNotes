## Kotlin之基础语法

#### 1. 空类型安全

```java
fun getName():String?{
    return null;
}

fun main(args:Array<String>){
    val name:String = getName() ?: return //如果getName()方法返回null，则return，如果不是null，把返回值赋值给name变量
    println(name.length)
        
    val value:String? = "hello,world"
    println(value!!.length) //两个感叹号表示我们知道这个value值不是null，可以直接.length，如果不加!!编译器会认为这个字段可能为null，会发生NPE，从而编译不过
}

val notNull:String = null //错误，不能为null
notNull.length //正确，不可空的值可以直接使用
val nullable:String? = null //正确，可以为null
nullable.length //错误，可能为空，不能直接获取长度
nullable!!.length //正确，强制认定nullable不可空
nullable?.length //正确，若nullable为空，则返回空
```

#### 2. 智能类型转换

```java
fun main(args:Array<String>){
    val parent:Parent = Child()
    if(parent is Child){
        println(parent.name) //此处不用再强转成Child类型，如果再Java中，前面使用instanceOf关键字判断后在这里还要再强转成Child才能使用Child中的属性和方法
    }
}

fun main(args:Array<String>){
    val parent:Parent: = Parent()
    val child:Child? = parent as Child //如果这样转，在运行的时候会报类型转换异常
        
    val child:Child? = parent as? Child //在as后面加个?后就表示如果parent转换成Child类型失败就给child赋值为null，因此这里不会抛出类型转换异常
}
```

#### 3. 区间

ClosedRange的子类，IntRange最常用

基本写法：
0..100 表示 [0,100]
0 until 100 表示 [0,100)
i in 0..100 判断i是否在区间[0,100]中

#### 4. 数组

基本写法：

```
val array:Array<String> = arrayOf(...)
```

基本类型的数组：

Kotlin为了避免基本数据类型的装箱拆箱的开销封装了定制的基本数据类型的数组，比如IntArray，CharArray等等。

#### 5. 常量与变量

Java中用final修饰的变量与不用final修饰的变量的区别：

```java
public class JavaMain {
    public final String FINAL_HELLOW_WORLD = "HelloWorld";
    public String helloWorld = FINAL_HELLOW_WORLD;
}

//上面那段代码转为字节码转为字节码
aload 0
ldc "HelloWorld"
putfield 'JavaMain.FINAL_HELLOW_WORLD','Ljava/lang/String;'
aload 0
ldc "HelloWorld" //这里使用的是"HelloWorld"这个值
putfield 'JavaMain.helloWorld','Ljava/lang/String;'
    
//我们把final去掉之后再看看字节码
public class JavaMain {
    public String FINAL_HELLOW_WORLD = "HelloWorld";
    public String helloWorld = FINAL_HELLOW_WORLD;
}

aload 0
ldc "HelloWorld"
putfield 'JavaMain.FINAL_HELLOW_WORLD','Ljava/lang/String;'
aload 0
aload 0
getfield 'JavaMain.FINAL_HELLOW_WORLD','Ljava/lang/String;' //这里先把FINAL_HELLOW_WORLD变量的值先取出来再赋值给helloWorld变量
putfield 'JavaMain.helloWorld','Ljava/lang/String;'
```

在Java中这种用final修饰的变量叫做编译期常量，很明显用final修饰的变量比不用final修饰的变量效率更高。

在Kotlin中有val和var的区分，val表示常量，var表示变量，那么val与Java中的final区别在哪儿呢？我们看下面代码：

```java
val FINAL_HELLO_WORLD: String = "HelloWorld"
var helloWorld: String = FINAL_HELLO_WORLD

//我们把这两行代码转换成字节码
ldc "HelloWorld"
putstatic 'MainKt.FINAL_HELLO_WORLD','Ljava/lang/String;'
getstatic 'MainKt.FINAL_HELLO_WORLD','Ljava/lang/String;' //多了一步get操作，这和没加final修饰的Java变量是一样的，因此val不能完全等价于Java中的final
putstatic 'MainKt.helloWorld','Ljava/lang/String;'

//想要Java中的final效果怎么办呢？加个const关键字吧
const val FINAL_HELLO_WORLD: String = "HelloWorld"
var helloWorld: String = FINAL_HELLO_WORLD

//编译成字节码
ldc "HelloWorld" //直接把"HelloWordl"字符串赋值给helloWorld变量，这就和final的效果一样了
putstatic 'MainKt.helloWorld','Ljava/lang/String;'
```

