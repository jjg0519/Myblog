# 安装 Compose

在安装 Compose之前，你需要先安装好 Docker 。然后你需要使用 `curl` 指令来安装 Compose

### 安装 Compose

运行下边的命令来安装 Compose**非常慢**

```bash
curl -L https://github.com/docker/compose/releases/download/1.3.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

> 注意：如果你在安装的时候出现了 “Permission denied” 的错误信息，这说明你的 `/usr/local/bin` 目录是不可写的，你需要使用超级用户来安装。运行 `sudo -i` , 然后运行上边的两个命令，然后 `exit` 退出。

可选，你也可以在 shell 中使用命令行安装。

Compose 适用于 OS X 和 64位的Linux 。 如果你使用其他平台，你可以安装一个 Compose 的 Python 包来完成安装。

```bash
# Ubunut
$ sudo pip install -U docker-compose

# Centos
$ yum install wget
$ wget https://bootstrap.pypa.io/get-pip.py
$ python get-pip.py
$ pip install -U docker-compose
$ chmod +x /usr/local/bin/docker-compose
$ docker-compose -version



pip uninstall PackageName  # 卸载
```





到这里安装就结束了；Compose已经安装完成。你可以使用 `docker-compose --version` 来进行测试 。

### 升级

如果你使用的是 Compose 1.2或者早期版本，当你升级完成后，你需要删除或者迁移你现有的容器。这是因为，1.3版本， Composer 使用 Docker 标签来对容器进行检测，所以它们需要重新创建索引标记。

如果 Composer 检测到创建的容器没有标签，它将拒绝运行，这样你就不会有两组容器。如果你想要保留已经存在的容器（举例：这里有容器的数据卷上保留这非常重要的数据），你可以使用下边的命令来迁移：

```bash
docker-compose migrate-to-labels
```

或者，如果这些容器是不必要的，你可以删除它们 - Composer 会重新创建一个新的。

```bash
docker rm -f myapp_web_1 myapp_db_1 ...
```

### docker-compose常用命令

在第二节中的`docker-compose up`，这两个容器都是在前台运行的。我们可以指定-d命令以daemon的方式启动容器。除此之外，docker-compose还支持下面参数：
`--verbose`：输出详细信息
`-f` 制定一个非docker-compose.yml命名的yaml文件
`-p` 设置一个项目名称（默认是directory名）
docker-compose的动作包括：
`build`：构建服务
`kill -s SIGINT`：给服务发送特定的信号。
`logs`：输出日志
`port`：输出绑定的端口
`ps`：输出运行的容器
`pull`：pull服务的image
`rm`：删除停止的容器
`run`: 运行某个服务，例如docker-compose run web python manage.py shell
`start`：运行某个服务中存在的容器。
`stop`:停止某个服务中存在的容器。
`up`：create + run + attach容器到服务。
`scale`：设置服务运行的容器数量。例如：docker-compose scale web=2 worker=3



删除单个容器

```bash
$docker rm Name/ID  
```

```bash

docker-compose -f /opt/oso.yml up -d mysql_develop
docker-compose -f /opt/oso.yml up -d mysql
docker-compose -f /opt/oso.yml up -d zookeepers
docker-compose -f /opt/oso.yml up -d redis_cluster
docker-compose -f /opt/oso.yml up -d redis-cluster
docker-compose -f /opt/oso.yml up -d jenkins2
```

