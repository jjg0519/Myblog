# 配置scp在Linux或Unix之间传输文件无需密码

##如何配置scp文件传输

实现scp在Linux或Unix之间传输文件，首先需要配置好scp，默认scp要使用密码的，通过以下配置可以不用输入密码，就完成Linux或Unix之间的文件传输

假设有2台Linux， A server， B server（ip假设为xxxx8），需要将文件（包括目录）从A传输到B，BFagent安装在A上面。 A上面的linuxidc用户，B上面也是linuxidc用户

A 机器上

A server上
第一步， 进入/home/linuxidc  cd /home/linuxidc  (因为我们使用的是linuxidc用户，如果使用了其他用户，就需要进去其他用户的目录， 比如 cd /home/weblogic)
第二部， 创建.ssh目录， mkdir .ssh
第三部， 进入.ssh目录，cd .ssh
第四部， 执行 ssh-keygen -b 1024 -t rsa

```bash
[root@shj-db02 ~]# cd ~
[root@shj-db02 ~]# mkdir .ssh
[root@shj-db02 ~]# cd .ssh
[root@shj-db02 ~]# ssh-keygen -b 1024 -t rsa
[root@shj-db02 ~]# ls ~/.ssh/
id_rsa  id_rsa.pub  known_hosts
```



B server上
第一步， 进入/home/linuxidc  cd /home/linuxidc  (因为我们使用的是linuxidc用户，如果使用了其他用户，就需要进去其他用户的目录， 比如 cd /home/weblogic)
第二部， 创建.ssh目录， mkdir .ssh
第三部， 进入.ssh目录，cd .ssh
第四部， 创建新文件authorized_keys，  touch authorized_keys

B server上
第五步， 将A server 生成的id_rsa.pub 内容复制到B server 的authorized_keys里面去

```bash
[root@shj-db02 ~]# cat id_rsa.pub
```



第六部， 测试文件传输，可以将/home/linuxidc 下面的某个目录传输给B。
​        例如将/home/linuxidc下面的dir001(该目录包括多个文件和目录) 传输到B server上/home/linuxidc/testdir目录下面
​        scp -r dir001 linuxidc@9.xxxx:/home/linuxidc/testdir

B server上

第五步， 进入/home/linuxidc/testdir, 检查传输的文件