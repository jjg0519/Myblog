# Java IO  OutputStream

OutputStream类是Java IO API中所有输出流的基类。子类包括BufferedOutputStream，FileOutputStream等等。参考[Java IO概述](http://ifeve.com/java-io-3/)这一小节底部的表格，可以浏览完整的子类的列表。

## **输出流和目标媒介**

输出流往往和某些数据的目标媒介相关联，比如文件，网络连接，管道等。更多细节请参考[Java IO概述](http://ifeve.com/java-io-3/)。当写入到输出流的数据逐渐输出完毕时，目标媒介是所有数据的归属地。

## **Write(byte)**

write(byte)方法用于把单个字节写入到输出流中。OutputStream的write(byte)方法将一个包含了待写入数据的int变量作为参数进行写入。只有int类型的第一个字节会被写入，其余位会被忽略。(译者注：写入低8位，忽略高24位)。

OutputStream的子类可能会包含write()方法的替代方法。比如，DataOutputStream允许你利用writeBoolean()，writeDouble()等方法将基本类型int，long，float，double，boolean等变量写入。

这是一个OutputStream的write()方法例子：

```java
OutputStream output = new FileOutputStream("c:\\data\\output-text.txt");

while(hasMoreData()) {
  int data = getMoreData();
  output.write(data);
}
output.close();
```

这个例子首先创建了待写入的FileOutputStream。在进入while循环之后，循环的判断条件是hasMoreData()方法的返回值。hasMoreData()方法的实现不予展示，请把这个函数理解为：当有剩余可写数据时，返回true，否则返回false。

请注意，为了清晰，这里忽略了必要的异常处理。想了解更多异常处理的信息，请参考[Java IO异常处理](http://ifeve.com/java-io-%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86/)。

## **write(byte[])**

OutputStream同样包含了将字节数据中全部或者部分数据写入到输出流中的方法，分别是write(byte[])和write(byte[], int offset, int length)。

write(byte[])把字节数组中所有数据写入到输出流中。

write(byte[], int offset, int length)把字节数据中从offset位置开始，length个字节的数据写入到输出流。

## **flush()**

OutputStream的flush()方法将所有写入到OutputStream的数据冲刷到相应的目标媒介中。比如，如果输出流是FileOutputStream，那么写入到其中的数据可能并没有真正写入到磁盘中。即使所有数据都写入到了FileOutputStream，这些数据还是有可能保留在内存的缓冲区中。通过调用flush()方法，可以把缓冲区内的数据刷新到磁盘(或者网络，以及其他任何形式的目标媒介)中。

## **close()**

当你结束数据写入时，需要关闭OutputStream。通过调用close()可以达到这一点。因为OutputStream的各种write()方法可能会抛出IO异常，所以你需要把调用close()的关闭操作方在finally块中执行。这是一个OutputStream调用close()的例子：

````java
OutputStream output = null;

try{
  output = new FileOutputStream("c:\\data\\output-text.txt");

  while(hasMoreData()) {
    int data = getMoreData();
    output.write(data);
  }
} finally {
    if(output != null) {
        output.close();
    }
}
````

这个例子在finally块中调用close()方法。虽然这种方式可以确保OutputStream关闭，但却不是一个完美的异常处理方案。我在[Java IO异常处理](http://ifeve.com/java-io-%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86/)这文章中更加详细地探讨了IO的异常处理。