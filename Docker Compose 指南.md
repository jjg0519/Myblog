

# Docker Compose 





## 安装方式

```bash
# 参考http://docs.docker.com/compose/install/#install-compose
curl -L https://github.com/docker/compose/releases/download/1.2.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
### 上面这个方法真的慢出翔，可以通过Python pip安装。
apt-get install python-pip python-dev
pip install -U docker-compose
```



Centos 
yum install wget
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
pip install -U docker-compose



删除单个容器

```bash
$docker rm Name/ID  
```


docker-compose -f /opt/oso.yml up -d mysql_develop
docker-compose -f /opt/oso.yml up -d mysql
docker-compose -f /opt/oso.yml up -d zookeepers
docker-compose -f /opt/oso.yml up -d redis_cluster
docker-compose -f /opt/oso.yml up -d redis-cluster
docker-compose -f /opt/oso.yml up -d jenkins2