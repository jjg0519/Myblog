# 大日志包进行分割的N种方法

**前言：**
   网站一个主要域名的日志包，解压缩之后有10多个G，当日志分析系统处理此日志包时，需要的分析时间过长，而且如分析过程中出错，要清空数据后再分析，所以需要将日志进行分割成多个日志包，通过多个日志分析系统并发分析，本文主要记录一下我测试的分割过程。

**说明：**
本文采用一个50M大小的日志文件进行测试。
日志文件名：log.txt.gz。
文件行数：208363

## split分割

语法:

```bash
split  [-<行数>][-b <字节>][-C <字节>][-l <行数>][要切割的文件][输出文件名]
-<行数> : 指定每多少行切成一个小文件
-b<字节> : 指定每多少字节切成一个小文件
-C<字节> : 与参数"-b"相似，但是在切 割时将尽量维持每行的完整性
```
###按行数分割
```bash
# gunzip log.txt.gz             //一定要先解压，否则分割的文件是不能cat/zcat显示；
# wc -l log.txt                 //计算一个文件的总行数；
 208363 log.txt
# split -l 120000 log.txt newlog    //通过指定行数，将日志分割成两个文件
# du -sh *
50M     log.txt
29M     newlogaa
22M     newlogab
# file *                         //分割后的文件与原文件属性一样
log.txt:  ASCII text, with very long lines, with CRLF line terminators
newlogaa: ASCII text, with very long lines, with CRLF line terminators
newlogab: ASCII text, with very long lines, with CRLF line terminators
# gzip newlogaa newlogab         //将分割后的文件进行压缩，以便传输
```

### 按大小分割

````bash
# split -C 26000000 log.txt newlo
````

相对-l(按行)来说，统计一个大文件的字节要比统计行数要快得多。用-C（大C）还是-l，请按实际情况而定。

## dd 分割

```bash
# gunzip log.txt.gz             //一定要先解压，否则分割的文件是不能cat/zcat显示；
# dd bs=20480 count=1500  if=log.txt of=newlogaa       //按大小分第一个文件
# dd bs=20480 count=1500  if=log.txt of=newlogab skip=1500   //将大小之后的生成另一个文件
# file *
log.txt:  ASCII text, with very long lines, with CRLF line terminators
newlogaa: ASCII text, with very long lines, with CRLF line terminators
newlogab: ASCII text, with very long lines, with CRLF line terminators
```

**分割没问题，但会出现同一行分到不同文件的情况，除非你以及日志分析系统可以"容忍"**

## head+tail 分割

```bash
#gzip log.txt.gz                //如不解压缩，下面请用zcat。
# wc -l log.txt                 //统计一个行数
 208363 log.txt
# head -n `echo $((208363/2+1))` log.txt > newloga.txt       //前x行重定向输出到一个文件中；
# tail –n `echo $((208363-208362/2-1))` log.txt >newlogb.txt  //后x行重定向输出到一个文件中；
# gzip newloga.txt newlogb.txt           //将两个文件进行压缩
```

## awk分割

```bash
#gzip log.txt.gz
#awk ‘{if (NR<120000) print $0}’ log.txt >newloga.txt
#awk ‘{if (NR>=120000) print $0}’ log.txt >newlogb.txt

#####以上两个命令，都要遍历整个文件，所以考虑到效率，应使用合并成：
# awk ‘{if (NR<120000) print $0 >”newloga.txt”;if (NR>=120000) print $0>”newlogb.txt”}’ log.txt
```

 

## 总结：

以上四种方法，除了dd之外的三种方式都可以很好的整行分割日志文件。进行分割时，应考虑在读一次文件的同时完成，如不然，按下面的方式分割：

```bash
Cat log.txt| head –12000 >newloga.txt
Cat log.txt | tail –23000  >newlogb.txt
```

如用此方法分割文件的后一部分，那么执行第二行命令文件时，前x行是白白读一遍的，执行的效率将很差，如文件过大，还可能出现内存不够的情况。

log

log

 

log

log