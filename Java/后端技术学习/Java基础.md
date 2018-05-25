## 一. File类

### 1. 创建文件

```java
//File类
public boolean createNewFile() throws IOException
```

该方法只能创建文件，而不能创建文件夹，如果创建文件成功返回true，创建文件失败返回false(比如创建的文件已存在)。创建文件的路径已经在File对象中已经给出。

### 2. 创建文件夹

```java
//File类
public boolean mkdir()
```

创建单层文件夹。

```java
//File类
public boolean mkdirs()
```

创建单层或者多层文件夹，推荐使用该方法创建文件夹。

创建的文件夹路径在File对象中已经给出。

### 3. 删除文件或者文件夹

```java
//File类
public boolean delete()
```

该方法能删除文件或者文件夹，删除成功返回true，删除失败返回false。删除的文件或者文件夹路径已经在File对象中给出。另：使用此方法删除的Windows文件是不会经过回收站的，直接从硬盘删除，使用不小心的话，删除的文件或者文件夹就找不回来了。

### 4. File类的获取功能

```java
//File类
public String[] list()
```

该方法可获取指定路径下的所有文件和文件夹(只有文件和文件夹的名字，不包括父路径)，包括隐藏或者其他一些看不见的文件。

```java
//File类
public File[] listFiles()
```

该方法可获取制定路径下的所有文件和文件夹的File对象，从File对象中可以获得文件或者文件夹的完整路径名。

```java
//File类
public static File[] listRoots()
```

该方法是静态的，获取系统的所有根目录。

### 5. 文件过滤

```java
public File[] listFiles(FileFilter filter)

@FunctionalInterface
public interface FileFilter {

    /**
     * Tests whether or not the specified abstract pathname should be
     * included in a pathname list.
     *
     * @param  pathname  The abstract pathname to be tested
     * @return  <code>true</code> if and only if <code>pathname</code>
     *          should be included
     */
    boolean accept(File pathname);
}
```

FileFilter是一个接口，该接口的功能就是实现文件的筛选条件，因此，我们在调用listFiles(FileFilter)方法的时候需要自己实现一个FileFilter对象。

视频观看到13