# CentOS搭建Gitlab安装步骤、汉化、日常管理以及异常故障排查

## 服务器快速搭建gitlab方法

可以参考gitlab中文社区 的教程
centos7安装gitlab：https://www.gitlab.cc/downloads/#centos7
centos6安装gitlab：https://www.gitlab.cc/downloads/#centos6



## Centos 6

### 安装配置依赖项

如想使用Postfix来发送邮件,在安装期间请选择'Internet Site'. 您也可以用sendmai或者 [配置SMTP服务](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/smtp.md) 并 [使用SMTP发送邮件.](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/smtp.md#smtp-on-localhost)

在 Centos 6 和 7 系统上, 下面的命令将在系统防火墙里面开放HTTP和SSH端口.

```bash
yum -y install curl openssh-server openssh-clients postfix cronie
sudo service postfix start
sudo chkconfig postfix on
sudo lokkit -s http -s ssh
```

### 添加GitLab仓库,并安装到服务器上

```bash
curl -sS http://packages.gitlab.cc/install/gitlab-ce/script.rpm.sh | sudo bash
sudo yum install gitlab-ce
```

如果你不习惯使用命令管道的安装方式, 你可以在这里下载 [安装脚本](http://packages.gitlab.cc/install/gitlab-ce/) 或者 [手动下载您使用的系统相应的安装包(RPM/Deb)](https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/) 然后安装

```bash
curl -LJO https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6/gitlab-ce-XXX.rpm
rpm -i gitlab-ce-XXX.rpm
```

### 启动GitLab

```bash
sudo gitlab-ctl reconfigure
```

### 使用浏览器访问GitLab

首次访问GitLab,系统会让你重新设置管理员的密码,设置成功后会返回登录界面.

默认的管理员账号是**root**,如果你想更改默认管理员账号,请输入上面设置的新密码登录系统后修改帐号名.

## Centos 7

### 安装配置依赖项

如想使用Postfix来发送邮件,在安装期间请选择'Internet Site'. 您也可以用sendmai或者 [配置SMTP服务](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/smtp.md) 并 [使用SMTP发送邮件.](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/smtp.md#smtp-on-localhost)

在 Centos 6 和 7 系统上, 下面的命令将在系统防火墙里面开放HTTP和SSH端口.

```bash
sudo yum install curl policycoreutils openssh-server openssh-clients
sudo systemctl enable sshd
sudo systemctl start sshd
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
sudo firewall-cmd --permanent --add-service=http
sudo systemctl reload firewalld
```

### 添加GitLab仓库,并安装到服务器上

```bash
curl -sS http://packages.gitlab.cc/install/gitlab-ce/script.rpm.sh | sudo bash
sudo yum install gitlab-ce
```

如果你不习惯使用命令管道的安装方式, 你可以在这里下载 [安装脚本](http://packages.gitlab.cc/install/gitlab-ce/) 或者 [手动下载您使用的系统相应的安装包(RPM/Deb)](https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/) 然后安装

```bash
curl -LJO https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-XXX.rpm
rpm -i gitlab-ce-XXX.rpm
```

### 启动GitLab

```bash
sudo gitlab-ctl reconfigure
##关闭
gitlab-ctl stop
```

## 卸载

```bash
yum list installed|grep git
git.x86_64            1.7.1-4.el6_7.1   @base                            
gitlab-ce.x86_64      9.0.2-ce.0.el6    @gitlab-ce               
xz.x86_64             4.999.9-0.5.beta.20091007git.el6
xz-libs.x86_64        4.999.9-0.5.beta.20091007git.el6
[root@localhost ~]# yum remove gitlab-ce.x86_64 
```



## 注意事项以及异常故障排查：

* 按照该方式，我安装了一个确实没问题，只不过是英文版。没有经过汉化。
* 默认安装登录需要重置root密码。可以自己单独设置一个复杂密码后登录。
* gitlab本身采用80端口，如安装前服务器有启用80，安装完访问会报错。需更改gitlab的默认端口。

修改vim /etc/gitlab/gitlab.rb：
external_url ‘http://localhost:90’

* unicorn本身采用8080端口，如安装前服务器有启用8080，安装完访问会报错。需更改unicorn的默认端口。

修改 /etc/gitlab/gitlab.rb：
unicorn[‘listen’] = ‘127.0.0.1’
unicorn[‘port’] = 3000

* 每次重新配置，都需要执行sudo gitlab-ctl reconfigure  使之生效。
* 日志位置：/var/log/gitlab 可以进去查看访问日志以及报错日志等，供访问查看以及异常排查。

gitlab-ctl tail #查看所有日志
gitlab-ctl tail nginx/gitlab_access.log #查看nginx访问日志