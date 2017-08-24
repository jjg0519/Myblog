# Java IO: System.in, System.out, System.err

System.in, System.out, System.err这3个流同样是常见的数据来源和数据流目的地。使用最多的可能是在控制台程序里利用System.out将输出打印到控制台上。

JVM启动的时候通过Java运行时初始化这3个流，所以你不需要初始化它们(尽管你可以在运行时替换掉它们)。

## **System.in**

System.in是一个典型的连接控制台程序和键盘输入的InputStream流。通常当数据通过命令行参数或者配置文件传递给命令行Java程序的时候，System.in并不是很常用。图形界面程序通过界面传递参数给程序，这是一块单独的Java IO输入机制。

## **System.out**

System.out是一个PrintStream流。System.out一般会把你写到其中的数据输出到控制台上。System.out通常仅用在类似命令行工具的控制台程序上。System.out也经常用于打印程序的调试信息(尽管它可能并不是获取程序调试信息的最佳方式)。

## **System.err**

System.err是一个PrintStream流。System.err与System.out的运行方式类似，但它更多的是用于打印错误文本。一些类似Eclipse的程序，为了让错误信息更加显眼，会将错误信息以红色文本的形式通过System.err输出到控制台上。



## **System.out和System.err的简单例子：**

这是一个System.out和System.err结合使用的简单示例：

```java
try {
  InputStream input = new FileInputStream("c:\\data\\...");
  System.out.println("File opened...");

} catch (IOException e){
  System.err.println("File opening failed:");
  e.printStackTrace();
}
```

## **替换系统流**

尽管System.in, System.out, System.err这3个流是java.lang.System类中的静态成员(译者注：这3个变量均为final static常量)，并且已经预先在JVM启动的时候初始化完成，你依然可以更改它们。只需要把一个新的InputStream设置给System.in或者一个新的OutputStream设置给System.out或者System.err，之后的数据都将会在新的流中进行读取、写入。

可以使用System.setIn(), System.setOut(), System.setErr()方法设置新的系统流(译者注：这三个方法均为静态方法，内部调用了本地native方法重新设置系统流)。例子如下：

```java
System.out.println("----------------------"); //这会打印到控制台
OutputStream output2 = new FileOutputStream("d:\\system.out.txt");
PrintStream printOut = new PrintStream(output2);
System.setOut(printOut);
System.out.println("----------------------"); //这个会输出到txt
```

