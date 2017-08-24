# Docker  jenkins 





```bash



#拉取jenkins容器
docker pull jenkins
#启动容器，执行docker ps命令，可以看到容器在后台运行，完全正常

docker run -d --name jenkins -p 8080:8080 -v /var/jenkins_home jenkins:latest

#挂载个本地目录，执行docker ps命令查看，容器已经结束了

docker run -d --name jenkins -p 8080:8080 -v /root/jenkins_home:/var/jenkins_home jenkins:latest

docker-compose -f /opt/oso.yml up -d jenkins2
 
#查看日志，提示权限问题
docker logs jenkins
/usr/local/bin/jenkins.sh: line 25: /var/jenkins_home/copy_reference_file.log: Permission denied

#修改宿主机的权限

sudo chown -R 1000 /opt/jenkins_home


docker run -p 8080:8180 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins
docker run -p 8180:8080 -p 51000:50000 -v jenkins_home:/var/jenkins_home b7a0940f1b57









adduser test
vim /etc/passwd

  		test:x:1000:1000::/test:/sbin/bash


chown -R test:test jenkins_home/

 
 
```