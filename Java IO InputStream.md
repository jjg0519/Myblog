# Java IO InputStream

InputStream类是Java IO API中所有输入流的基类。InputStream子类包括FileInputStream，BufferedInputStream，PushbackInputStream等等。参考[Java IO概述](http://ifeve.com/java-io-3/)这一小节底部的表格，可以浏览完整的InputStream子类的列表。

## **Java InputStream例子**

InputStream用于读取基于字节的数据，一次读取一个字节，这是一个InputStream的例子：

```java
InputStream inputstream = new FileInputStream("c:\\data\\input-text.txt");

int data = inputstream.read();
while(data != -1) {
  //do something with data...
  doSomethingWithData(data);

  data = inputstream.read();
}
inputstream.close();
```



这个例子创建了FileInputStream实例。FileInputStream是InputStream的子类，所以可以把FileInputStream实例赋值给InputStream变量。

注意：为了清晰，代码忽略了一些必要的异常处理。想了解更多异常处理的信息，请参考Java IO异常处理。

从Java7开始，你可以使用“[try-with-resource](http://tutorials.jenkov.com/java-exception-handling/try-with-resources.html)”结构确保InputStream在结束使用之后关闭，链接指向了一篇关于“try-with-resource”是如何工作的文章，这里只是一个简单的例子：

```java
try( InputStream inputstream = new FileInputStream("file.txt") ) {

    int data = inputstream.read();
    while(data != -1){
        System.out.print((char) data);
        data = inputstream.read();
    }
}
```

InputStream的子类可能会包含read()方法的替代方法。比如，DataInputStream允许你利用readBoolean()，readDouble()等方法读取Java基本类型变量int，long，float，double和boolean。

## **流末尾**

如果read()方法返回-1，意味着程序已经读到了流的末尾，此时流内已经没有多余的数据可供读取了。-1是一个int类型，不是byte或者char类型，这是不一样的。

当达到流末尾时，你就可以关闭流了。

## **read(byte[])**

InputStream包含了2个从InputStream中读取数据并将数据存储到缓冲数组中的read()方法，他们分别是：

- int read(byte[])
- int read(byte, int offset, int length)

一次性读取一个字节数组的方式，比一次性读取一个字节的方式快的多，所以，尽可能使用这两个方法代替read()方法。

read(byte[])方法会尝试读取与给定字节数组容量一样大的字节数，返回值说明了已经读取过的字节数。如果InputStream内可读的数据不足以填满字节数组，那么数组剩余的部分将包含本次读取之前的数据。记得检查有多少数据实际被写入到了字节数组中。

read(byte, int offset, int length)方法同样将数据读取到字节数组中，不同的是，该方法从数组的offset位置开始，并且最多将length个字节写入到数组中。同样地，read(byte, int offset, int length)方法返回一个int变量，告诉你已经有多少字节已经被写入到字节数组中，所以请记得在读取数据前检查上一次调用read(byte, int offset, int length)的返回值。

这两个方法都会在读取到达到流末尾时返回-1。

这是一个使用InputStream的read(byte[])的例子：

```java
InputStream inputstream = new FileInputStream("c:\\data\\input-text.txt");

byte[] data      = new byte[1024];
int    bytesRead = inputstream.read(data);

while(bytesRead != -1) {
  doSomethingWithData(data, bytesRead);

  bytesRead = inputstream.read(data);
}
inputstream.close()
```

在代码中，首先创建了一个字节数组。然后声明一个叫做bytesRead的存储每次调用read(byte[])返回值的int变量，并且将第一次调用read(byte[])得到的返回值赋值给它。

在while循环内部，把字节数组和已读取字节数作为参数传递给doSomethingWithData方法然后执行调用。在循环的末尾，再次将数据写入到字节数组中。

你不需要想象出read(byte, int offset, int length)替代read(byte[])的场景，几乎可以在使用read(byte, int offset, int length)的任何地方使用read(byte[])。

## **输入流和数据源**

一个输入流往往会和数据源联系起来，比如文件，网络连接，管道等，更多细节已经在[Java IO概述](http://ifeve.com/java-io-3/)文章中介绍过了。