# Ubuntu  日常

# 常用命令

*   文件打开数？设置最大打开文件数

    ```bash
    ulimit  -a
    ```

    ​

*   ​

## Ubuntu  相应文件,文件夹功能介绍

### 根目录下文件夹功能介绍

-   /bin/    用以存储二进制可执行命令文件，/usr/bin/也存储了一些基于用户的命令文件。
-   /sbin/    许多系统命令的存储位置，/usr/sbin/中也包括了许多命令。
-   /root/    超级用户，即根用户的主目录。
-   /home/    普通用户的默认目录，在该目录下，每个用户拥有一个以用户名命名的文件夹。
-   /boot/    存放Ubuntu内核和系统启动文件。
-   /mnt/     通常包括系统引导后被挂载的文件系统的挂载点。
-   /dev/    存储设备文件，包括计算机的所有外部设备，如硬盘、是、键盘、鼠标等。
-   /etc/    存放文件管理配置文件和目录。
-   /lib/    存储各种程序所需要的共享库文件。
-   /lost+found/    一般为空，当非法关机时，会存放一些零散的文件。
-   /var/    用于存放很多不断变化的文件，例如日志文件等。
-   /usr/    包括与系统用户直接有关的文件和目录
-   /media/    存放Ubuntu系统自动挂载的设备文件。
-   /proc/    这是一个虚拟目录，它是内存的映射，包括系统信息和进程信息。
-   /tmp/    存储系统和用户的临时信息。
-   /initrd/    用来加载启动时临时挂载的initrd.img映像文件，以及载入所要的设备模块目录。
-   /opt/    作为可选文件和程序的存放目录，否则将无法引导计算机进入操作系统。
-   /srv/    存储系统提供的服务数据。
-   /sys/    系统设备和文件层次结构，并向用户程序提供详细的内核数据信息。

### 其他文件,文件夹功能介绍

### /etc/apt/sources.list&/etc/apt/sources.list.d/*.list



/etc/apt/sources.list.d/目录下的文件是通过sudo add-apt-repository命令安装的第三方源



文件/etc/apt/sources.list是一个普通可编辑的文本文件，保存了ubuntu软件更新的源服务器的地址。和sources.list功能一样的是/etc/apt/sources.list.d/*.list(*代表一个文件名，只能由字母、数字、下划线、英文句号组成)。sources.list.d目录下的*.list文件为在单独文件中写入源的地址提供了一种方式，通常用来安装第三方的软件。



最近在12.10上使用sudo apt-get install命令时，出现了404 not found的问题，此时ping archive.ubuntu.com可以ping通，在http://archive.ubuntu.com/ubuntu/dists/ 目录下已经没有quantal相关目录。具体原因是ubuntu对12.10的维护时间不超过一年，超过了相应的时间之后，对应的源的文件都转移到了http://old-releases.ubuntu.com/ubuntu/dists/  目录下。ubuntu发布的版本可以从 [这里](https://wiki.ubuntu.com/Releases) 看到，从中一方面可以看到ubuntu数字版本号和英文名称的对应关系，也可以看到以04结尾的版本LTS标识，标识长期维护，这些版本的源在archive.ubuntu.com中呆的时间就比较长。



### /etc/security/limits.conf

```conf
soft nproc 65535
hard nproc 65535
soft nofile 65535
hard nofile 65535
```

soft nproc: 可打开的[文件描述符](http://zhidao.baidu.com/search?word=%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6&fr=qb_search_exp&ie=utf8)的最大数(软限制)
hard nproc： 可打开的[文件描述符](http://zhidao.baidu.com/search?word=%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6&fr=qb_search_exp&ie=utf8)的最大数(硬限制)
soft nofile：单个用户可用的最大进程数量(软限制)
hard nofile：单个用户可用的最大进程数量(硬限制)



### /etc/security/limits.d/20-nproc.conf

用户进程限制

### 1212

### 1