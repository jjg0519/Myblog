# Java IO 字节流的ByteArray和Filter

本小节会简要概括Java IO中字节数组与过滤器的输入输出流，主要涉及以下4个类型的流：ByteArrayInputStream，ByteArrayOutputStream，FilterInputStream，FilterOutputStream。请注意，为了清晰，这里忽略了必要的异常处理。想了解更多异常处理的信息，请参考Java IO异常处理。

## **ByteArrayInputStream**

ByteArrayInputStream允许你从字节数组中读取字节流数据，代码如下：

```java
byte[] bytes = ... //get byte array from somewhere.

InputStream input = new ByteArrayInputStream(bytes);

int data = input.read();
while(data != -1) {
  //do something with data

  data = input.read();
}
input.close();
```

如果数据存储在数组中，ByteArrayInputStream可以很方便地读取数据。如果你有一个InputStream变量，又想从数组中读取数据呢？很简单，只需要把字节数组传递给ByteArrayInputStream的构造函数，在把这个ByteArrayInputStream赋值给InputStream变量就可以了(译者注：InputStream是所有字节输入流流的基类，Reader是所有字符输入流的基类，OutputStream与Writer同理)。

## **ByteArrayOutputStream**

ByteArrayOutputStream允许你以数组的形式获取写入到该输出流中的数据，代码如下：

```java
ByteArrayOutputStream output = new ByteArrayOutputStream();

//write data to output stream

byte[] bytes = output.toByteArray();
```



## **FilterInputStream**

[原文链接](http://tutorials.jenkov.com/java-io/filterinputstream.html)

FilterInputStream是实现自定义过滤输入流的基类，基本上它仅仅只是覆盖了InputStream中的所有方法。

```java
FilterInputStream inputStream = new FilterInputStream(new FileInputStream("c:\\myfile.txt"));
```

就我自己而言，我没发现这个类明显的用途。除了构造函数取一个InputStream变量作为参数之外，我没看到FilterInputStream任何对InputStream新增或者修改的地方。如果你选择继承FilterInputStream实现自定义的类，同样也可以直接继承自InputStream从而避免额外的类层级结构。

## **FilterOutputStream**

[原文链接](http://tutorials.jenkov.com/java-io/filteroutputstream.html)

内容同FilterInputStream，不再赘述。