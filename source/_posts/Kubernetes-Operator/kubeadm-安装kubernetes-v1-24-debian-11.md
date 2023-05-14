---
title: kubeadm 安装kubernetes(v1.24) debian 11
date: 2023-04-15 17:40:44
tags:
  - Kubernetes
categories: Kubernetes-Operator
---
{% notel primary 说明 %}
以下命令都是基于 root用户执行，所以没有加 提权命令，如果你不是root 请加上 `sudu`或者使用`su root`，由于服务器环境配置需要所有节点进行执行，推荐使用Xshell的 <font color="	#FF7F24">"发送键输入到所有会话功能"</font>，这样可以在一台机器上执行命令，多台服务器同时执行（快捷键`ctrl + shift + A`）
{% endnotel %}

### 配置清单
|       主机名    |     IPADDR    |  SYSTEM   |
|     :------:    |    :-----:    | :------:  |
|  k8s-master-01  | 172.16.10.101 | Debian 11 |
|  k8s-worker-01  | 172.16.10.221 | Debian 11 |
|  k8s-worker-02  | 172.16.10.222 | Debian 11 |

### 服务器环境配置（所有节点执行）
```sh
# 设置 ssh可以远程登录 root
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config

# 重启 ssh
systemctl restart ssh

# 关闭 Linux交换分区，提升 kubernetes性能
swapoff -a
sed -ri '/\sswap\s/s/^#?/#/' /etc/fstab

# 在 k8s-master-01执行
hostnamectl --static set-hostname k8s-master-01 && bash
# 在 k8s-worker-01执行
hostnamectl --static set-hostname k8s-worker-01 && bash
# 在 k8s-worker-02执行
hostnamectl --static set-hostname k8s-worker-02 && bash


# hosts配置 
cat >> /etc/hosts << EOF 
172.16.10.101 k8s-master-01
172.16.10.221 k8s-worker-01
172.16.10.222 k8s-worker-02
EOF

# 转发 IPv4 并让 iptables 看到桥接流量 
cat <<EOF |  tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

# 设置所需的 sysctl 参数，参数在重新启动后保持不变
cat <<EOF |  tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# 应用 sysctl 参数而不重新启动
sysctl --system

# 通过运行以下指令确认 br_netfilter 和 overlay 模块被加载：
lsmod | grep br_netfilter
lsmod | grep overlay

# 通过运行以下指令确认 net.bridge.bridge-nf-call-iptables、net.bridge.bridge-nf-call-ip6tables和 net.ipv4.ip_forward系统变量在你的sysctl配置中被设置为 1：
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

### 安装docker & contaianerd
```sh
# 卸载docker和containerd
apt-get remove docker docker-engine docker.io containerd runc

# 重新安装
apt-get update
apt-get install \
    ca-certificates \
    curl \
    gnupg

install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg |  gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
   tee /etc/apt/sources.list.d/docker.list > /dev/null

 apt-get update

# 最新稳定版本
 apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```
### 配置 CRI（二选一）
{% tabs first %}
<!-- tab Docker CRI -->
```sh
# "bip": "172.18.97.1/24", 查看内网IP是否和docker默认网段(172.16.0.1/16)有冲突，如有冲突需要单独配置 bip替换有冲突的网段，请注意bip是写主机地址而不是网络地址
# 我用的是中国科技大镜像加速站，你也可以替换为其它镜像源
cat <<EOF |  tee /etc/docker/daemon.json
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"],  
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

 systemctl enable docker
 systemctl daemon-reload
 systemctl restart docker
 ```
<!-- endtab -->
<!-- tab Containerd CRI -->
```sh
# containerd安装如上docker所示
# 配置containerd，修改sandbox_image 镜像源
# 导出默认配置，config.toml这个文件默认是不存在的
containerd config default > /etc/containerd/config.toml

# 修改前检查
grep sandbox_image  /etc/containerd/config.toml

# 修改sandbox_image 镜像源
sed -i "s#registry.k8s.io/pause#registry.aliyuncs.com/google_containers/pause#g" /etc/containerd/config.toml

# 修改后检查
grep sandbox_image  /etc/containerd/config.toml

# 配置containerd cgroup 驱动程序systemd
# 把SystemdCgroup = false修改为：SystemdCgroup = true，
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

# Containerd配置镜像加速
vim /etc/containerd/config.toml
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://docker.mirrors.ustc.edu.cn" ,"https://registry-1.docker.io"]

# 重启
systemctl daemon-reload
systemctl enable --now containerd
systemctl restart containerd
``` 
<!-- endtab -->
<!-- tab CRI-Dockerd -->
```sh
# 安装cri-dockerd
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.1/cri-dockerd_0.3.1.3-0.debian-buster_amd64.deb

# unix:///var/run/cri-dockerd.sock
# systemd的服务地址 /lib/systemd/system/cri-dockerd.service
dpkg -i cri-dockerd_0.3.1.3-0.debian-buster_amd64.deb

# 修改镜像地址为国内，否则kubelet拉取不了镜像导致启动失败
# 重载沙箱（pause）镜像 可执行命令 kubeadm config images list 查看国内阿里云最新沙箱镜像
vim /lib/systemd/system/cri-docker.service
ExecStart=/usr/bin/cri-dockerd --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.9 --container-runtime-endpoint fd://

systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
```
<!-- endtab -->
{% endtabs %}

### 更换 kubernetes源
```sh
apt install -y apt-transport-https ca-certificates curl

curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main" |  tee /etc/apt/sources.list.d/kubernetes.list

apt update
```

### 安装指定版本kubeadm
```sh
# 安装kubeadm & kubelet & kubectl
apt install -y kubeadm=1.24.12-00 kubelet=1.24.12-00 kubectl=1.24.12-00

# 验证版本是否正确
kubeadm version
kubectl version --client
kubectl version --short

# 避免意外升级导致版本错误
apt-mark hold kubeadm kubelet kubectl
```


### 控制面节点初始化(master)
kubeadm 的用法非常简单，只需要一个命令 kubeadm init 就可以把组件在 Master 节点上运行起来，不过它还有很多参数用来调整集群的配置，你可以用 -h 查看。这里说下几个重点参数：

* `--pod-network-cidr`，设置集群里 Pod 的 IP 地址段。
* `--service-cidr`，设置集群里 Service 的 IP 地址段。默认：10.96.0.0/12
* `--apiserver-advertise-address`，设置 apiserver 的 IP 地址，对于多网卡服务器来说很重要（比如 VirtualBox 虚拟机就用* 了两块网卡），可以指定 apiserver 在哪个网卡上对外提供服务。
* `--kubernetes-version`，指定 Kubernetes 的版本号。
下面的这个安装命令里，我指定了 Pod的地址段是 10.96.0.0/12
apiserver 的服务地址是 172.16.10.101，Kubernetes的版本号是 1.24.12
---
```sh
kubeadm init \
 --apiserver-advertise-address=172.16.10.101 \
 --image-repository registry.aliyuncs.com/google_containers \
 --kubernetes-version v1.24.12 \
 --service-cidr=10.96.0.0/12 \
 --pod-network-cidr=10.12.9.0/16 \
 --v=5
```

{% notel info 说明 %}
`--cri-socket=unix:///var/run/cri-dockerd.sock` 如果使用 containerd 作为CRI，就不需要加上该参数。
如果使用 docker 作为CRI就需要加上 `--cri-socket` 参数，同时还必须提前安装 `cri-dockerd` 后，再初始化一个 Kubernetes 控制平面节点(master)。

`--image-repository` 加上国内阿里云镜像仓库会快很多，不然默认使用谷歌的，会很慢，一直在init阶段，初始化不成功
{% endnotel %}

```sh
# 配置文件方式 当前用户
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# 环境变量方式 临时生效（退出当前窗口重连环境变量失效）
export KUBECONFIG=/etc/kubernetes/admin.conf

# 环境变量方式 永久生效（推荐）
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source  ~/.bash_profile

# 获取目前已有的token，如果没用，说明所有的token都失效了
kubeadm token list

# 默认情况下，令牌会在 24 小时后过期。如果要在当前令牌过期后将节点加入集群， 则可以通过在控制平面节点上运行以下命令来创建新令牌：
kubeadm token create --print-join-command

# 可生成永久token，不建议这样做
kubeadm token create --ttl 0 --print-join-command
```
{% notel warning 提示 %}
如果使用 docker 作为 CRI时，其他节点加入集群需要添加 --cri-socket 参数, Like this
`kubeadm join ......  --cri-socket=unix:///var/run/cri-dockerd.sock`
{% endnotel %}

### 加入Worker节点（worker执行）
`kubeadm join 172.16.10.101:6443 --token xi7hnt.0e2i9xqauw7pccxf 	--discovery-token-ca-cert-hash sha256:63235175aa2ec145685a8b905c69d47327aaaf1e2dd2dddc9b615259fe9ad1ba`

### 安装网络插件
```sh
# 下载kube-flannel.yml
wget https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# 修改podCIDR
vim kube-flannel.yml 

  net-conf.json: |
    {
      "Network": "10.12.9.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }

# 通过yaml文件部署kube-flannel
kubectl apply -f kube-flannel.yml
```
### 集群验证
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230415232546.png)