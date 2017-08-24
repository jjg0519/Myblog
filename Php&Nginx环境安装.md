# Php环境安装

## 安装

安装依赖

```bash
yum install gcc make gd-devel libjpeg-devel libpng-devel libxml2-devel bzip2-devel libcurl-devel -y
```

## 编译安装

```bash
./configure --prefix=/usr/local/php-5.6.30 --with-config-file-path=/usr/local/php--5.6.30/etc --with-bz2 --with-curl --enable-ftp --enable-sockets --disable-ipv6 --with-gd --with-jpeg-dir=/usr/local --with-png-dir=/usr/local --with-freetype-dir=/usr/local --enable-gd-native-ttf --with-iconv-dir=/usr/local --enable-mbstring --enable-calendar --with-gettext --with-libxml-dir=/usr/local --with-zlib --with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --with-mysql=mysqlnd --enable-dom --enable-xml --enable-fpm --with-libdir=lib64

./configure --prefix=/usr/local/php-5.6.30 \
--with-config-file-path=/usr/local/php-5.6.30/etc --with-bz2 --with-curl \
--enable-ftp --enable-sockets --disable-ipv6 --with-gd \
--with-jpeg-dir=/usr/local --with-png-dir=/usr/local \
--with-freetype-dir=/usr/local --enable-gd-native-ttf \
--with-iconv-dir=/usr/local --enable-mbstring --enable-calendar \
--with-gettext --with-libxml-dir=/usr/local --with-zlib \
--with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --with-mysql=mysqlnd \
--enable-dom --enable-xml --enable-fpm --with-libdir=lib64 --enable-bcmath

make&&make install
```

## 配置和启动

```bash
[root@localhost php-5.6.30]# cp php.ini-production /usr/local/php-5.6.30/etc/php.ini
[root@localhost php-5.6.30]# cp /usr/local/php-5.6.30/etc/php-fpm.conf.default /usr/local/php-5.6.30/etc/php-fpm.conf
#启动php-fpm
[root@localhost php-5.6.30]# /usr/local/php-5.6.30/sbin/php-fpm

```

## 配置成系统脚本

vim   /etc/init.d/php-fpm



```bash
#设置 php-fpm开机启动
# cp /tmp/php-5.6.11/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
# chmod +x /etc/init.d/php-fpm
# chkconfig --add php-fpm
# chkconfig php-fpm on
# service php-fpm start
```

或者

```bash
#!/bin/bash
#
# Startup script for the PHP-FPM server.
#
# chkconfig: 345 85 15
# description: PHP is an HTML-embedded scripting language
# processname: php-fpm
# config: /usr/local/php/etc/php.ini
 
# Source function library.
. /etc/rc.d/init.d/functions
 
PHP_PATH=/usr/local
DESC="php-fpm daemon"
NAME=php-fpm
# php-fpm路径
DAEMON=$PHP_PATH/php/sbin/$NAME
# 配置文件路径
CONFIGFILE=$PHP_PATH/php/etc/php-fpm.conf
# PID文件路径(在php-fpm.conf设置)
PIDFILE=$PHP_PATH/php/var/run/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
 
# Gracefully exit if the package has been removed.
test -x $DAEMON || exit 0
 
rh_start() {
  $DAEMON -y $CONFIGFILE || echo -n " already running"
}
 
rh_stop() {
  kill -QUIT `cat $PIDFILE` || echo -n " not running"
}
 
rh_reload() {
  kill -HUP `cat $PIDFILE` || echo -n " can't reload"
}
 
case "$1" in
  start)
        echo -n "Starting $DESC: $NAME"
        rh_start
        echo "."
        ;;
  stop)
        echo -n "Stopping $DESC: $NAME"
        rh_stop
        echo "."
        ;;
  reload)
        echo -n "Reloading $DESC configuration..."
        rh_reload
        echo "reloaded."
  ;;
  restart)
        echo -n "Restarting $DESC: $NAME"
        rh_stop
        sleep 1
        rh_start
        echo "."
        ;;
  *)
         echo "Usage: $SCRIPTNAME {start|stop|restart|reload}" >&2
         exit 3
        ;;
esac
exit 0
```

```bash
sudo chmod +x /etc/init.d/php-fpm
sudo /sbin/chkconfig php-fpm on




##  快捷脚本
/etc/init.d/php-fpm start
/etc/init.d/php-fpm stop
/etc/init.d/php-fpm restart
/etc/init.d/php-fpm reload

```



## 检验是否启动成功

```bash
#执行以上命令，如果没报错一般情况下表示启动正常,如果不放心,也可以通过端口判断是PHP否启动
[root@localhost php-5.6.30]# netstat -lnt | grep 9000
tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN     
[root@localhost php-5.6.30]# 
```

## 配置测试站点test.php.com

```bash
 

mkdir -p /data/logs/nginx/ # 用于存放nginx日志.请看2.3的配置文件
mkdir -p /data/site/test.php.com/ # 站点根目录
vim /data/site/test.ttlsa.com/info.php
<?php
phpinfo();
?>
```

## Nginx 配置

### 配置说明

nginx将会连接回环地址9000端口执行PHP文件,需要使用tcp/ip协议,速度比较慢.建议大家换成使用socket方式连接。将fastcgi_pass 127.0.0.1:9000;改成fastcgi_pass unix:/var/run/phpfpm.sock;
启动nginx

```nginx
 server {
                listen 70;
                server_name test.php.com;
  
                access_log /data/logs/nginx/test.php.com.access.log main;   #注意:开启了日志就要把log_format 打开

                index index.php index.html index.html;
                root /data/site/test.php.com;

                location /
                {
                        try_files $uri $uri/ /index.php?$args;
                }

                location ~ .*\.(php)?$
                {
                        expires -1s;
                        try_files $uri =404;
                        fastcgi_split_path_info ^(.+\.php)(/.+)$;
                        include fastcgi_params;
                        fastcgi_param PATH_INFO $fastcgi_path_info;
                        fastcgi_index index.php;
                        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                        fastcgi_pass 127.0.0.1:9000;
                }
        }
```

## 访问测试

```bash
#启动Nginx
# /usr/local/nginx/sbin/nginx
# curl http://test.php.com/info.php
test php
```

## 打开防火墙端口

```bash
[root@rhel7 ~]# firewall-cmd --zone=public --add-port=80/tcp --permanent
[root@rhel7 ~]# firewall-cmd --zone=public --add-port=70/tcp --permanent
[root@rhel7 ~]# firewall-cmd --reload  
```



