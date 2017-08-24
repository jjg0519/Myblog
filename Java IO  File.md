# Java IO: File

Java IO API中的FIle类可以让你访问底层文件系统，通过File类，你可以做到以下几点：

- 检测文件是否存在
- 读取文件长度
- 重命名或移动文件
- 删除文件
- 检测某个路径是文件还是目录
- 读取目录中的文件列表

请注意：File只能访问文件以及文件系统的元数据。如果你想读写文件内容，需要使用FileInputStream、FileOutputStream或者RandomAccessFile。如果你正在使用Java NIO，并且想使用完整的NIO解决方案，你会使用到java.nio.FileChannel(否则你也可以使用File)。

## **实例化一个java.io.File对象**

在使用File之前，必须拥有一个File对象，这是实例化的代码例子：

```java
File file = new File("c:\\data\\input-file.txt");
```

很简单，对吗？File类同样拥有多种不同实例化方式的构造函数。

## **检测文件是否存在**

当你获得一个File对象之后，可以检测相应的文件是否存在。当文件不存在的时候，构造函数并不会执行失败。你已经准备好创建一个File了，对吧？

通过调用exists()方法，可以检测文件是否存在，代码如下：

```java
File file = new File("c:\\data\\input-file.txt");
boolean fileExists = file.exists();
```

## **文件长度**

通过调用length()可以获得文件的字节长度，代码如下：

```java
File file = new File("c:\\data\\input-file.txt");

long length = file.length();
```

## **重命名或移动文件**

通过调用File类中的renameTo()方法可以重命名(或者移动)文件，代码如下：

```java
File file = new File("c:\\data\\input-file.txt");

boolean success = file.renameTo(new File("c:\\data\\new-file.txt"));
```



## **删除文件**

通过调用delete()方法可以删除文件，代码如下：

```java
File file = new File("c:\\data\\input-file.txt");

boolean success = file.delete();
```





delete()方法与rename()方法一样，返回布尔值表明是否成功删除文件，同样也会有相同的操作失败原因。

## **检测某个路径是文件还是目录**

File对象既可以指向一个文件，也可以指向一个目录。可以通过调用isDirectory()方法，可以判断当前File对象指向的是文件还是目录。当方法返回值是true时，File指向的是目录，否则指向的是文件，代码如下：

```java
File file = new File("c:\\data");

boolean isDirectory = file.isDirectory();
```



## **读取目录中的文件列表**

你可以通过调用list()或者listFiles()方法获取一个目录中的所有文件列表。list()方法返回当前File对象指向的目录中所有文件与子目录的字符串名称(译者注：不会返回子目录下的文件及其子目录名称)。listFiles()方法返回当前File对象指向的目录中所有文件与子目录相关联的File对象(译者注：与list()方法类似，不会返回子目录下的文件及其子目录)。代码如下：

```java
File file = new File("c:\\data");

String[] fileNames = file.list();

File[]   files = file.listFiles();
```

