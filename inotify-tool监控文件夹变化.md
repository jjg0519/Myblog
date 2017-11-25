# inotify-too监控文件夹变化





## 安装

- inotify-tools项目地址：<https://github.com/rvoicilas/inotify-tools>
- inotify-tools下载地址：<http://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz>

### 命令行安装

###Debian/Ubuntu

```bash
apt-get install inotify-tools
```

###Fedora/Centos

```bash
yum install inotify-tools
```

### 源代码

```bash
[root@rsync-client ~]# wget http://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz
tar xzf inotify-tools-3.13.tar.gz
[root@rsync-client ~]# cd inotify-tools-3.13
[root@rsync-client ~]# ./configure --prefix=/usr && make && su -c 'make install'

root@shj-db02 inotify-tools-3.14]# ll -h /usr/local/lib
total 216K
-rw-r--r--. 1 root root 123K Nov 25 09:39 libinotifytools.a
-rwxr-xr-x. 1 root root  991 Nov 25 09:39 libinotifytools.la
lrwxrwxrwx. 1 root root   24 Nov 25 09:39 libinotifytools.so -> libinotifytools.so.0.4.1
lrwxrwxrwx. 1 root root   24 Nov 25 09:39 libinotifytools.so.0 -> libinotifytools.so.0.4.1
-rwxr-xr-x. 1 root root  86K Nov 25 09:39 libinotifytools.so.0.4.1

```

##inotify配置相关参数

inotify定义了下列的接口参数，可以用来限制inotify消耗kernel memory的大小。由于这些参数都是内存参数，因此，可以根据应用需求，实时的调节其大小：

- `/proc/sys/fs/inotify/max_queued_evnets`表示调用inotify_init时分配给inotify instance中可排队的event的数目的最大值，超出这个值的事件被丢弃，但会触发IN_Q_OVERFLOW事件。
- `/proc/sys/fs/inotify/max_user_instances`表示每一个real user ID可创建的inotify instatnces的数量上限。
- `/proc/sys/fs/inotify/max_user_watches`表示每个inotify instatnces可监控的最大目录数量。如果监控的文件数目巨大，需要根据情况，适当增加此值的大小。

根据以上在32位或者64位系统都可以执行：

```bash
echo 104857600 > /proc/sys/fs/inotify/max_user_watches
echo 'echo 104857600 > /proc/sys/fs/inotify/max_user_watches' >> /etc/rc.local
#把他加入/etc/rc.local就可以实现每次重启都生效
```

如果遇到以下错误：

> inotifywait: error while loading shared libraries: libinotifytools.so.0: cannot open shared object file: No such file or directory

解决方法：

```bash
#32位系统：
ln -s /usr/local/lib/libinotifytools.so.0 /usr/lib/libinotifytools.so.0
#64位系统：
ln -s /usr/local/lib/libinotifytools.so.0 /usr/lib64/libinotifytools.so.0
```

## inotifywait命令参数

- `-m`是要持续监视变化。
- `-r`使用递归形式监视目录。
- `-q`减少冗余信息，只打印出需要的信息。
- `-e`指定要监视的事件列表。
- `--timefmt`是指定时间的输出格式。
- `--format`指定文件变化的详细信息

###inotifywait可监听的事件

| 事件     | 描述                |
| ------ | ----------------- |
| access | **访问**，读取文件。      |
| modify | **修改**，文件内容被修改。   |
| attrib | **属性**，文件元数据被修改。  |
| move   | **移动**，对文件进行移动操作。 |
| create | **创建**，生成新文件      |
| open   | **打开**，对文件进行打开操作。 |
| close  | **关闭**，对文件进行关闭操作。 |
| delete | **删除**，文件被删除。     |

###inotifywait使用例子

####实时监控/home的所有事件（包括文件的访问，写入，修改，删除等）

```bash
inotifywait -rm /home
```

####统计/home文件系统的事件

```bash
inotifywatch -v -e access -e modify -t 60 -r /home
```

####实时监控/home目录的文件或目录创建，修改和删除相关事件

```bash
[root@rsync-client ~]# inotifywait -mrq -e create,modify,delete /home
/home/ CREATE,ISDIR test2
/home/test2/ CREATE .bash_profile
/home/test2/ MODIFY .bash_profile
/home/test2/ CREATE .bash_logout
/home/test2/ MODIFY .bash_logout
/home/test2/ CREATE .bashrc
/home/test2/ MODIFY .bashrc
```

####实时监控/etc/passwd的文件修改，删除和权限相关事件，并且要求指定输出格式为27/06/14 16:12 /etc/passwd ATTRIB。

```bash
[root@rsync-client ~]# inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format  '%T %w%f %e' --event modify,delete,attrib  /etc/passwd
27/06/14 16:39 /etc/passwd ATTRIB
27/06/14 16:39 /etc/passwd IGNORED
```

####写一个脚本实现对 /data/web 目录进行监控，监控文件删除，修改，创建和权限相关事件，并且要求将监控信息写入/var/log/web_watch.log。要求日志条目要清晰明了，能突显文件路径、事件名和时间

```bash
[root@rsync-client ~]# cat web_watch.sh
#!/bin/bash
inotifywait -mrq --timefmt '%y/%m/%d %H:%M' --format  '%T %w%f %e' --event delete,modify,create,attrib  /data/web | while read  date time file event
  do
      case $event in
          MODIFY|CREATE|MOVE|MODIFY,ISDIR|CREATE,ISDIR|MODIFY,ISDIR)
                  echo $event'-'$file'-'$date'-'$time >> /var/log/web_watch.log
              ;;
   
          MOVED_FROM|MOVED_FROM,ISDIR|DELETE|DELETE,ISDIR)
                  echo $event'-'$file'-'$date'-'$time /var/log/web_watch.log
              ;;
      esac
  done
[root@rsync-client ~]# cat /var/log/web_watch.log 
CREATE-/data/web/a-14/06/27-16:21
CREATE-/data/web/aa-14/06/27-16:21
CREATE-/data/web/aaaa-14/06/27-16:24
CREATE-/data/web/aaaaa-14/06/27-16:24
```

####按格式输出某个目录下的文件变化

```
#!/bin/bash
#filename watchdir.sh
path=$1
/usr/local/bin/inotifywait -mrq --timefmt '%d/%m/%y/%H:%M' --format '%T %w %f' -e modify,delete,create,attrib $path

执行输出：
./watchdir.sh /data/wsdata/tools/
04/01/13/16:34 /data/wsdata/tools/ .j.jsp.swp
04/01/13/16:34 /data/wsdata/tools/ .j.jsp.swx
04/01/13/16:34 /data/wsdata/tools/ .j.jsp.swx
04/01/13/16:34 /data/wsdata/tools/ .j.jsp.swp
04/01/13/16:34 /data/wsdata/tools/ .j.jsp.swp
04/01/13/16:34 /data/wsdata/tools/ .j.jsp.swp
04/01/13/16:34 /data/wsdata/tools/ .j.jsp.swp
04/01/13/16:34 /data/wsdata/tools/ .j.jsp.swp
04/01/13/16:35 /data/wsdata/tools/ 4913
04/01/13/16:35 /data/wsdata/tools/ 4913
04/01/13/16:35 /data/wsdata/tools/ 4913
04/01/13/16:35 /data/wsdata/tools/ j.jsp
04/01/13/16:35 /data/wsdata/tools/ j.jsp
04/01/13/16:35 /data/wsdata/tools/ j.jsp
04/01/13/16:35 /data/wsdata/tools/ j.jsp~
04/01/13/16:35 /data/wsdata/tools/ .j.jsp.swp
```

## inotifywatch参数

语法

```bash
inotifywatch [-hvzrqf] [-e ] [-t ] [-a ] [-d ] [ ... ]
```

参数

```bash
-h，–help    # 输出帮助信息
-v，–verbose # 输出详细信息
@             # 排除不需要监视的文件，可以是相对路径，也可以是绝对路径。
–fromfile    # 从文件读取需要监视的文件或排除的文件，一个文件一行，排除的文件以@开头。
-z，–zero    # 输出表格的行和列，即使元素为空
–exclude     # 正则匹配需要排除的文件，大小写敏感。
–excludei    # 正则匹配需要排除的文件，忽略大小写。
-r，–recursive  # 监视一个目录下的所有子目录。
-t，–timeout    # 设置超时时间
-e，–event      # 只监听指定的事件。
-a，–ascending  # 以指定事件升序排列。
-d，–descending # 以指定事件降序排列
```

### inotifywatch例子

#### 统计/home目录所在文件系统发生的事件次数

```bash

[root@rsync-client ~]# inotifywatch -v -e create -e modify -e delete -t 30 -r /home
Establishing watches...
Setting up watch(es) on /home
OK, /home is now being watched.
Total of 3 watches.
Finished establishing watches, now collecting statistics.
Will listen for events for 60 seconds.
total  modify  create  delete  filename
8           3            4          1       /home/
```

监控的时候，我在新打开的终端上，创建了4个文件，修改了3个文件内容，删除了一个文件。等监控的30秒时间到了之后，他就会显示出上面的事件次数报告！





