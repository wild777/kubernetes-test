阿里云搭建kubernetes三节点
```
安装步骤 
1安装准备 
2安装docker 
3初始化master 
4加入node 
5检查
```
# 集群方案 
```
Ubuntu 16.4 
容器运行时docker 18.09.6
kubernetes版本1.14.1 
网络方案flannel 
```
# 安装环境
三台ubuntu16 
系统类型	   IP地址	       节点角色 	 CPU	Memory	Hostname 

ubuntu16   172.26.45.142	       master    1	1G	m1 

ubuntu16   172.26.45.143	      worker	 1	1G	s1 

ubuntu16   172.26.45.144      worker	 1	1G	s2 



# 1安装准备（所有节点）
```
ubuntu安装主机名不同，k8s靠主机名识别节点，名字一样有可能识别不到节点 
查看主机名 
$ hostname
修改主机名 
$ hostnamectl set-hostname xxx
配置host，使所有节点之间可以通过hostname互相访问 
$ vi /etc/hosts
172.26.45.142 m1
172.26.45.143 s1
172.26.45.144 s2

cat /etc/hosts
```
1.1关闭、禁用防火墙
```
关闭防火墙
$ ufw disable
查看状态 
$ ufw status
关闭swap
$ swapoff -a
```
# 2安装docker（所有节点）

```
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
apt install docker.io
```
# 3安装kubelet kubeadm kubectl
```
$ curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
$ cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
$ apt-get update
$ apt-get install -y kubelet kubeadm kubectl

查看kubeadm版本 
$ kubeadm version
$ kubectl version
查看kubelet状态
systemctl status kubelet
```
3.1初始化master节点（master节点） 
```
初始化添加镜像仓库地址，指定pod网络 容器使用ip地址，一核cpu
$ kubeadm init --image-repository  registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=NumCPU
必须记录kubeadm join
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
3.2k8s安装网络插件（master节点） 
```
$ sysctl net.bridge.bridge-nf-call-iptables=1
$ curl -O https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml

修改kube-flannel.yml文件 
$ vi kube-flannel.yml
在spec  tolerations下面 
增加一个key 

- key: node.kubernetes.io/not-ready
 operator: Exists
 effect: NoSchedule
保存退出  
$ kubectl apply -f kube-flannel.yml
```

# 4添加node节点（worker节点）
```
一定要等待主节点的flannel pod跑起来，再去加入node

将之前记录下来的kubeadm join命令执行 
kubeadm join 172.26.45.143:6443 --token 5woyzh.ypycczn8aly1hyz6 --discovery-token-ca-cert-hash sha256:5bedf02f14498af83f58f001f40724129abeff80ca5625c1c2461aa854accf74

```


# 5检查node状态（master节点）
```
$ kubectl get node

$ kubectl get po --all-namespaces

$ kubectl describe pod kube-flannel-ds-hnqlz -n kube-system

$ 
```