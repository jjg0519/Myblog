#GitHub

```bash
~/.ssh
$ ssh-keygen -t rsa -C "wangmax0330@gmail.com"
cat github_id_rsa.pub 
ssh -T git@github.com 连接Github测试是否能通
ssh -v git@github.com  进行调试
```
发现默认加载的密钥是id_rsa,我自定义名字github_id_rsa根本没有使用。同时可以使用如下命令查看密钥列表：
```bash
ssh-add -l
ssh-agent bash
```



```bash
git init -- 新建一个本地仓库
git add README.md -- 将README.md文件加入到仓库中 git commit -m
git commit -m "first commit" -- 将文件commit到本地仓库
git remote add origin git@github.com:wangmax0330/pikai-study-mybatis.git -- 添加远程仓库，origin只是一个远程仓库的别名，可以随意取
git push -u origin master -- 将本地仓库push远程仓库，并将origin设为默认远程仓库
```
对于本地已经存在的git仓库
```bash
git remote add origin git@github.com:wangmax0330/pikai-study-mybatis.git 
git push -u origin master
```


git@github.com:wangmax0330/Myblog.git
git@github.com:wangmax0330/dacp-jdbc-study.git
git@github.com:wangmax0330/docker-jdk-tomcat-demo.git
git@github.com:wangmax0330/Break-GFW.git
git@github.com:wangmax0330/dacp-wechat-dev.git
git@github.com:wangmax0330/dacp-cache.git
git@github.com:wangmax0330/dacp-image.git
git@github.com:wangmax0330/dacp-sources.git
git@github.com:wangmax0330/dacp-parent.git
git@github.com:wangmax0330/dacp-core.git
git@github.com:wangmax0330/pikai-study-jdbcStudy.git
git@github.com:wangmax0330/pikai-study-mybatis.git
git@github.com:wangmax0330/pikai-study-spring.git
git@github.com:wangmax0330/pikia-test.git


