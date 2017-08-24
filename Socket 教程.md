# Socket 教程

## 2.3  关闭Socker

## 2.4 半关闭Socket

###问题1

进程A与进程B通过Socket通信,假定进程A输出数据,进程B读入数据.进程A如何告诉进程B所有的数据已经输出完毕呢?

- 当进程A与进程B交换的是字符流,并且都一行一行地读/写数据时,可以事先约定以一个特殊的标志作为结束标志.

- 进程A先发送一条消息,告诉所发送的正文长度,然后再发送正文.进程B先获知进程A将发送的正文的长度,接下来只要读取完该长度的字符或者字节,就停止读数据.

- 进程A发完所有的数据后,关闭Socket.当进程B读入进程A发送的所有数据后,再次执行输入流read()方法时,该方法返回-1;

- 当掉用Socket的close)方法关闭Socket时,它的输出流和输入流也关闭.有的时候,可能仅仅希望关闭输出流或输入流之一.此时可以采用Socket类提供的半关闭方法.

  - shutdownInput()关闭输入流
  - shutdownIotput() 关闭输出流

  值得注意的是,先后掉用Socket的shutdownInput()和shutdownOutput()方法,仅仅关闭了输入流和输出流,并不等价于调用Socket的close()方法.在通信结束后,仍然调用Socket的close()方法,因为只有该方法才会释放Socket占用的资源.如占用的本地端口等.

  当客户与服务器通信,如果有一方断开连接会怎么样



## 2.5 设置Socket的选项

Socket 有以下几个选项

* TCP_NODELAY 表示立即发送数据
* SO_RESUSEADDR 表示时候允许重用Socket所绑定的本地地址
* SO_TIMEOUT 表示接受数据时的等待超时时间.
* SO_LINGER 表示执行Socket的close()方法,是否立即关闭giceng的Socket
* SO_SNFBUF 表示发送数据的缓冲区的大小
* SO_RECVUF表示接受数据的缓冲区大小
* SO_KEEPALIVE 表示对于长时间处于空闲状态的Socket,时候要自动把它关闭
* OOBINLINE表示时候支持发送一个字节的TCP紧急数据