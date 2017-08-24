# Java IO: PipedInputStream

PipedInputStream可以从管道中读取字节流数据，代码如下：

```java

InputStream input = new PipedInputStream(pipedOutputStream);

int data = input.read();
while(data != -1) {
  //do something with data...
  doSomethingWithData(data);

  data = input.read();
}
input.close();
```

请注意，为了清晰，这里忽略了必要的异常处理。想了解更多异常处理的信息，请参考[Java IO异常处理](http://ifeve.com/java-io-%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86/)。

PipedInputStream的read()方法返回读取到的包含一个字节内容的int变量(译者注：0~255)。如果read()方法返回-1，意味着程序已经读到了流的末尾，此时流内已经没有多余的数据可供读取了，你可以关闭流。-1是一个int类型，不是byte类型，这是不一样的。

 

## **Java IO管道**

正如你所看到的例子那样，一个PipedInputStream需要与一个PipedOutputStream相关联，当这两种流联系起来时，就形成了一条管道。要想更多地了解Java IO中的管道，请参考[Java IO管道](http://ifeve.com/java-io-%E7%AE%A1%E9%81%93/)。