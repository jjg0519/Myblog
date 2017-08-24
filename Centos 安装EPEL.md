# Centos 6安装EPEL

```bash
[root@localhost ~]# rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
[root@localhost ~]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
#实在不行就执行下面的
[root@localhost ~]# yum -y install epel-release
```

