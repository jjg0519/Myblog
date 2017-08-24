# Java网络教程之Socket

当我们想要在Java中使用TCP/IP通过网络连接到服务器时，就需要创建java.net.Socket对象并连接到服务器。假如希望使用Java NIO，也可以创建Java NIO中的[SocketChannel](http://tutorials.jenkov.com/java-nio/socketchannel.html)对象。

## 创建Socket

下面的示例代码是连接到IP地址为78.64.84.171服务器上的80端口，这台服务器就是我们的Web服务器（www.jenkov.com），而80端口就是Web服务端口。

```
Socket socket = new Socket("78.46.84.171", 80);

```

我们也可以像如下示例中使用域名代替IP地址：

```
Socket socket = new Socket("jenkov.com", 80);

```

## Socket发送数据

要通过Socket发送数据，我们需要获取Socket的输出流（[OutputStream](http://tutorials.jenkov.com/java-io/outputstream.html)），示例代码如下：

```
Socket socket = new Socket("jenkov.com", 80);
OutputStream out = socket.getOutputStream(); 

out.write("some data".getBytes());
out.flush();
out.close(); 

socket.close();

```

代码非常简单，但是想要通过网络将数据发送到服务器端，一定不要忘记调用flush()方法。操作系统底层的TCP/IP实现会先将数据放入一个更大的数据缓存块中，而缓存块的大小是与TCP/IP的数据包大小相适应的。（译者注：调用flush()方法只是将数据写入操作系统缓存中，并不保证数据会立即发送）

## Socket读取数据

从Socket中读取数据，我们就需要获取Socket的输入流（[InputStream](http://tutorials.jenkov.com/java-io/inputstream.html)），代码如下：

```
Socket socket = new Socket("jenkov.com", 80);
InputStream in = socket.getInputStream(); 

int data = in.read();
//... read more data... 

in.close();
socket.close();

```

代码也并不复杂，但需要注意的是，从Socket的输入流中读取数据并不能读取文件那样，一直调用read()方法直到返回-1为止，因为对Socket而言，只有当服务端关闭连接时，Socket的输入流才会返回-1，而是事实上服务器并不会不停地关闭连接。假设我们想要通过一个连接发送多个请求，那么在这种情况下关闭连接就显得非常愚蠢。

因此，从Socket的输入流中读取数据时我们必须要知道需要读取的字节数，这可以通过让服务器在数据中告知发送了多少字节来实现，也可以采用在数据末尾设置特殊字符标记的方式连实现。

##关闭Socket

当使用完Socket后我们必须将Socket关闭，断开与服务器之间的连接。关闭Socket只需要调用Socket.close()方法即可，代码如下：

```
Socket socket = new Socket("jenkov.com", 80); 

socket.close();
```