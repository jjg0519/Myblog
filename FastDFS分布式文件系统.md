FastDFS分布式文件系统

[TOC]



![](image.1125917-20170322230555361-918559722.png)

一般来说，一台服务器只有一个storage server，多个storage server可以组成一个group，同一group间storage server的数据自动同步（备份与恢复）。

不同group数据互相隔离，一个tracker可以管理多个group，也可以多对多。

client用于管理 tracker server 和storage server。



## fastdfs配置安装

机器A: **192.168.1.106**

机器B: **192.168.1.107**



### 安装软件

#### 软件包

```bash
fastdfs-5.05.tar.gz
libfastcommon-1.0.36.tar.gz
nginx-1.11.3.tar.gz
fastdfs-nginx-module_v1.16.tar.gz
```

#### 安装前环境依赖

```bash
[root@shj-233 software]# yum -y install  gcc
[root@shj-233 software]# yum -y install  perl

[root@shj-233 software]# yum -y install pcre-devel
[root@shj-233 software]#yum install -y zlib-devel
```

#### 依赖库libfastcommon

```bash
[root@shj-233 software]# tar -zxvf libfastcommon-1.0.36.tar.gz 
[root@shj-233 software]# cd libfastcommon-1.0.36 
[root@shj-233 libfastcommon-1.0.36]# ./make.sh  
[root@shj-233 libfastcommon-1.0.36]# ./make.sh install  
```

libfastcommon 默认安装到了

    /usr/lib64/libfastcommon.so  
    /usr/lib64/libfdfsclient.so  
因为 FastDFS 主程序设置的 lib 目录是/usr/local/lib， 所以需要创建软链接.

```bash
# ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so  
# ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so  
# ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so  
# ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so  
```



####安装fastdfs

```bash
[root@shj-233 software]# unzip fastdfs-5.09.zip  
[root@shj-233 software]# cd fastdfs-5.09  
[root@shj-233 FastDFS]#./make.sh  
[root@shj-233 FastDFS]#./make.sh install  
[root@shj-233 FastDFS]#cp conf/http.conf /etc/fdfs/  
[root@shj-233 FastDFS]#cp conf/mime.types /etc/fdfs/  
[root@shj-233 FastDFS]# cp conf/storage_ids.conf /etc/fdfs/  
[root@shj-233 FastDFS]# ls /usr/bin/fdfs_*  
[root@shj-233 FastDFS]# ls /etc/fdfs/
client.conf.sample  http.conf  mime.types  storage.conf.sample  tracker.conf.sample

```

>安装完毕后，执行进程默认都在/usr/bin目录中
>
>安装完毕后，启动的配置文件默认都在**/etc/fdfs**中。



####安装nginx fastdfs moudel（用于下载文件)

```bash
[root@shj-233 software]# tar -xzvf nginx-1.11.3.tar.gz  
[root@shj-233 software]# tar -zxvf fastdfs-nginx-module_v1.10.tar.gz
[root@shj-233 software]# mkdir /usr/local/nginx-1.11.3  
[root@shj-233 software]# cd nginx-1.11.3  
[root@shj-233 software]# ./configure --prefix=/usr/local/nginx-1.11.3 --add-module=/opt/software/fastdfs-nginx-module/src

[root@shj-233 software]#  make&make install 
     
```

fastdfs的文件名包含了下载路径，所以可以通过nginx模块进行静态下载，而不需要通过tracker服务。

```bash
[root@shj-233 software]# cp fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/  
```



#### 启动fastdfs前的一些目录准备

```bash
mkdir /fdfs0 #文件存储路径0  
mkdir /fdfs1 #文件存储路径1  

mkdir /fastdfs/tracker #tracker服务的日志和一些数据存储  
mkdir /fastdfs/storage #storage服务的日志和一些数据存储（非文件数据）  

mkdir /fastdfs/mod_fastdfs #fastdfs nginx模块  
mkdir /fastdfs/mod_fastdfs/logs #fastdfs nginx模块日志  

mkdir /fastdfs/client #测试客户端的日志，备用  
```

#### 防火墙开启

```bash
[root@shj-234 opt]#vim /etc/sysconfig/iptables
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22122 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 23000 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
[root@shj-234 opt]#service iptables restart

#tracker
iptables -I INPUT -p tcp -m state --state NEW -m tcp --dport22122 -j ACCEPT
#storage
iptables -I INPUT -p tcp -m state --state NEW -m tcp --dport23000 -j ACCEPT
#nginx
iptables -I INPUT -p tcp -m state --state NEW -m tcp --dport80 -j ACCEPT
```



#### 配置文件介绍

```bash
[root@shj-233 software]# ll /etc/fdfs/
total 60
-rw-r--r--. 1 root root  1461 May 29 14:29 client.conf.sample
-rw-r--r--. 1 root root   858 May 29 14:30 http.conf
-rw-r--r--. 1 root root 31172 May 29 14:30 mime.types
-rw-r--r--. 1 root root  3679 May 29 14:58 mod_fastdfs.conf
-rw-r--r--. 1 root root  7829 May 29 14:29 storage.conf.sample
-rw-r--r--. 1 root root  7102 May 29 14:29 tracker.conf.sample


tracker.conf     #tracker服务依赖的配置  
storage_ids.conf #tracker服务依赖的配置  
storage.conf     #storage服务依赖的配置  
http.conf        #nginx模块依赖的配置  
mime.types       #nginx模块依赖的配置  
mod_fastdfs.conf #nginx模块依赖的配置  
client.conf      #测试客户依赖的配置  
```

#####配置tracker的配置

```bash
[root@shj-233 fdfs]# pwd
/etc/fdfs
[root@shj-233 fdfs]#cp tracker.conf.sample tracker.conf  
[root@shj-233 fdfs]#vim tracker.conf  
#修改配置项如下（A/B机器只需要更换bind_addr）  
bind_addr=192.168.1.106  
port=22122  
base_path=/fastdfs/tracker  
use_storage_id = true  
storage_ids_filename = /etc/fdfs/storage_ids.conf  
id_type_in_filename = id  


[root@shj-233 fdfs]#cp /opt/software/FastDFS/conf/storage_ids.conf ./
[root@shj-233 fdfs]#cp storage_ids.conf.sample storage_ids.conf  
[root@shj-233 fdfs]#vim storage_ids.conf  
#修改配置项如下（A/B机器一致）  
# <id>  <group_name>  <ip_or_hostname>  
100001   group1  192.168.1.106  
100002   group1  192.168.1.107  
```

##### 启动tracker

```bash
启动服务：fdfs_trackerd /etc/fdfs/tracker.conf start

重启服务：fdfs_trackerd /etc/fdfs/tracker.conf restart

停止服务：fdfs_trackerd /etc/fdfs/tracker.conf stop
```

#####配置storage的配置

```bash
[root@shj-233 fdfs]#cp storage.conf.sample storage.conf  
[root@shj-233 fdfs]#vim storage.conf  
#修改配置项如下（A/B机器只需要更换bind_addr）  
group_name=group1  
bind_addr=192.168.1.233  
port=23000  
base_path=/fastdfs/storage  
store_path_count=2  
store_path0=/fdfs0  
store_path1=/fdfs1  
tracker_server=192.168.1.233:22122  
tracker_server=192.168.1.234:22122
```



##### 启动storage

```bash
启动服务：fdfs_storaged /etc/fdfs/storage.conf start

重启服务：fdfs_storaged /etc/fdfs/storage.conf restart

停止服务：fdfs_storaged /etc/fdfs/storage.conf stop
```

##### 检查集群是否都处于active状态

```bash
[root@shj-233 fdfs]#/usr/bin/fdfs_monitor /etc/fdfs/storage.conf
```



##### 文件上传测试

```bash
[root@shj-233 fdfs]#[root@shj-233 fdfs]#cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf
vim /etc/fdfs/client.conf
# 修改以下配置，其它保持默认
base_path=/fastdfs/client
tracker_server=192.168.1.233:22122

#./111111.jpg  是需要上传文件路径
[root@shj-233 nginx]# /usr/bin/fdfs_upload_file /etc/fdfs/client.conf ./111111.jpg 
group1/M00/00/00/ooYBAFsOSCSAaflmAAtqkDIr08w747.jpg

#返回文件ID号：group1/M00/00/00/ooYBAFsOSCSAaflmAAtqkDIr08w747.jpg

#（能返回以上文件ID，说明文件已经上传成功）
```



##### 配置nginx fastdfs 

```bash
[root@shj-233 fdfs]#vim mod_fastdfs.conf  
#修改配置项如下  
base_path=/fastdfs/mod_fastdfs  
load_fdfs_parameters_from_tracker=true  
use_storage_id = true  
storage_ids_filename = /etc/fdfs/storage_ids.conf  
tracker_server=192.168.1.233:22122  
tracker_server=192.168.1.234:22122  
group_name=group1  
url_have_group_name = true  
store_path_count=2  
store_path0=/fdfs0  
store_path1=/fdfs1  
log_filename=/fastdfs/mod_fastdfs/logs/mod_fastdfs.log  
response_mode=proxy  
group_count = 0  
```

##### 配置nginx

```bash
[root@shj-233 fdfs]#mkdir /fastdfs/nginx  
[root@shj-233 fdfs]#mkdir /fastdfs/nginx/conf  
[root@shj-233 fdfs]#mkdir /fastdfs/nginx/logs  

[root@shj-233 fdfs]#cd /fastdfs/nginx/conf  
[root@shj-233 nginx]#vim fastdfs.proxy.conf  
#修改配置项如下  
user root;
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include mime.types;
     default_type  application/octet-stream;
    sendfile on;
    keepalive_timeout 65;
    log_format main '$msec $status $request $request_time '
    '$http_referer $remote_addr [ $time_local ] '
    '$upstream_response_time $host $bytes_sent '
    '$request_length $upstream_addr';

    server {
        listen 8080
        location~/group([0-9])/M00 {
            ngx_fastdfs_module;
        }
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root html;
    }
}
```

##### 启动nginx

```bash
测试配置：nginx -p `pwd` -c conf/fastdfs.proxy.conf -t
[root@shj-233 fdfs]#/usr/local/nginx-1.11.3/sbin/nginx -c /fastdfs/nginx/conf/fastdfs.proxy.conf -t

启动：nginx -p `pwd` -c conf/fastdfs.proxy.conf

[root@shj-233 fdfs]# /usr/local/nginx-1.11.3/sbin/nginx -c /fastdfs/nginx/conf/fastdfs.proxy.conf 
ngx_http_fastdfs_set pid=14274

```

#####测试下载

```bash
curl -i "192.168.1.233:8080/group1/M00/00/00/ooYBAFsOSCSAaflmAAtqkDIr08w747.jpg
"
http://192.168.1.233:8080/group1/M00/00/00/ooYBAFsOSCSAaflmAAtqkDIr08w747.jpg

```

##nginx-lua-fastdfs-GraphicsMagick 动态生成缩略图

fastdfs开源的分布式文件系统，此脚本利用nginx lua模块，动态生成图片缩略图，fastdfs只存一份原图。lua通过socket获取fastdfs的原图，并存放到本地，根据不同规则url，例如：_60x60.jpg、_80x80.jpg，类似淘宝图片url规则。利用gm命令生成本地缩略图，第二次访问直接返回本地图片。定时任务凌晨清除7天内未访问的图片，节省空间。

###图片访问举例

1. <http://192.168.1.113/group1/M00/00/00/wKgBcVN0wDiAILQXAAdtg6qArdU189.jpg>
2. <http://192.168.1.113/group1/M00/00/00/wKgBcVN0wDiAILQXAAdtg6qArdU189.jpg_80x80.jpg>
3. <http://gi1.md.alicdn.com/imgextra/i1/401612253/T2ASPfXE4XXXXXXXXX_!!401612253.jpg_60x60.jpg>
4. <http://gi1.md.alicdn.com/imgextra/i1/401612253/T2ASPfXE4XXXXXXXXX_!!401612253.jpg_80x80.jpg>

###安装lua和nginx development kit模块

```bash
[root@shj-233 software]#wget http://luajit.org/download/LuaJIT-2.0.5.tar.gz
[root@shj-233 software]#tar -zxvf LuaJIT-2.0.5.tar.gz 
[root@shj-233 software]#cd LuaJIT-2.0.5
[root@shj-233 LuaJIT-2.0.5]#make install PREFIX=/usr/local/luajit
[root@shj-233 LuaJIT-2.0.5]#vim /etc/profile
[root@shj-233 LuaJIT-2.0.5]#export LUAJIT_LIB=/usr/local/luajit/lib/
[root@shj-233 LuaJIT-2.0.5]#export LUAJIT_INC=/usr/local/luajit/include/luajit-2.0/
[root@shj-233 LuaJIT-2.0.5]# source /etc/profile

```



```bash
[root@shj-233 software]#wget https://github.com/simpl/ngx_devel_kit/archive/v0.2.18.tar.gz
[root@shj-233 software]#wget https://codeload.github.com/openresty/lua-nginx-module/tar.gz/v0.10.13

#查看已安装的nginx 组件
[root@shj-233 nginx-1.11.3]# /usr/local/nginx-1.11.3/sbin/nginx -V
nginx version: nginx/1.11.3
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-18) (GCC) 
configure arguments: --prefix=/usr/local/nginx-1.11.3 --add-module=/opt/software/fastdfs-nginx-module/src

##安装新的nginx 组件(包含原有的组件) 
[root@shj-233 nginx-1.11.3]#./configure --prefix=/usr/local/nginx-1.11.3 --add-module=/opt/software/fastdfs-nginx-module/src --add-module=../ngx_devel_kit-0.2.18/ --add-module=../lua-nginx-module-0.10.13/
[root@shj-233 nginx-1.11.3]#make &&make install 


#查看已安装的nginx 组件,发现找不到libluajit-5.1.so.2
[root@shj-233 nginx-1.11.3]# /usr/local/nginx-1.11.3/sbin/nginx -V
[root@shj-233 nginx-1.11.3]# l /usr/local/nginx-1.11.3/sbin/nginx -V
/usr/local/nginx-1.11.3/sbin/nginx: error while loading shared libraries: libluajit-5.1.so.2: cannot open shared object file: No such file or directory

#在ld.so.conf加入lua的so库
[root@shj-233 nginx-1.11.3]# vim /etc/ld.so.conf
include ld.so.conf.d/*.conf
/usr/local/luajit/lib
[root@shj-233 nginx-1.11.3]# ldconfig -v
##再次检查
[root@shj-233 nginx-1.11.3]#  /usr/local/nginx-1.11.3/sbin/nginx -V
nginx version: nginx/1.11.3
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-18) (GCC) 
configure arguments: --prefix=/usr/local/nginx-1.11.3 --add-module=/opt/software/fastdfs-nginx-module/src --add-module=../ngx_devel_kit-0.2.18/ --add-module=../lua-nginx-module-0.10.13/
[root@shj-233 nginx-1.11.3]# 


###重启nginx
[root@shj-233 nginx-1.11.3]# /usr/local/nginx-1.11.3/sbin/nginx -c /fastdfs/nginx/conf/fastdfs.proxy.conf -s reload


###测试lua是否成功
[root@shj-233 nginx-1.11.3]#vim /fastdfs/nginx/conf/fastdfs.proxy.conf
#加入
server{
        listen 8000;
             location ~/lua {
                default_type 'text/plain';
                content_by_lua 'ngx.say("hello, lua")';
        }
}

[root@shj-233 nginx-1.11.3]# /usr/local/nginx-1.11.3/sbin/nginx -c /fastdfs/nginx/conf/fastdfs.proxy.conf -t
[root@shj-233 nginx-1.11.3]#/usr/local/nginx-1.11.3/sbin/nginx -c /fastdfs/nginx/conf/fastdfs.proxy.conf -s reload

##访问http://localhost:8080/lua
#会出现“hello,lua”
#安装成功!
```

### 安装GraphicsMagick

#### 安装libjpeg(支持jpg)依赖

```bash

http://www.imagemagick.org/download/delegates/
##或者
yum install -y libjpeg-devel libjpeg
./configure -prefix=/usr/local/GraphicsMagick-1.3.25/
make
make install
```

#### 安装libpng(支持png)依赖

```bash
wget http://www.imagemagick.org/download/delegates/libpng-1.6.21.tar.gz
##或者
yum install -y libpng-devel libpng
```

####安装GraphicsMagick

```bash
[root@shj-233 software]# wget https://astuteinternet.dl.sourceforge.net/project/graphicsmagick/graphicsmagick/1.3.29/GraphicsMagick-1.3.29.tar.gz
[root@shj-233 GraphicsMagick-1.3.29]#./configure -prefix=/usr/local/GraphicsMagick-1.3.29
[root@shj-233 GraphicsMagick-1.3.29]#make&make install

##验证是否安装成功
[root@shj-233 GraphicsMagick-1.3.29]# /usr/local/GraphicsMagick-1.3.29/bin/gm -version
GraphicsMagick 1.3.29 2018-04-29 Q8 http://www.GraphicsMagick.org/
Copyright (C) 2002-2018 GraphicsMagick Group.
Additional copyrights and licenses apply to this software.
See http://www.GraphicsMagick.org/www/Copyright.html for details.

Feature Support:
  Native Thread Safe       yes
  Large Files (> 32 bit)   yes
  Large Memory (> 32 bit)  yes
  BZIP                     no
  DPS                      no
  FlashPix                 no
  FreeType                 no
  Ghostscript (Library)    no
  JBIG                     no
  JPEG-2000                no
  JPEG                     yes
  Little CMS               no
  Loadable Modules         no
  OpenMP                   yes (200805)
  PNG                      yes
  TIFF                     no
  TRIO                     no
  UMEM                     no
  WebP                     no
  WMF                      no
  X11                      no
  XML                      no
  ZLIB                     yes


Host type: x86_64-unknown-linux-gnu

Configured using the command:
  ./configure  '-prefix=/usr/local/GraphicsMagick-1.3.29'

Final Build Parameters:
  CC       = gcc -std=gnu99
  CFLAGS   = -fopenmp -g -O2 -Wall -pthread
  CPPFLAGS = 
  CXX      = g++
  CXXFLAGS = -pthread
  LDFLAGS  = 
  LIBS     = -lz -lm -lgomp -lpthread
[root@shj-233 GraphicsMagick-1.3.29]# 

#在执行完上述命令后会有一段输出，可以查看GraphicsMagick支持的图片格式，
#在Configured value下为yes的表示为支持，PNG、JPEG v1和ZLIB必须为yes，若不为yes将按照前提中所写的进行操作，然后再重复执行上述命令，一直到全部支持为止，否则将无法正常进行截图操作
```

#### 配置GraphicsMagick

```bash
##配置GraphicsMagick环境变量
[root@shj-233 nginx]#vi /etc/profile 
export GMAGICK_HOME="/usr/local/GraphicsMagick-1.3.29/" 
export PATH="$GMAGICK_HOME/bin:$PATH" 
LD_LIBRARY_PATH=$GMAGICK_HOME/lib:$LD_LIBRARY_PATH 
export LD_LIBRARY_PATH 
[root@shj-233 nginx]#source /etc/profile  生效配置环境

##测试gm
[root@shj-233 nginx]# /usr/local/GraphicsMagick-1.3.29/bin/gm convert -resize 100x80^ -gravity Center -crop 100x80+0+0 111111.jpg thumb.jpg
[root@shj-233 nginx]# ll
total 768
-rw-r--r--. 1 root root 748176 May 30 11:37 111111.jpg
drwxr-xr-x. 2 root root   4096 May 31 11:27 conf
drwxr-xr-x. 2 root root   4096 May 30 11:16 logs
-rw-r--r--. 1 root root  24768 May 31 11:32 thumb.jpg

```

###配置nginx

```bash
[root@shj-233 nginx]# vim fastdfs.proxy.conf
user root root;  
worker_processes 1;  

events {  
worker_connections 1024;  
}  

http {  
include       /usr/local/nginx-1.11.3/conf/mime.types;
default_type application/octet-stream;

log_format  main  '$msec $status $request $request_time '  
'$http_referer $remote_addr [ $time_local ] '  
'$upstream_response_time $host $bytes_sent '  
'$request_length $upstream_addr';  

sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
keepalive_timeout 65;

lua_package_path "/fastdfs/nginx/conf/lua/?.lua;;";

 server {
    	listen 80;
        location / {
            default_type text/html;
            content_by_lua '
                ngx.say("<p>hello, world</p>")
                ';
        }

        location /hello {
            default_type text/html;
        }

        location /group1/M00 {
            alias /data/images;

            #set $image_root "/usr/local/openresty/nginx/proxy_tmp/images";
            set $image_root "/data/images";
            if ($uri ~ "/([a-zA-Z0-9]+)/([a-zA-Z0-9]+)/([a-zA-Z0-9]+)/([a-zA-Z0-9]+)/(.*)") {
                set $image_dir "$image_root/$3/$4/";
                set $image_name "$5";
                set $file "$image_dir$image_name";
            }

            if (!-f $file) {
                # 关闭lua代码缓存，方便调试lua脚本
                #lua_code_cache off;
                content_by_lua_file "/fastdfs/nginx/conf/lua/fastdfs.lua";
            }

            #ngx_fastdfs_module;
        }
    }
server{
	listen 8000;
	     location ~/lua {
                default_type 'text/plain';
                content_by_lua 'ngx.say("hello, lua")';
        }
}
} 

```

### lua 脚本 

fastdfs.lua
restyfastdfs.lua

####fastdfs.lua

```bash
-- 写入文件
local function writefile(filename, info)
    local wfile=io.open(filename, "w") --写入文件(w覆盖)
    assert(wfile)  --打开时验证是否出错		
    wfile:write(info)  --写入传入的内容
    wfile:close()  --调用结束后记得关闭
end

-- 检测路径是否目录
local function is_dir(sPath)
    if type(sPath) ~= "string" then return false end

    local response = os.execute( "cd " .. sPath )
    if response == 0 then
        return true
    end
    return false
end

-- 检测文件是否存在
local file_exists = function(name)
    local f=io.open(name,"r")
    if f~=nil then io.close(f) return true else return false end
end

local area = nil
local originalUri = ngx.var.uri;
local originalFile = ngx.var.file;
local index = string.find(ngx.var.uri, "([0-9]+)x([0-9]+)");  
if index then 
    originalUri = string.sub(ngx.var.uri, 0, index-2);  
    area = string.sub(ngx.var.uri, index);  
    index = string.find(area, "([.])");  
    area = string.sub(area, 0, index-1);  

    local index = string.find(originalFile, "([0-9]+)x([0-9]+)");  
    originalFile = string.sub(originalFile, 0, index-2)
	    print( "111111222");
 ngx.log(ngx.ERR,"1111111111");

end

-- check original file
if not file_exists(originalFile) then
    local fileid = string.sub(originalUri, 2);
    -- main
    local fastdfs = require('restyfastdfs')
    local fdfs = fastdfs:new()
    fdfs:set_tracker("192.168.1.233", 22122)
    fdfs:set_timeout(1000)
    fdfs:set_tracker_keepalive(0, 100)
    fdfs:set_storage_keepalive(0, 100)
    local data = fdfs:do_download(fileid)
    if data then
       -- check image dir
        if not is_dir(ngx.var.image_dir) then
            os.execute("mkdir -p " .. ngx.var.image_dir)
        end
        writefile(originalFile, data)
    end
end

-- 创建缩略图
local image_sizes = {"80x80", "800x600", "40x40", "60x60"};  
function table.contains(table, element)  
    for _, value in pairs(table) do  
        if value == element then
            return true  
        end  
    end  
    return false  
end 

if table.contains(image_sizes, area) then  
    print( "111111222");
 ngx.log(ngx.ERR,"1111111111"); 
  local command = "/usr/local/GraphicsMagick-1.3.29/bin/gm convert " .. originalFile  .. " -thumbnail " .. area .. " -background gray -gravity center -extent " .. area .. " " .. ngx.var.file;  
    os.execute(command);  
   print( "111111");
ngx.log(ngx.ERR,"3333");

end;

if file_exists(ngx.var.file) then
    --ngx.req.set_uri(ngx.var.uri, true);  
    ngx.log(ngx.ERR,"---------");
    ngx.log(ngx.ERR,ngx.var.uri);
    ngx.exec(ngx.var.uri)
else
    ngx.log(ngx.ERR,"--22-------");
    ngx.log(ngx.ERR,ngx.var.uri);
 
    ngx.exit(404)
end

```

####restyfastdfs.lua

```lua
-- Copyright (C) 2012 Azure Wang
-- @link: https://github.com/azurewang/Nginx_Lua-FastDFS

local string = string
local table  = table
local bit    = bit
local ngx    = ngx
local tonumber = tonumber
local setmetatable = setmetatable
local error = error

module(...)

local VERSION = '0.1'

local FDFS_PROTO_PKG_LEN_SIZE = 8
local TRACKER_PROTO_CMD_SERVICE_QUERY_STORE_WITHOUT_GROUP_ONE = 101
local TRACKER_PROTO_CMD_SERVICE_QUERY_STORE_WITH_GROUP_ONE = 104
local TRACKER_PROTO_CMD_SERVICE_QUERY_UPDATE = 103
local TRACKER_PROTO_CMD_SERVICE_QUERY_FETCH_ONE = 102
local STORAGE_PROTO_CMD_UPLOAD_FILE = 11
local STORAGE_PROTO_CMD_DELETE_FILE = 12
local STORAGE_PROTO_CMD_DOWNLOAD_FILE = 14
local STORAGE_PROTO_CMD_UPLOAD_SLAVE_FILE = 21
local STORAGE_PROTO_CMD_QUERY_FILE_INFO = 22
local STORAGE_PROTO_CMD_UPLOAD_APPENDER_FILE = 23
local STORAGE_PROTO_CMD_APPEND_FILE = 24
local FDFS_FILE_EXT_NAME_MAX_LEN = 6
local FDFS_PROTO_CMD_QUIT = 82
local TRACKER_PROTO_CMD_RESP = 100

local mt = { __index = _M }

function new(self)
    return setmetatable({}, mt)
end

function set_tracker(self, host, port)
    local tracker = {host = host, port = port}
    self.tracker = tracker
end

function set_timeout(self, timeout)
    if timeout then
        self.timeout = timeout
    end
end

function set_tracker_keepalive(self, timeout, size)
    local keepalive = {timeout = timeout, size = size}
    self.tracker_keepalive = keepalive
end

function set_storage_keepalive(self, timeout, size)
    local keepalive = {timeout = timeout, size = size}
    self.storage_keepalive = keepalive
end

function int2buf(n)
    -- only trans 32bit  full is 64bit
    return string.rep("\00", 4) .. string.char(bit.band(bit.rshift(n, 24), 0xff), bit.band(bit.rshift(n, 16), 0xff), bit.band(bit.rshift(n, 8), 0xff), bit.band(n, 0xff))
end

function buf2int(buf)
    -- only trans 32bit  full is 64bit
    local c1, c2, c3, c4 = string.byte(buf, 5, 8)
    return bit.bor(bit.lshift(c1, 24), bit.lshift(c2, 16),bit.lshift(c3, 8), c4)
end

function read_fdfs_header(sock)
    local header = {}
    local buf, err = sock:receive(10)
    if not buf then
        ngx.log(ngx.ERR, "fdfs: read header error")
        sock:close()
        ngx.exit(500)
    end
    header.len = buf2int(string.sub(buf, 1, 8))
    header.cmd = string.byte(buf, 9)
    header.status = string.byte(buf, 10)
    return header
end

function fix_string(str, fix_length)
    local len = string.len(str)
    if len > fix_length then
        len = fix_length
    end
    local fix_str = string.sub(str, 1, len)
    if len < fix_length then
        fix_str = fix_str .. string.rep("\00", fix_length - len )
    end
    return fix_str
end

function strip_string(str)
    local pos = string.find(str, "\00")
    if pos then
        return string.sub(str, 1, pos - 1)
    else
        return str
    end
end

function get_ext_name(filename)
    local extname = filename:match("%.(%w+)$")
    if extname then
        return fix_string(extname, FDFS_FILE_EXT_NAME_MAX_LEN)
    else
        return nil
    end
end

function read_tracket_result(sock, header)
    if header.len > 0 then
        local res = {}
        local buf = sock:receive(header.len)
        res.group_name = strip_string(string.sub(buf, 1, 16))
        res.host       = strip_string(string.sub(buf, 17, 31)) 
        res.port       = buf2int(string.sub(buf, 32, 39))
        res.store_path_index = string.byte(string.sub(buf, 40, 40))
        return res
    else
        return nil
    end
end

function read_storage_result(sock, header)
    if header.len > 0 then
        local res = {}
        local buf = sock:receive(header.len)
        res.group_name = strip_string(string.sub(buf, 1, 16))
        res.file_name  = strip_string(string.sub(buf, 17, header.len))
        return res
    else
        return nil
    end
end

function query_upload_storage(self, group_name)
    local tracker = self.tracker
    if not tracker then
        return nil
    end
    local out = {}
    if group_name then
        -- query upload with group_name
        -- package length
        table.insert(out, int2buf(16))
        -- cmd
        table.insert(out, string.char(TRACKER_PROTO_CMD_SERVICE_QUERY_STORE_WITH_GROUP_ONE))
        -- status
        table.insert(out, "\00")
        -- group name
        table.insert(out, fix_string(group_name, 16))
    else
        -- query upload without group_name
        -- package length
        table.insert(out,  string.rep("\00", FDFS_PROTO_PKG_LEN_SIZE))
        -- cmd
        table.insert(out, string.char(TRACKER_PROTO_CMD_SERVICE_QUERY_STORE_WITHOUT_GROUP_ONE))
        -- status
        table.insert(out, "\00")
    end
    -- init socket
    local sock, err = ngx.socket.tcp()
    if not sock then
        return nil, err
    end
    if self.timeout then
        sock:settimeout(self.timeout)
    end
    -- connect tracker
    local ok, err = sock:connect(tracker.host, tracker.port)
    if not ok then
        return nil, err
    end
    -- send request
    local bytes, err = sock:send(out)
    -- read request header
    local hdr = read_fdfs_header(sock)
    -- read body
    local res = read_tracket_result(sock, hdr)
    -- keepalive
    local keepalive = self.tracker_keepalive
    if keepalive then
        sock:setkeepalive(keepalive.timeout, keepalive.size)
    end
    return res
end

function do_upload_appender(self, ext_name)
    local storage = self:query_upload_storage()
    if not storage then
        return nil
    end
    -- ext_name
    if ext_name then
        ext_name = fix_string(ext_name, FDFS_FILE_EXT_NAME_MAX_LEN)
    end
    -- get file size
    local file_size = tonumber(ngx.var.content_length)
    if not file_size or file_size <= 0 then
        return nil
    end
    local sock, err = ngx.socket.tcp()
    if not sock then
        return nil, err
    end
    if self.timeout then
        sock:settimeout(self.timeout)
    end
    local ok, err = sock:connect(storage.host, storage.port)
    if not ok then
        return nil, err
    end
    -- send header
    local out = {}
    table.insert(out, int2buf(file_size + 15))
    table.insert(out, string.char(STORAGE_PROTO_CMD_UPLOAD_APPENDER_FILE))
    -- status
    table.insert(out, "\00")
    -- store_path_index
    table.insert(out, string.char(storage.store_path_index))
    -- filesize
    table.insert(out, int2buf(file_size))
    -- exitname
    table.insert(out, ext_name)
    local bytes, err = sock:send(out)
    -- send file data
    local send_count = 0
    local req_sock, err = ngx.req.socket()
    if not req_sock then
        ngx.log(ngx.ERR, err)
        ngx.exit(500)
    end
        while true do
        local chunk, _, part = req_sock:receive(1024 * 32)
        if not part then
            local bytes, err = sock:send(chunk)
            if not bytes then
                ngx.log(ngx.ngx.ERR, "fdfs: send body error")
                sock:close()
                ngx.exit(500)
            end
            send_count = send_count + bytes
        else
            -- part have data, not read full end
            local bytes, err = sock:send(part)
            if not bytes then
                ngx.log(ngx.ngx.ERR, "fdfs: send body error")
                sock:close()
                ngx.exit(500)
            end
            send_count = send_count + bytes
            break
        end
    end
    if send_count ~= file_size then
        -- send file not full
        ngx.log(ngx.ngx.ERR, "fdfs: read file body not full")
        sock:close()
        ngx.exit(500)
    end
    -- read response
    local res_hdr = read_fdfs_header(sock)
    local res = read_storage_result(sock, res_hdr)
    local keepalive = self.storage_keepalive
    if keepalive then
        sock:setkeepalive(keepalive.timeout, keepalive.size)
    end
    return res
end

function do_upload(self, ext_name)
    local storage = self:query_upload_storage()
    if not storage then
        return nil
    end
    -- ext_name
    if ext_name then
        ext_name = fix_string(ext_name, FDFS_FILE_EXT_NAME_MAX_LEN)
    end
    -- get file size
    local file_size = tonumber(ngx.var.content_length)
    if not file_size or file_size <= 0 then
        return nil
    end
    local sock, err = ngx.socket.tcp()
    if not sock then
        return nil, err
    end
    if self.timeout then
        sock:settimeout(self.timeout)
    end
    local ok, err = sock:connect(storage.host, storage.port)
    if not ok then
        return nil, err
    end
    -- send header
    local out = {}
    table.insert(out, int2buf(file_size + 15))
    table.insert(out, string.char(STORAGE_PROTO_CMD_UPLOAD_FILE))
    -- status
    table.insert(out, "\00")
    -- store_path_index
    table.insert(out, string.char(storage.store_path_index))
    -- filesize
    table.insert(out, int2buf(file_size))
    -- exitname
    table.insert(out, ext_name)
    local bytes, err = sock:send(out)
    -- send file data
    local send_count = 0
    local req_sock, err = ngx.req.socket()
    if not req_sock then
        ngx.log(ngx.ERR, err)
        ngx.exit(500)
    end
    while true do
        local chunk, _, part = req_sock:receive(1024 * 32)
        if not part then
            local bytes, err = sock:send(chunk)
            if not bytes then
                ngx.log(ngx.ngx.ERR, "fdfs: send body error")
                sock:close()
                ngx.exit(500)
            end
            send_count = send_count + bytes
        else
            -- part have data, not read full end
            local bytes, err = sock:send(part)
            if not bytes then
                ngx.log(ngx.ngx.ERR, "fdfs: send body error")
                sock:close()
                ngx.exit(500)
            end
            send_count = send_count + bytes
            break
        end
    end
    if send_count ~= file_size then
        -- send file not full
        ngx.log(ngx.ngx.ERR, "fdfs: read file body not full")
        sock:close()
        ngx.exit(500)
    end
    -- read response
    local res_hdr = read_fdfs_header(sock)
    local res = read_storage_result(sock, res_hdr)
    local keepalive = self.storage_keepalive
    if keepalive then
        sock:setkeepalive(keepalive.timeout, keepalive.size)
    end
    return res
end

function query_update_storage_ex(self, group_name, file_name)
    local out = {}
    -- package length
    table.insert(out, int2buf(16 + string.len(file_name)))
    -- cmd
    table.insert(out, string.char(TRACKER_PROTO_CMD_SERVICE_QUERY_UPDATE))
    -- status
    table.insert(out, "\00")
    -- group_name
    table.insert(out, fix_string(group_name, 16))
    -- file name
    table.insert(out, file_name)
    -- get tracker
    local tracker = self.tracker
    if not tracker then
        return nil
    end
    -- init socket
    local sock, err = ngx.socket.tcp()
    if not sock then
        return nil, err
    end
    if self.timeout then
        sock:settimeout(self.timeout)
    end
    -- connect tracker
    local ok, err = sock:connect(tracker.host, tracker.port)
    if not ok then
        return nil, err
    end
    -- send request
    local bytes, err = sock:send(out)
    -- read request header
    local hdr = read_fdfs_header(sock)
    -- read body
    local res = read_tracket_result(sock, hdr)
    -- keepalive
    local keepalive = self.tracker_keepalive
    if keepalive then
        sock:setkeepalive(keepalive.timeout, keepalive.size)
    end
    return res
end

function query_update_storage(self, fileid)
    local pos = fileid:find('/')
    if not pos then
        return nil
    else
        local group_name = fileid:sub(1, pos-1)
        local file_name  = fileid:sub(pos + 1)
        local res = self:query_update_storage_ex(group_name, file_name)
        if res then
            res.file_name = file_name
        end
        return res
    end
end

function do_delete(self, fileid)
    local storage = self:query_update_storage(fileid)
    if not storage then
        return nil
    end
    local out = {}
    table.insert(out, int2buf(16 + string.len(storage.file_name)))
    table.insert(out, string.char(STORAGE_PROTO_CMD_DELETE_FILE))
    table.insert(out, "\00")
    -- group name
    table.insert(out, fix_string(storage.group_name, 16))
    -- file name
    table.insert(out, storage.file_name)
    -- init socket
    local sock, err = ngx.socket.tcp()
    if not sock then
        return nil, err
    end
    sock:settimeout(self.timeout)
    local ok, err = sock:connect(storage.host, storage.port)
    if not ok then
        return nil, err
    end
    local bytes, err = sock:send(out)
    if not bytes then
        ngx.log(ngx.ngx.ERR, "fdfs: send body error")
        sock:close()
        ngx.exit(500)
    end
    -- read request header
    local hdr = read_fdfs_header(sock)
    local keepalive = self.storage_keepalive
    if keepalive then
        sock:setkeepalive(keepalive.timeout, keepalive.size)
    end
    return hdr
end

function query_download_storage(self, fileid)
    local pos = fileid:find('/')
    if not pos then
        return nil
    else
        local group_name = fileid:sub(1, pos-1)
        local file_name  = fileid:sub(pos + 1)
        local res = self:query_download_storage_ex(group_name, file_name)
        res.file_name = file_name
        return res
    end
end

function query_download_storage_ex(self, group_name, file_name)
    local out = {}
    -- package length
    table.insert(out, int2buf(16 + string.len(file_name)))
    -- cmd
    table.insert(out, string.char(TRACKER_PROTO_CMD_SERVICE_QUERY_FETCH_ONE))
    -- status
    table.insert(out, "\00")
    -- group_name
    table.insert(out, fix_string(group_name, 16))
    -- file name
    table.insert(out, file_name)
    -- get tracker
    local tracker = self.tracker
    if not tracker then
        return nil
    end
    -- init socket
    local sock, err = ngx.socket.tcp()
    if not sock then
        return nil, err
    end
    if self.timeout then
        sock:settimeout(self.timeout)
    end
    -- connect tracker
    local ok, err = sock:connect(tracker.host, tracker.port)
    if not ok then
        return nil, err
    end
    -- send request
    local bytes, err = sock:send(out)
    -- read request header
    local hdr = read_fdfs_header(sock)
    -- read body
    local res = read_tracket_result(sock, hdr)
    -- keepalive
    local keepalive = self.tracker_keepalive
    if keepalive then
        sock:setkeepalive(keepalive.timeout, keepalive.size)
    end
    return res
end

function do_download(self, fileid)
    local storage = self:query_download_storage(fileid)
    if not storage then
        return nil
    end
    local out = {}
    -- file_offset(8)  download_bytes(8)  group_name(16)  file_name(n)
    table.insert(out, int2buf(32 + string.len(storage.file_name)))
    table.insert(out, string.char(STORAGE_PROTO_CMD_DOWNLOAD_FILE))
    table.insert(out, "\00")
    -- file_offset  download_bytes  8 + 8
    table.insert(out, string.rep("\00", 16))
    -- group name
    table.insert(out, fix_string(storage.group_name, 16))
    -- file name
    table.insert(out, storage.file_name)
    -- init socket
    local sock, err = ngx.socket.tcp()
    if not sock then
        return nil, err
    end
    sock:settimeout(self.timeout)
    local ok, err = sock:connect(storage.host, storage.port)
    if not ok then
        return nil, err
    end
    local bytes, err = sock:send(out)
    if not bytes then
        ngx.log(ngx.ERR, "fdfs: send request error" .. err)
        sock:close()
        ngx.exit(500)
    end
    -- read request header
    local hdr = read_fdfs_header(sock)
    -- read request bodya
    local data, partial
    if hdr.len > 0 then
        data, err, partial = sock:receive(hdr.len)
        if not data then
            ngx.log(ngx.ERR, "read file body error:" .. err)
            sock:close()
            ngx.exit(500)
        end
    end
    local keepalive = self.storage_keepalive
    if keepalive then
        sock:setkeepalive(keepalive.timeout, keepalive.size)
    end
    return data
end

function do_append(self, fileid)
    local storage = self:query_update_storage(fileid)
    if not storage then
        return nil
    end
    local file_name = storage.file_name
    local file_name_len = string.len(file_name)
    -- get file size
    local file_size = tonumber(ngx.var.content_length)
    if not file_size or file_size <= 0 then
        return nil
    end
    local sock, err = ngx.socket.tcp()
    if not sock then
        return nil, err
    end
    if self.timeout then
        sock:settimeout(self.timeout)
    end
    local ok, err = sock:connect(storage.host, storage.port)
    if not ok then
        return nil, err
    end
    -- send request
    local out = {}
    table.insert(out, int2buf(file_size + file_name_len + 16))
    table.insert(out, string.char(STORAGE_PROTO_CMD_APPEND_FILE))
    -- status
    table.insert(out, "\00")
    table.insert(out, int2buf(file_name_len))
    table.insert(out, int2buf(file_size))
    table.insert(out, file_name)
    local bytes, err = sock:send(out)
    -- send file data
    local send_count = 0
    local req_sock, err = ngx.req.socket()
    if not req_sock then
        ngx.log(ngx.ERR, err)
        ngx.exit(500)
    end
    while true do
        local chunk, _, part = req_sock:receive(1024 * 32)
        if not part then
            local bytes, err = sock:send(chunk)
            if not bytes then
                ngx.log(ngx.ngx.ERR, "fdfs: send body error")
                sock:close()
                ngx.exit(500)
            end
            send_count = send_count + bytes
        else
            -- part have data, not read full end
            local bytes, err = sock:send(part)
            if not bytes then
                ngx.log(ngx.ngx.ERR, "fdfs: send body error")
                sock:close()
                ngx.exit(500)
            end
            send_count = send_count + bytes
            break
        end
    end
    if send_count ~= file_size then
        -- send file not full
        ngx.log(ngx.ngx.ERR, "fdfs: read file body not full")
        sock:close()
        ngx.exit(500)
    end
    -- read response
    local res_hdr = read_fdfs_header(sock)
    local res = read_storage_result(sock, res_hdr)
    local keepalive = self.storage_keepalive
    if keepalive then
        sock:setkeepalive(keepalive.timeout, keepalive.size)
    end
    return res_hdr
end

-- _M.query_upload_storage = query_upload_storage
-- _M.do_upload_storage    = do_upload_storage
-- _M.do_delete_storage    = do_delete_storage

local class_mt = {
    -- to prevent use of casual module global variables
    __newindex = function (table, key, val)
        error('attempt to write to undeclared variable "' .. key .. '"')
    end
}

setmetatable(_M, class_mt)

```

####crontab.sh 定时器

```bash
凌晨2点执行，查找目录下面7天内没有被访问的文件并删除，释放空间
0 2 * * * find /data/images -atime -7 | xargs rm -rf

```



