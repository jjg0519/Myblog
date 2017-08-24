# Nginx 上传下载服务器

```bash
wget  http://nginx.org/download/nginx-1.10.2.tar.gz
wget http://www.grid.NET.ru/nginx/download/nginx_upload_module-2.2.0.tar.gz
git clone https://github.com/masterzen/nginx-upload-progress-module.git

####下载nginx_upload_module-2.2.0补丁文件,命名为davromaniak.txt
wget http://paste.davromaniak.eu/?dl=110
mv index.html\?dl\=110 davromaniak.txt

####将davromaniak.txt放入nginx_upload_module-2.2.0文件夹，打好补丁
cd nginx_upload_module-2.2.0
cp ../davromaniak.txt  ./
yum -y install patch 
patch ngx_http_upload_module.c davromaniak.txt


yum install -y zlib-devel
yum -y install pcre-devel
yyum install openssl*
#为了支持rewrite功能，我们需要安装pcre
yum install pcre* //如过你已经装了，请跳过这一步
yum -y install perl-devel perl-ExtUtils-Embed

#安装
cd /data/soft/nginx-1.10.2
CFLAGS="-Dnginx_version=7052" ./configure --prefix=/opt/nginx  --add-module=../nginx_upload_module-2.2.0/ --with-http_perl_module --add-module=../nginx-upload-progress-module/


./configure --prefix=/usr/local/nginx \
--with-http_ssl_module --with-http_spdy_module \
--with-http_stub_status_module --with-pcre


```



**举例如下（绿色代码）：** 


```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;
    autoindex on;
    autoindex_exact_size off;
    autoindex_localtime on;
...
}
```

autoindex on; 自动显示目录

autoindex_exact_size off;

默认为on，显示出文件的确切大小，单位是bytes。

改为off后，显示出文件的大概大小，单位是kB或者MB或者GB

autoindex_localtime on;

默认为off，显示的文件时间为GMT时间。

改为on后，显示的文件时间为文件的服务器时间

autoindex_exact_size on; 打开显示文件的实际大小，单位是bytes； 



## Nginx 配置开机启动

这里使用的是编写shell脚本的方式来处理

Nginx默认被安装在/usr/local/nginx

vi /etc/init.d/nginx  (输入下面的代码)

```bash


#!/bin/bash
# nginx Startup script for the Nginx HTTP Server
# it is v.0.0.2 version.
# chkconfig: - 85 15
# description: Nginx is a high-performance web and proxy server.
#              It has a lot of features, but it's not for everyone.
# processname: nginx
# pidfile: /var/run/nginx.pid
# config: /usr/local/nginx/conf/nginx.conf
nginxd=/usr/local/nginx/sbin/nginx
nginx_config=/usr/local/nginx/conf/nginx.conf
nginx_pid=/var/run/nginx.pid
RETVAL=0
prog="nginx"
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ ${NETWORKING} = "no" ] && exit 0
[ -x $nginxd ] || exit 0
# Start nginx daemons functions.
start() {
if [ -e $nginx_pid ];then
   echo "nginx already running...."
   exit 1
fi
   echo -n $"Starting $prog: "
   daemon $nginxd -c ${nginx_config}
   RETVAL=$?
   echo
   [ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
   return $RETVAL
}
# Stop nginx daemons functions.
stop() {
        echo -n $"Stopping $prog: "
        killproc $nginxd
        RETVAL=$?
        echo
        [ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /var/run/nginx.pid
}
# reload nginx service functions.
reload() {
    echo -n $"Reloading $prog: "
    #kill -HUP `cat ${nginx_pid}`
    killproc $nginxd -HUP
    RETVAL=$?
    echo
}
# See how we were called.
case "$1" in
start)
        start
        ;;
stop)
        stop
        ;;
reload)
        reload
        ;;
restart)
        stop
        start
        ;;
status)
        status $prog
        RETVAL=$?
        ;;
*)
        echo $"Usage: $prog {start|stop|restart|reload|status|help}"
        exit 1
esac
exit $RETVAL
```

