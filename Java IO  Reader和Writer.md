# Java IO  Reader和Writer

Reader是Java IO中所有Reader的基类。Reader与InputStream类似，不同点在于，Reader基于字符而非基于字节。换句话说，Reader用于读取文本，而InputStream用于读取原始字节。

请记住，Java内部使用UTF8编码表示字符串。输入流中一个字节可能并不等同于一个UTF8字符。如果你从输入流中以字节为单位读取UTF8编码的文本，并且尝试将读取到的字节转换成字符，你可能会得不到预期的结果。

read()方法返回一个包含了读取到的字符内容的int类型变量(译者注：0~65535)。如果方法返回-1，表明Reader中已经没有剩余可读取字符，此时可以关闭Reader。-1是一个int类型，不是byte或者char类型，这是不一样的。

你通常会使用Reader的子类，而不会直接使用Reader。Reader的子类包括InputStreamReader，CharArrayReader，FileReader等等。可以查看[Java IO概述](http://ifeve.com/java-io-3/)浏览完整的Reader表格。

Reader通常与文件、字符数组、网络等数据源相关联，[Java IO概述](http://ifeve.com/java-io-3/)中同样说明了这一点。

## **Writer**

[原文链接](http://tutorials.jenkov.com/java-io/writer.html)

Writer是Java IO中所有Writer的基类。与Reader和InputStream的关系类似，Writer基于字符而非基于字节，Writer用于写入文本，OutputStream用于写入字节。

同样，你最好使用Writer的子类，不需要直接使用Writer，因为子类的实现更加明确，更能表现你的意图。常用子类包括OutputStreamWriter，CharArrayWriter，FileWriter等。

Writer的write(int c)方法，会将传入参数的低16位写入到Writer中，忽略高16位的数据。