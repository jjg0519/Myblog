# Java IO: 其他字节流

本小节会简要概括Java IO中的PushbackInputStream，SequenceInputStream和PrintStream。其中，最常用的是PrintStream，System.out和System.err都是PrintStream类型的变量，请查看[Java IO: System.in, System.out, System.err](http://ifeve.com/java-io-system-in-system-out-system-err/#more-16426)浏览更多关于System.out和System.err的信息。

## PushbackInputStream

PushbackInputStream用于解析InputStream内的数据。有时候你需要提前知道接下来将要读取到的字节内容，才能判断用何种方式进行数据解析。PushBackInputStream允许你这么做，你可以把读取到的字节重新推回到InputStream中，以便再次通过read()读取。代码如下：

```java
PushbackInputStream input = new PushbackInputStream(                                new FileInputStream("c:\\data\\input.txt"));
int data = input.read();
input.unread(data);
```

可以通过PushBackInputStream的构造函数设置推回缓冲区的大小，代码如下：

```java
PushbackInputStream input = new PushbackInputStream(new FileInputStream("c:\\data\\input.txt"), 8);

```

这个例子设置了8个字节的缓冲区，意味着你最多可以重新读取8个字节的数据。

## SequenceInputStream

SequenceInputStream把一个或者多个InputStream整合起来，形成一个逻辑连贯的输入流。当读取SequenceInputStream时，会先从第一个输入流中读取，完成之后再从第二个输入流读取，以此推类。代码如下：

```java
InputStream input1 = new FileInputStream("c:\\data\\file1.txt");
InputStream input2 = new FileInputStream("c:\\data\\file2.txt");

SequenceInputStream sequenceInputStream =
    new SequenceInputStream(input1, input2);

int data = sequenceInputStream.read();
while(data != -1){
    System.out.println(data);
    data = sequenceInputStream.read();
}
```

通过SequenceInputStream，例子中的2个InputStream使用起来就如同只有一个InputStream一样(译者注：SequenceInputStream的read()方法会在读取到当前流末尾时，关闭流，并把当前流指向逻辑链中的下一个流，最后返回新的当前流的read()值)。

## PrintStream

PrintStream允许你把格式化数据写入到底层OutputStream中。比如，写入格式化成文本的int，long以及其他原始数据类型到输出流中，而非它们的字节数据。代码如下：

```java
PrintStream printStream = new PrintStream(outputStream);

printStream.print(true);
printStream.print((int) 123);
printStream.print((float) 123.456);

printStream.close();
```

PrintStream包含2个强大的函数，分别是format()和printf()(这两个函数几乎做了一样的事情，但是C程序员会更熟悉printf())

译者注：其中一个printf()函数实现如下：

````java
PrintStream printStream = new PrintStream(outputStream);

printStream.printf(Locale.UK, "Text + data: %1$", 123);

printStream.close();
````







本小节会简要概括Java IO中的PushbackReader，LineNumberReader，StreamTokenizer，PrintWriter，StringReader，StringWriter。

## PushbackReader

PushbackReader与PushbackInputStream类似，唯一不同的是PushbackReader处理字符，PushbackInputStream处理字节。代码如下：

```java
PushbackReader pushbackReader =
    new PushbackReader(new FileReader("c:\\data\\input.txt"));

int data = pushbackReader.read();

pushbackReader.unread(data);
```

同样可以设置缓冲区大小，代码如下：

```jaVA
int pushbackLimit = 8;
PushbackReader reader = new PushbackReader(
                                new FileReader("c:\\data\\input.txt"),
                                pushbackLimit);
```

## LineNumberReader

LineNumberReader是记录了已读取数据行号的BufferedReader。默认情况下，行号从0开始，当LineNumberReader读取到行终止符时，行号会递增(译者注：换行\n，回车\r，或者换行回车\n\r都是行终止符)。

你可以通过getLineNumber()方法获取当前行号，通过setLineNumber()方法设置当前行数(译者注：setLineNumber()仅仅改变LineNumberReader内的记录行号的变量值，不会改变当前流的读取位置。流的读取依然是顺序进行，意味着你不能通过setLineNumber()实现流的跳跃读取)。代码如下：

```JAVA
LineNumberReader lineNumberReader = 
    new LineNumberReader(new FileReader("c:\\data\\input.txt"));

int data = lineNumberReader.read();
while(data != -1){
    char dataChar = (char) data;
    data = lineNumberReader.read();
    int lineNumber = lineNumberReader.getLineNumber();
}
lineNumberReader.close();
```

如果解析的文本有错误，LineNumberReader可以很方便地定位问题。当你把错误报告给用户时，如果能够同时把出错的行号提供给用户，用户就能迅速发现并且解决问题。

## StreamTokenizer

StreamTokenizer(译者注：请注意不是StringTokenizer)可以把输入流(译者注：InputStream和Reader。通过InputStream构造StreamTokenizer的构造函数已经在JDK1.1版本过时，推荐将InputStream转化成Reader，再利用此Reader构造StringTokenizer)分解成一系列符号。比如，句子”Mary had a little lamb”的每个单词都是一个单独的符号。

当你解析文件或者计算机语言时，为了进一步的处理，需要将解析的数据分解成符号。通常这个过程也称作分词。

通过循环调用nextToken()可以遍历底层输入流的所有符号。在每次调用nextToken()之后，StreamTokenizer有一些变量可以帮助我们获取读取到的符号的类型和值。这些变量是：

ttype 读取到的符号的类型(字符，数字，或者行结尾符)

sval 如果读取到的符号是字符串类型，该变量的值就是读取到的字符串的值

nval 如果读取到的符号是数字类型，该变量的值就是读取到的数字的值

```JAVA

StreamTokenizer streamTokenizer = new StreamTokenizer(
        new StringReader("Mary had 1 little lamb..."));

while(streamTokenizer.nextToken() != StreamTokenizer.TT_EOF){

    if(streamTokenizer.ttype == StreamTokenizer.TT_WORD) {
        System.out.println(streamTokenizer.sval);
    } else if(streamTokenizer.ttype == StreamTokenizer.TT_NUMBER) {
        System.out.println(streamTokenizer.nval);
    } else if(streamTokenizer.ttype == StreamTokenizer.TT_EOL) {
        System.out.println();
    }

}
streamTokenizer.close();
```





StreamTokenizer可以识别标示符，数字，引用的字符串，和多种注释类型。你也可以指定何种字符解释成空格、注释的开始以及结束等。在StreamTokenizer开始解析之前，所有的功能都可以进行配置。请查阅官方文档获取更多信息。

## **PrintWriter**

[原文链接](http://tutorials.jenkov.com/java-io/printwriter.html)

与[PrintStream](http://ifeve.com/java-io-%E5%85%B6%E4%BB%96%E5%AD%97%E8%8A%82%E6%B5%81/)类似，PrintWriter可以把格式化后的数据写入到底层writer中。由于内容相似，不再赘述。

值得一提的是，PrintWriter有更多种构造函数供使用者选择，除了可以输出到文件、Writer以外，还可以输出到OutputStream中(译者注：PrintStream只能把数据输出到文件和OutputStream)。

## **StringReader**

[原文链接](http://tutorials.jenkov.com/java-io/stringreader.html)

StringReader能够将原始字符串转换成Reader，代码如下：

```JAVA

String input = "Input String... ";
StringReader stringReader = new StringReader(input);

int data = stringReader.read();
while(data != -1) {
  //do something with data...
  doSomethingWithData(data);

  data = stringReader.read();
}
stringReader.close();
```



## StringWriter

[原文链接](http://tutorials.jenkov.com/java-io:-stringwriter.html)

StringWriter能够以字符串的形式从Writer中获取写入到其中数据，代码如下：

toString()方法能够获取StringWriter中的字符串数据。

```JAVA

String input = "Input String... ";
StringReader stringReader = new StringReader(input);

int data = stringReader.read();
while(data != -1) {
  //do something with data...
  doSomethingWithData(data);

  data = stringReader.read();
}
stringReader.close();
```



getBuffer()方法能够获取StringWriter内部构造字符串时所使用的StringBuffer对象。