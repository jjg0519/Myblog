# Java IO: 流

Java IO流是既可以从中读取，也可以写入到其中的数据流。正如这个系列教程之前提到过的，流通常会与数据源、数据流向目的地相关联，比如文件、网络等等。

流和数组不一样，不能通过索引读写数据。在流中，你也不能像数组那样前后移动读取数据，除非使用[RandomAccessFile](http://tutorials.jenkov.com/java-io/randomaccessfile.html) 处理文件。流仅仅只是一个连续的数据流。

某些类似[PushbackInputStream](http://tutorials.jenkov.com/java-io/pushbackinputstream.html) 流的实现允许你将数据重新推回到流中，以便重新读取。然而你只能把有限的数据推回流中，并且你不能像操作数组那样随意读取数据。流中的数据只能够顺序访问。

Java IO流通常是基于字节或者基于字符的。字节流通常以“stream”命名，比如InputStream和OutputStream。除了[DataInputStream](http://tutorials.jenkov.com/java-io/datainputstream.html) 和[DataOutputStream](http://tutorials.jenkov.com/java-io/dataoutputstream.html) 还能够读写int, long, float和double类型的值以外，其他流在一个操作时间内只能读取或者写入一个原始字节。

字符流通常以“Reader”或者“Writer”命名。字符流能够读写字符(比如Latin1或者Unicode字符)。可以浏览[Java Readers and Writers](http://tutorials.jenkov.com/java-io/readers-writers.html)获取更多关于字符流输入输出的信息。





## **InputStream**

java.io.InputStream类是所有Java IO输入流的基类。如果你正在开发一个从流中读取数据的组件，请尝试用InputStream替代任何它的子类(比如FileInputStream)进行开发。这么做能够让你的代码兼容任何类型而非某种确定类型的输入流。

然而仅仅依靠InputStream并不总是可行。如果你需要将读过的数据推回到流中，你必须使用PushbackInputStream，这意味着你的流变量只能是这个类型，否则在代码中就不能调用PushbackInputStream的unread()方法。

通常使用输入流中的read()方法读取数据。read()方法返回一个整数，代表了读取到的字节的内容(译者注：0 ~ 255)。当达到流末尾没有更多数据可以读取的时候，read()方法返回-1。

这是一个简单的示例：

```java
InputStream input = new FileInputStream("c:\\data\\input-file.txt");

int data = input.read();

while(data != -1){
  data = input.read();
}
```





## **OutputStream**

java.io.OutputStream是Java IO中所有输出流的基类。如果你正在开发一个能够将数据写入流中的组件，请尝试使用OutputStream替代它的所有子类。

这是一个简单的示例：

```java
OutputStream output = new FileOutputStream("c:\\data\\output-file.txt");
output.write("Hello World".getBytes());
output.close()
```



## **组合流**

你可以将流整合起来以便实现更高级的输入和输出操作。比如，一次读取一个字节是很慢的，所以可以从磁盘中一次读取一大块数据，然后从读到的数据块中获取字节。为了实现缓冲，可以把InputStream包装到BufferedInputStream中。代码示例：

````java
InputStream input = new BufferedInputStream(new FileInputStream("c:\\data\\input-file.txt"));
````

缓冲同样可以应用到OutputStream中。你可以实现将大块数据批量地写入到磁盘(或者相应的流)中，这个功能由BufferedOutputStream实现。

缓冲只是通过流整合实现的其中一个效果。你可以把InputStream包装到PushbackInputStream中，之后可以将读取过的数据推回到流中重新读取，在解析过程中有时候这样做很方便。或者，你可以将两个InputStream整合成一个[SequenceInputStream](http://tutorials.jenkov.com/java-io/sequenceinputstream.html)。

将不同的流整合到一个链中，可以实现更多种高级操作。通过编写包装了标准流的类，可以实现你想要的效果和过滤器。