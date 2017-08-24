# Ubunut 开启root账号

```bash
# 打开终端开启root账号
methew@ubuntu-server:~$ sudo passwd -u root
[sudo] password for methew: 
passwd: unlocking the password would result in a passwordless account.
You should set a password with usermod -p to unlock the password of this account.
methew@ubuntu-server:~$ sudo passwd root
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
#测试root账号,输入root 密码
methew@ubuntu-server:~$ su -
Password: 
root@ubuntu-server:~# exit
logout
methew@ubuntu-server:~$ 
#编辑ssh的配置文件,命令
methew@ubuntu-server:~$ sudo vim /etc/ssh/sshd_config

#在Authentication部分，注释掉
PermitRootLogin without-password
#在Authentication部分，添加
PermitRootLogin yes
#重新启动ssh服务，命令
sudo service ssh restart
```

