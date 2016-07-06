#java LinkedList ArrayList 随机访问效率 list.get(int index)
理论上来说，肯定LinkedList比ArrayList随机访问效率要低，然后LinkedList比ArrayList插入删除元素要快。

 

突然想起之前写一个日记本程序，是用LinkedList+Map索引，作为数据库。Map记录了LinkedList中每一个日记的index和日期之间的对应关系。从Map中获取到某个日期对应日记的index，然后再去LinkedList，get(index)。

 

 

复制代码
        Integer a = 1;
        LinkedList list = new LinkedList();
        for (int i = 0; i < 2000000; i++) {
            list.add(a);
        }
        System.out.println(list.size());
        long start = System.nanoTime();
        list.get(1000000);
        long end = System.nanoTime();
        System.out.println(end - start);
复制代码
上边一段代码，看出了几样事情：

 

1.LinkedList的随机访问速度确实差点，大概用了17毫秒。下边会贴出LinkedList随机访问的源代码，也就是这里为什么选择1000000中间数的原因。

2.Java栈区和堆区都是有限的，list那里如果一次添加5000000个item就会内存溢出

    （Exception in thread "main" java.lang.OutOfMemoryError: Java heap space）。

     但有点奇怪，不是new了在内存堆区吗？内存堆区也会爆~~

 

下边是LinkedList随机访问的源代码，采取了折半的遍历方式，每个循环里边进行一次int的比较。

 

复制代码
    private Entry<E> entry(int index) {
        if (index < 0 || index >= size)
            throw new IndexOutOfBoundsException("Index: "+index+
                                                ", Size: "+size);
        Entry<E> e = header;
        if (index < (size >> 1)) {
            for (int i = 0; i <= index; i++)
                e = e.next;
        } else {
            for (int i = size; i > index; i--)
                e = e.previous;
        }
        return e;
    }
复制代码
 

 

换了ArrayList的话，添加5000000个item都不会爆，但再大点，还是会爆~~

随机访问效率确实高很多，只需要16微秒左右，足足快了1千倍，而且跟get的index无关。