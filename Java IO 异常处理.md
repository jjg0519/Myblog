# Java IO: 异常处理

流与Reader和Writer在结束使用的时候，需要正确地关闭它们。通过调用close()方法可以达到这一点。不过这需要一些思考。请看下边的代码：

```java
InputStream input = new FileInputStream("c:\\data\\input-text.txt");
int data = input.read();
while(data != -1) {
  //do something with data...
  doSomethingWithData(data);

  data = input.read();
}
input.close();
```

第一眼看这段代码时，可能觉得没什么问题。可是如果在调用doSomethingWithData()方法时出现了异常，会发生什么呢？没错，这个InputStream对象就不会被关闭。

为了避免异常造成流无法被关闭，我们可以把代码重写成这样:

```java
InputStream input = null;

try{
  input = new FileInputStream("c:\\data\\input-text.txt");

  int data = input.read();
  while(data != -1) {
      //do something with data...
      doSomethingWithData(data);

      data = input.read();
  }
}catch(IOException e){
  //do something with e... log, perhaps rethrow etc.
} finally {
  if(input != null) input.close();
}
```

注意到这里把InputStream的关闭代码放到了finally块中，无论在try-catch块中发生了什么，finally内的代码始终会被执行，所以这个InputStream总是会被关闭。

但是如果close()方法抛出了异常，告诉你流已经被关闭过了呢？为了解决这个难题，你也需要把close()方法写在try-catch内部，就像这样：

```java
} finally {
  try{
    if(input != null) input.close();
  } catch(IOException e){
    //do something, or ignore.
  }
}
```

这段解决了InputStream(或者OutputStream)流关闭的问题的代码，确实是有一些不优雅，尽管能够正确处理异常。如果你的代码中重复地遍布了这段丑陋的异常处理代码，这不是很好的一个解决方案。如果一个匆忙的家伙贪图方便忽略了异常处理呢？

此外，想象一下某个异常最先从doSomethingWithData方法内抛出。第一个catch会捕获到异常，然后在finally里程序会尝试关闭InputStream。但是如果还有异常从close()方法内抛出呢？这两个异常中得哪个异常应当往调用栈上传播呢？

幸运的是，有一个办法能够解决这个问题。这个解决方案称作“异常处理模板”。创建一个正确关闭流的模板，能够在代码中做到一次编写，重复使用，既优雅又简单。详情参见[Java异常处理模板](http://tutorials.jenkov.com/java-exception-handling/exception-handling-templates.html)。

## Java7中IO的异常处理

从Java7开始，一种新的被称作“try-with-resource”的异常处理机制被引入进来。这种机制旨在解决针对InputStream和OutputStream这类在使用完毕之后需要关闭的资源的异常处理。可以浏览[Try with Resource in Java 7](http://tutorials.jenkov.com/java-exception-handling/try-with-resources.html)获得更多信息。

