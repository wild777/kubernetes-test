# 十分钟安装kubernetes v1.14.1
kubeadm搭建，全程阿里云镜像
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
容器运行时docker 18.09.5 
kubernetes版本1.14.1 
网络方案flannel 
```
# 安装环境
三台ubuntu16 
系统类型	   IP地址	       节点角色 	 CPU	Memory	Hostname 

ubuntu16   192.168.1.24	       master    1	1G	m1 

ubuntu16   192.168.1.25	      worker	 1	1G	s1 

ubuntu16   192.168.1.26	      worker	 1	1G	s2 



# 1安装准备（所有节点）
```
ubuntu安装主机名不同，k8s靠主机名识别节点，名字一样有可能识别不到节点 
查看主机名 
$ hostname
修改主机名 
$ hostnamectl set-hostname xxx
配置host，使所有节点之间可以通过hostname互相访问 
$ vi /etc/hosts
192.168.1.24 m1
192.168.1.25 s1
192.168.1.26 s2

cat /etc/hosts
```
1.1关闭、禁用防火墙
```
关闭防火墙
$ ufw disable
查看状态 
$ ufw status
```
# 2安装docker（所有节点）
```
首先安装依赖:
$ apt-get update
$ sudo apt-get install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common
根据你的发行版，下面的内容有所不同。你使用的发行版：   
信任 Docker 的 GPG 公钥: 

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - 
对于 amd64 架构的计算机，添加软件仓库: 

$ sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

最后安装 
 
$ sudo apt-get update
$ sudo apt-get install -y docker-ce
 
查看运行状态 
$ systemctl status docker
```
2.1 docker加速器 
```
$ curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
$ systemctl restart docker.service
```
# 3安装kubelet kubeadm kubectl
```
$ curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
$ cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
$ apt-get update
$ apt-get install -y kubelet kubeadm kubectl

禁止更新这三个 
$ apt-mark hold kubelet kubeadm kubectl

查看kubeadm版本 
$ kubeadm version
$ kubectl version
查看kubelet状态
systemctl status kubelet
```
3.1初始化master节点（master节点） 
```
关闭swap 
$ swapoff -a 
$ kubeadm init --image-repository  registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=NumCPU

需要复制这行代码，在后面加入node
kubeadm join 例子 --token 3pti18.ln4bnmd6pkrkmenr --discovery-token-ca-cert-hash sha256:b76dc8cf29b6de24ef405126cb59a7b5f1ebc3c474df1aa8736429b27a529acf

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
关闭swap 
$ swapoff -a
将之前记录下来的kubeadm join命令执行 

$ kubeadm join 例子 --token sd6c5c.akighyvtqh3ihg2a --discovery-token-ca-cert-hash sha256:55419ffaaedbef0ce31177ca1211944234be21b364c6e0d81a47defefe83e462
 
$ sysctl net.bridge.bridge-nf-call-iptables=1
```

# 5检查node状态（master节点）
```
$ kubectl get node

$ kubectl get po --all-namespaces

$ kubectl describe pod kube-flannel-ds-hnqlz -n kube-system
```














--




















