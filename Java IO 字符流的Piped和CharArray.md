# Java IO: 字符流的Piped和CharArray

本章节将简要介绍管道与字符数组相关的reader和writer，主要涉及PipedReader、PipedWriter、CharArrayReader、CharArrayWriter。

## PipedReader

PipedReader能够从管道中读取字符流。与PipedInputStream类似，不同的是PipedReader读取的是字符而非字节。换句话说，PipedReader用于读取管道中的文本。代码如下：

```java
PipedWriter pipedWriter = new PipedWriter();
PipedReader pipedReader = new PipedReader(pipedWriter);

int data = pipedReader.read();
while(data != -1) {
  //do something with data...
  doSomethingWithData(data);

  data = pipedReader.read();
}
pipedReader.close();
```

注意：为了清晰，代码忽略了一些必要的异常处理。想了解更多异常处理的信息，请参考[Java IO异常处理](http://ifeve.com/java-io-%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86/)。

read()方法返回一个包含了读取到的字符内容的int类型变量(译者注：0~65535)。如果方法返回-1，表明PipedReader中已经没有剩余可读取字符，此时可以关闭PipedReader。-1是一个int类型，不是byte或者char类型，这是不一样的。

正如你所看到的例子那样，一个PipedReader需要与一个PipedWriter相关联，当这两种流联系起来时，就形成了一条管道。要想更多地了解Java IO中的管道，请参考[Java IO管道](http://ifeve.com/java-io-%E7%AE%A1%E9%81%93/)。

## PipedWriter

PipedWriter能够往管道中写入字符流。与PipedOutputStream类似，不同的是PipedWriter处理的是字符而非字节，PipedWriter用于写入文本数据。代码如下：

````java
PipedWriter pipedWriter = new PipedWriter();

while(moreData()) {
  int data = getMoreData();
  pipedWriter.write(data);
}
pipedWriter.close();
````

PipedWriter的write()方法取一个包含了待写入字节的int类型变量作为参数进行写入，同时也有采用字符串、字符数组作为参数的write()方法。

## CharArrayReader

CharArrayReader能够让你从字符数组中读取字符流。代码如下：

```java
char[] chars = "123".toCharArray();

CharArrayReader charArrayReader =
    new CharArrayReader(chars);

int data = charArrayReader.read();
while(data != -1) {
  //do something with data

  data = charArrayReader.read();
}

charArrayReader.close();
```

如果数据的存储媒介是字符数组，CharArrayReader可以很方便的读取到你想要的数据。CharArrayReader会包含一个字符数组，然后将字符数组转换成字符流。(译者注：CharArrayReader有2个构造函数，一个是CharArrayReader(char[] buf)，将整个字符数组创建成一个字符流。另外一个是CharArrayReader(char[] buf, int offset, int length)，把buf从offset开始，length个字符创建成一个字符流。更多细节请参考[Java官方文档](http://docs.oracle.com/javase/7/docs/api/))



## CharArrayWriter

CharArrayWriter能够把字符写入到字符输出流writer中，并且能够将写入的字符转换成字符数组。代码如下：

```java
CharArrayWriter charArrayWriter = new CharArrayWriter();

charArrayWriter.write("CharArrayWriter");

char[] chars1 = charArrayWriter.toCharArray();

charArrayWriter.close();
```



当你需要以字符数组的形式访问写入到writer中的字符流数据时，CharArrayWriter是个不错的选择。