harbor集群安装

```
安装准备
三节点k8s
mater   192.168.1.24
worker1 192.168.1.25
worker2 192.168.1.26
安装步骤
1worker节点安装harbor
2master节点安装nginx
3检查push镜像
```
# 1离线安装harbor（worker节点）

```
下载https://github.com/goharbor/harbor/releases
harbor-offline-installer-v1.6.0.tgz
解压以后
cd harbor
改配置文件
vi harbor.cfg
hostname = 192.168.1.25

在线安装docker-compose
apt install docker-compose
检查docker-compose
docker-compose --version

启动harbor，需要等待一会
cd harbor/
./install.sh

浏览器检查一下
http://192.168.1.25/harbor/sign-in
好了harbor的一个节点就启动成功了
另一个节点worker2 192.168.1.26，也是这样安装
```
# 2安装nginx（master节点）
```
mkdir nginx
cd nginx/
docker pull nginx:1.13.12
写配置文件
vi nginx.conf
写入
```
```
user nginx;
worker_processes 1;

error_log /var/log/nginx/error.log warn;

pid /var/run/nginx.pid;

events {

	worker_connections 1024;
}

stream {

	upstream hub {
		server 192.168.1.25:80;
	}
	server {
		listen 80;
		proxy_pass hub;
		proxy_timeout 300s;
		proxy_connect_timeout 5s;
	}
}
```
```
vi restart.sh
写入


#!/bin/bash
docker stop harbornginx

docker rm harbornginx

docker run -idt --net=host --name harbornginx -v /root/nginx/nginx.conf:/etc/nginx/nginx.conf nginx:1.13.12


启动nginx
sh restart.sh
检查
http://192.168.1.24/harbor/sign-in

在本地绑一个host
C:\Windows\System32\drivers\etc\hosts
写入
192.168.1.24 test.mason.com

检查一下，使用域名访问
test.mason.com
好了harbor集群安装成功
```

# 3检查push镜像（master节点）

```
例子将nginx push到harbor仓库
写配置文件
vi /etc/docker/daemon.json
写入
{
	"insecure-registries": ["test.mason.com"]
}
```

```
配置完重启docker服务
systemctl daemon-reload
systemctl restart docker

重启完docker，nginx就停掉了，需要启动一下
sh restart.sh

域名配一个host(master节点)
vi /etc/hosts
写入
192.168.1.24 test.mason.com
```


浏览器登录 test.mason.com
创建用户
7wild
xxxxxxxxxxxxx

创建项目kubernetes


```
harbor登录（master节点）
docker login test.mason.com
7wild
xxxxxxxxxxxxx
```



```
改名字
docker tag nginx:1.13.12 test.mason.com/kubernetes/nginx:1.13.12
push镜像
docker push test.mason.com/kubernetes/nginx:1.13.12

子节点pull检查（worker节点）
vi /etc/hosts
192.168.1.24 test.mason.com

配置文件
vi /etc/docker/daemon.json
{
	"insecure-registries": ["test.mason.com"]
}

service docker restart

docker pull test.mason.com/kubernetes/nginx:1.13.12
```
```
docker tag hello-world:latest test.mason.com/kubernetes/helloworld
docker push test.mason.com/kubernetes/helloworld
docker pull test.mason.com/kubernetes/helloworld
```






















