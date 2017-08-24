#Ubuntu  部署svn 服务器环境


-------------------

[TOC]

###安装
``` bash
sudo apt-get install subversion
```

检查时候安装成功
>svn help --svn帮助
>svn --version --svn版本
>svnserve --version --svn server版本

###创建SVN库
``` bash
mkdir svn
svnadmin create svn/svnname  --svnname为版本库名称
```

###SVN 配置

创建版本库后，在这个目录下会生成3个配置文件：
``` bash
[root@singledb conf]# pwd
/datacenter/svn/officeSapce//conf
[root@singledb conf]# ls
authz  passwd  svnserve.conf
```


（1）svnserve.conf：  svn服务配置文件下。 
（2）passwd： 用户名口令文件。 
（3）authz： 权限配置文件。  
  
svnserve.conf 文件， 该文件配置项分为以下5项： 
       >anon-access： 控制非鉴权用户访问版本库的权限。 
       >auth-access：  控制鉴权用户访问版本库的权限。 
       >password-db： 指定用户名口令文件名。 
       >authz-db：指定权限配置文件名，通过该文件可以实现以路径为基础的访问控制。 
       >realm：指定版本库的认证域，即在登录时提示的认证域名称。若两个版本库的认证域相同，建议使用相同的用户名口令数据文件 

* 配置版本库
  # sudo vi svnserve.conf  #将以下参数去掉注释 
``` bash
  [general] 
  anon-access = none    #匿名访问权限，默认read，none为不允许访问 
  auth-access = write  #认证用户权限  
  password-db = passwd  #用户信息存放文件，默认在版本库/conf下面，也可以绝对路径指定文件位置 
  authz-db = authz
```

* 配置用户
 # vi passwd
``` 
[users]
# harry = harryssecret
# sally = sallyssecret
methew=158180
guest=guest
```
* 配置权限配置文件 
 # vi authz
``` 
[groups]
admin=methew
dev=test
[/]
@admin=rw
@dev=r
* =
```

###启动SVN服务
#####启动服务
```
svnserve -d -r /datacenter/svn/officeSapce/
```
查看服务是否运行ps -ef|grep svn, 看到 svnserve -d -r /datacenter/svn/officeSapce 代表已经在运行
```
[root@cmini conf]# ps -ef|grep svn
root 2774 1 0 16:46 ? 00:00:00 svnserve -d -r /datacenter/svn/officeSapce/
root 14174 9352 0 22:38 pts/1 00:00:00 grep svn
```
#####关闭服务
```
kill 2774
```

#####重启服务
```
svnserve -d -r /datacenter/svn/officeSapce/
```
描述说明：
-d 表示svnserver以“守护”进程模式运行
-r 指定文件系统的根位置（版本库的根目录），这样客户端不用输入全路径，就可以访问版本库

启动制定了根目录
```
svnserve -d -r /datacenter/svn/officeSapce/
```
checkout时可以不指定
```
svn://192.168.12.118/
```
启动制定了根目录
```
svnserve -d -r /datacenter/svn/
```
checkout时需要指定
```
svn://192.168.12.118/officeSapce/
```
![Alt text](https://raw.githubusercontent.com/wangmax0330/dacp-image/master/20160521140558.png)
