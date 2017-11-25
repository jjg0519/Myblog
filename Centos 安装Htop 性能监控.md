# Centos 安装Htop 性能监控

```bash
rpm -ivh http://download.fedoraproject.org/pub/epel/5/i386/epel-release-5-4.noarch.rpm 

rpm -ivh  https://mirrors.ustc.edu.cn/epel/6/i386/epel-release-6-8.noarch.rpm
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL
yum install htop


##卸载
rpm -qi epel-release
rpm -e epel-release
```

