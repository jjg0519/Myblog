# Java IO InputStreamReader和OutputStreamWriter



本章节将简要介绍InputStreamReader和OutputStreamWriter。细心的读者可能会发现，在之前的文章中，IO中的类要么以Stream结尾，要么以Reader或者Writer结尾，那这两个同时以字节流和字符流的类名后缀结尾的类是什么用途呢？简单来说，这两个类把字节流转换成字符流，中间做了数据的转换，类似适配器模式的思想。

## InputStreamReader

```java
InputStream inputStream       = new FileInputStream("c:\\data\\input.txt");
Reader      inputStreamReader = new InputStreamReader(inputStream);

int data = inputStreamReader.read();
while(data != -1){
    char theChar = (char) data;
    data = inputStreamReader.read();
}

inputStreamReader.close();
```

注意：为了清晰，代码忽略了一些必要的异常处理。想了解更多异常处理的信息，请参考Java IO异常处理。

read()方法返回一个包含了读取到的字符内容的int类型变量(译者注：0~65535)。代码如下：

```java
int data = inputStreamReader.read();
```

你可以把返回的int值转换成char变量，就像这样：

```java
char aChar = (char) data;//译者注：这里不会造成数据丢失，因为返回的int类型变量data只有低16位有数据，高16位没有数据

```

如果方法返回-1，表明Reader中已经没有剩余可读取字符，此时可以关闭Reader。-1是一个int类型，不是byte或者char类型，这是不一样的。

InputStreamReader同样拥有其他可选的构造函数，能够让你指定将底层字节流解释成何种编码的字符流。例子如下：

```java
InputStream inputStream       = new FileInputStream("c:\\data\\input.txt");
Reader      inputStreamReader = new InputStreamReader(inputStream, "UTF-8");
```

注意构造函数的第二个参数，此时该InputStreamReader会将输入的字节流转换成UTF8字符流。

[使用try-with-resources](http://tutorials.jenkov.com/java-exception-handling/try-with-resources.html) 关闭流

```java
InputStream input = new FileInputStream("data/text.txt");

try(InputStreamReader inputStreamReader =
    new InputStreamReader(input)){

    int data = inputStreamReader.read();
    while(data != -) {
        System.out.print((char) data));
        data = inputStreamReader.read();
    }
}
```

## OutputStreamWriter

OutputStreamWriter会包含一个OutputStream，从而可以将该输出字节流转换成字符流，代码如下：

```java
OutputStream outputStream       = new FileOutputStream("c:\\data\\output.txt");
Writer       outputStreamWriter = new OutputStreamWriter(outputStream);
outputStreamWriter.write("Hello World");
outputStreamWriter.close();
```

使用[try-with-resources](http://tutorials.jenkov.com/java-exception-handling/try-with-resources.html) 关闭流

```java
OutputStream output = new FileOutputStream("data/data.bin");
try(OutputStreamWriter outputStreamWriter =
    new OutputStreamWriter(output)){
    Person person = new Person();
    person.name = "Jakob Jenkov";
    person.age  = 40;
    outputStreamWriter.writeObject(person);
}
```

