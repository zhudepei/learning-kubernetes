## 安装环境
* Ubuntu 18.04
* Kubernetes 1.17

## 一些初始配置
```shell
apt update
apt-get update && apt-get install -y \
  apt-transport-https ca-certificates curl software-properties-common gnupg2
```

## 添加 Aliyun 镜像源
```shell
# 写入两个 key
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | apt-key add -
curl -fsSL https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -

# 添加镜像源
cat <<EOF >/etc/apt/sources.list.d/docker-k8s.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable
EOF
```

## 安装 Docker
```shell
# 查看 docker 可用版本
apt-cache madison docker-ce

# 安装指定版本
#apt install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io

# 安装最新版本
## Install Docker CE.
apt-get update && apt-get install -y \
  containerd.io=1.2.13-1 \
  docker-ce=5:19.03.8~3-0~ubuntu-$(lsb_release -cs) \
  docker-ce-cli=5:19.03.8~3-0~ubuntu-$(lsb_release -cs)

# Setup daemon.
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

# Restart docker.
systemctl daemon-reload
systemctl restart docker
```

# 关闭 swap
```shell
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab
```

# 设置主机名 hostname
```shell
hostnamectl set-hostname your-hostname
```

# 安装 kubeadm、kubelet、kubectl
```shell
sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

# 用apt-mark 指令设定套件不更新
sudo apt-mark hold kubelet kubeadm kubectl
```

# 用 kubeadm 初始化安装 kubernetes
```shell
kubeadm init --control-plane-endpoint "load-balancer-dns:6443" --apiserver-cert-extra-sans="your ip address" 
--pod-network-cidr="192.168.0.0/18" --service-cidr="192.168.64.0/18" --upload-certs --image-repository registry.aliyuncs.com/google_containers
```
* --control-plane-endpoint 标志设置负载均衡器的 地址或DNS 和端口
* --apiserver-cert-extra-sans 指定kubectl 从哪个 IP 外网访问集群
* --image-repository 指定使用阿里云镜像源
* --upload--certs 标志用来将在所有控制平面实例之间的共享证书上传到集群
