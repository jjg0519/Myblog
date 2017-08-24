# Git 常用命令和用法

## 生成ssh密钥(适合多个git服务器)

```bash
~/.ssh
##生成key,并重命名为github_id_rsa.pub 
$ ssh-keygen -t rsa -C "wangmax0330@gmail.com"
cat github_id_rsa.pub  
ssh -T git@github.com #连接Github测试是否能通
ssh -v git@github.com  #进行调试,看是哪里出了问题
```

假如发现默认加载的密钥是id_rsa,我自定义名字github_id_rsa根本没有使用。同时可以使用如下命令查看密钥列表：

```bash
ssh-add -l
ssh-agent bash
```

## 配置config,解决多个git服务器问题

```bash
Methew@Methew-PC MINGW64 /e/Workspaces/OwnWorkspace
$ cat ~/.ssh/config
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa
```

## 初始化一个本地仓库

```bash
git init #新建一个本地仓库
git add README.md #将README.md文件加入到仓库中 git commit -m
git commit -m "first commit" #将文件commit到本地仓库

```

## 将本地仓库push到git服务器

```bash
git remote add origin git@github.com:wangmax0330/pikai-study-mybatis.git # 添加远程仓库，origin只是一个远程仓库的别名，可以随意取
git push -u origin master #将本地仓库push远程仓库，并将origin设为默认远程仓库
```





## 常用命令

```bash
git解决冲突
```

## 常见问题

### error: The following untracked working tree files would be overwritten by checkout 

```bash
$ git clean -d -fx ""
```

