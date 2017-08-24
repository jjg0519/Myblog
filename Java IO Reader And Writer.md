# Java IO: Reader And Writer

Java IO的Reader和Writer除了基于字符之外，其他方面都与InputStream和OutputStream非常类似。他们被用于读写文本。InputStream和OutputStream是基于字节的，还记得吗？

## **Reader**

Reader类是Java IO中所有Reader的基类。子类包括BufferedReader，PushbackReader，InputStreamReader，StringReader和其他Reader。

这是一个简单的Java IO Reader的例子：

```java
   Reader reader = new FileReader("c:\\data\\myfile.txt");

    int data = reader.read();
    while(data != -1){
        char dataChar = (char) data;
        data = reader.read();
    }
```

请注意，InputStream的read()方法返回一个字节，意味着这个返回值的范围在0到255之间(当达到流末尾时，返回-1)，Reader的read()方法返回一个字符，意味着这个返回值的范围在0到65535之间(当达到流末尾时，同样返回-1)。这并不意味着Reade只会从数据源中一次读取2个字节，Reader会根据文本的编码，一次读取一个或者多个字节。

你通常会使用Reader的子类，而不会直接使用Reader。Reader的子类包括InputStreamReader，CharArrayReader，FileReader等等。可以查看[Java IO概述](http://ifeve.com/java-io-3/)浏览完整的Reader表格。

## **整合Reader与InputStream**

一个Reader可以和一个InputStream相结合。如果你有一个InputStream输入流，并且想从其中读取字符，可以把这个InputStream包装到InputStreamReader中。把InputStream传递到InputStreamReader的构造函数中：

```java
Reader reader = new InputStreamReader(inputStream);
```

在构造函数中可以指定解码方式。更多内容请参阅[InputStreamReader](http://tutorials.jenkov.com/java-io/inputstreamreader.html)。

## **Writer**

Writer类是Java IO中所有Writer的基类。子类包括BufferedWriter和PrintWriter等等。这是一个Java IO Writer的例子：

```java
Writer writer = new FileWriter("c:\\data\\file-output.txt");

writer.write("Hello World Writer");
writer.close();
```

同样，你最好使用Writer的子类，不需要直接使用Writer，因为子类的实现更加明确，更能表现你的意图。常用子类包括OutputStreamWriter，CharArrayWriter，FileWriter等。Writer的write(int c)方法，会将传入参数的低16位写入到Writer中，忽略高16位的数据。

## **整合Writer和OutputStream**

与Reader和InputStream类似，一个Writer可以和一个OutputStream相结合。把OutputStream包装到OutputStreamWriter中，所有写入到OutputStreamWriter的字符都将会传递给OutputStream。这是一个OutputStreamWriter的例子：

```java
Writer writer = new OutputStreamWriter(outputStream);
```



## **整合Reader和Writer**

和字节流一样，Reader和Writer可以相互结合实现更多更有趣的IO，工作原理和把Reader与InputStream或者Writer与OutputStream相结合类似。举个栗子，可以通过将Reader包装到BufferedReader、Writer包装到BufferedWriter中实现缓冲。以下是例子：

```java
Reader reader = new BufferedReader(new FileReader(...));
Writer writer = new BufferedWriter(new FileWriter(...));
```

