# Java IO: PipedOutputStream

PipedOutputStream可以往管道里写入读取字节流数据，代码如下：

```java

OutputStream output = new PipedOutputStream(pipedInputStream);

while(moreData) {
  int data = getMoreData();
  output.write(data);
}
output.close();
```



请注意，为了清晰，这里忽略了必要的异常处理。想了解更多异常处理的信息，请参考[Java IO异常处理](http://ifeve.com/java-io-%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86/)。

PipedOutputStream的write()方法取一个包含了待写入字节的int类型变量作为参数进行写入。

## **Java IO管道**

一个PipedOutputStream总是需要与一个PipedInputStream相关联。当这两种流联系起来时，它们就形成了一条管道。要想更多地了解Java IO中的管道，请参考[Java IO管道](http://ifeve.com/java-io-%E7%AE%A1%E9%81%93/)。

