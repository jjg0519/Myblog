# Centos 通过http方式访问本地目录（文件服务器

## 安装apache服务器

```bash
yum install httpd
```

## 检查下载安装是否成功

```bash
[root@localhost /]# httpd -version
Server version: Apache/2.2.3
Server built:   Sep 16 2014 11:05:09
```

   安装成功后，启动apache这个时候有服务启动在80端口会报address already in use

## 修改apache的配置文件，默认是/etc/httpd/conf/httpd.conf

```bash
#修改ServerNmae为机器ip:端口
ServerName 192.168.104.157:8000
#修改http文件服务器的下载目录
DocumentRoot "/config_mam"
把原来的
    <Directory "/var/www/html"></Directory>
修改为
    <Directory "/config_mam"></Directory> 
#如果想修改相应的端口，则修改
    Listen 80 
为相应的端口号
    Listen 8000
```

## 去掉apache的欢迎页面，为下载目录的结构

```bash
把/etc/httpd/conf.d/welcome.conf删除或者备份成/etc/httpd/conf.d/welcome.conf.bak
```

## 此时已经可用

```bash
访问192.168.104.157:8000则返回了/config_mam的目录结构
```