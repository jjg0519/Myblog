### JAVA面试700问（一）

**1、Java环境中的字节码是什么？**

- 由Java 编译器生成的一种代码。
- 由JVM生成的一种代码。
- Java源文件（Java Source File）的别名。
- 一种写在类的实例方法中的代码。

答案：由Java 编译器生成的一种代码。

**2、什么是Java垃圾回收机制？**

- 操作系统周期性的删除系统中所有可用的Java文件.
- 自动删除那些被程序引用但未被使用的包
- 当一个对象的引用（references）不再存在，被这些对象占用的内存会被自动的回收。
- JVM检查所有Java应用的输出删除所有不在有意义的输出。

答案：当一个对象的引用（references）不再存在，被这些对象占用的内存会被自动的回收。

 

**Java小应用程序（Java Applet）跟Java应用程序（Java Application）有什么区别？**

- Java应用程序通常情况下是可以被信任的程序，而Java小应用程序不是。
- Java小应用程序必须在浏览器环境下执行。
- Java小应用程序无法访问计算机中的文件。
- 以上所有都是。

答案：以上所有都是。

**3、在下面这段代码编译和执行的时候：submarine.dive(depth);下面哪个答案是正确的？**

- depth肯定是int类型。
- dive肯定是一个方法。
- dive肯定是实例变量的名字。
- submarine肯定是一个类名。

答案：dive肯定是一个方法。

**4、下面哪个关于匿名内部类的说法是正确的?**

- 仅能继承一个类或实现一个接口。
- 仅能继承一个类或实现多个接口。
- 可以实现多个接口无论是否继承了其他类。

答案：仅能继承一个类或实现一个接口。（译者注：给定的答案是”仅能继承一个类或实现多个接口。“，但经过测试发现匿名内部类无法实现多个接口，正确答案应该是”仅能继承一个类或实现一个接口“）

**5、如果一个线程被定义为守护线程（daemon thread），那么它必须声明在下列哪个方法之前？**

- start方法。
- run方法。
- stop方法。
- 以上都不是。

答案：start方法。

**6、在下列什么情况下你可能会使用Thread的yield方法？**

- 在当前线程调用来使得其他线程拥有同样的或者更高的优先级去运行。
- 在处于等待状态下的线程调用来使它能够运行。
- 让一个线程拥有更高的运行优先级。
- 在当前线程调用并传入一个参数表明让哪个线程可以运行。

答案：在当前线程调用来使得其他线程拥有同样的或者更高的优先级去运行。

**7、下面哪个是提示JVM进行垃圾回收的正确语法：**

- System.free();
- System.setGarbageCollection();
- System.out.gc();
- System.gc();

答案：System.gc();

**8、当子类中定义的方法与父类中定义的方法有同样的方法签名（译者注：方法名+方法参数列表），那么子类的方法是：**

- 重载(Overloading )。
- 重写（Overriding ）。
- 包装（Packing ）。
- 以上都不是。

答案：重写（Overriding ）。

**9、在AWT或Swing中，BoxLayout 布局管理器是如何对组件进行布局的？1）从左至右2）从上到下3）从右到左4）从下至上**

- 1。
- 2。
- 1和2。
- 3和4。

答案：1和2。

**10、不能有子类的类是什么类：**

- 抽象（abstract）。
- 父类（parent class）。
- Final。
- 以上都不是。

答案：Final

**11、Swing组件里面用到下面哪个设计模式：**

- MVC（Model view controller ）。
- 事件委托（Event delegation model）。
- DOM（Document object model ）。
- 网络模式（network model）。

答案：MVC。

**12、让多个线程同时作用到同一个对象上并且能保证结果的可靠性的机制叫做：**

- 装箱（Boxing）。
- 非同步（Unsynchronized ）。
- 同步（synchronized）。
- 以上都不是。

答案：同步（synchronized）。

**13、java.util package包下的所有集合类都实现的是不同的接口**

- 正确。
- 错误。

答案：正确。

**14、DeflaterOutputStream和InflaterInputStream在哪个包下面？**

- java.io。
- java.util。
- java.io.zip。
- java.util.zip。

答案：java.util.zip。

**15、把内存中对象存储到文件的技术是：**

- 同步（synchronization ）。
- 序列化（serialization ）。
- zip压缩。
- doping。

答案：序列化（serialization ）。

**16、静态（static）变量或瞬时（transient）变量不能被序列化**

- 正确。
- 错误。

答案：正确。

**17、javax.swing中的组件是用什么语言开发的：**

- C++。
- C。
- pascal。
- pure java。

答案：pure java

**18、FileOutputStream 读取的是什么类型的数据：**

- character。
- file。
- bytes。
- bit。

答案：bytes。

**19、Java中所有带缓冲机制的类的默认缓冲大小是多少？**

- 128 bytes。
- 256 bytes。
- 512 bytes。
- 1024 bytes。

答案：512 bytes