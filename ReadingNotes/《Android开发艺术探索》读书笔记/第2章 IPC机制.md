## 第2章 IPC机制

### 2. IPC基础概念介绍

#### 2.1 Serializable接口

Serializable接口是一个空接口，它提供了一种标准，意思就是实现了这个接口的bean类是可以被序列化和反序列化的。Serializable是Java自带的序列化和反序列化机制，实现起来也相对简单，大致分为两步：

- bean类实现Serializable接口(implements)；
- 指定serialVersionUID；

其实serialVersionUID可以不用指定，但是指定serialVersionUID可以最大限度的增加反序列化的成功率

https://www.ibm.com/developerworks/cn/java/j-lo-serial/

https://blog.csdn.net/dreamtdp/article/details/15378329

https://www.cnblogs.com/huhx/p/sSerializableTheory.html

http://longdick.iteye.com/blog/458557