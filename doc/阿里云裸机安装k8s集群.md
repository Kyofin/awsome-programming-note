# 阿里云裸机安装k8s集群

## 安装docker （所有节点）
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun


## 关闭 SELinux（所有节点）
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

## 所有节点确保防火墙关闭（所有节点）
systemctl stop firewalld
systemctl disable firewalld

## 添加 k8s 安装源（所有节点）
```
cat <<EOF > kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
mv kubernetes.repo /etc/yum.repos.d/
```

## 添加 Docker 安装源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

## 安装kubectl、kubeadm、kubelet （所有节点）
yum list kubelet kubeadm kubectl  --showduplicates|sort -r
yum install -y kubelet-1.18.0-0  kubeadm-1.18.0-0  kubectl-1.18.0-0 

## 解决pod无法正常运行 （所有节点）
```
cat <<EOF > subnet.env
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
EOF
mv subnet.env /run/flannel/
```

## 启动 kubelet、docker，并设置开机启动（所有节点）
systemctl enable kubelet
systemctl start kubelet
systemctl enable docker
systemctl start docker

## 修改 docker 配置（所有节点）
```
# kubernetes 官方推荐 docker 等使用 systemd 作为 cgroupdriver，否则 kubelet 启动不了
cat <<EOF > daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://ud6340vz.mirror.aliyuncs.com"]
}
EOF
mv daemon.json /etc/docker/
```

## 重启生效
systemctl daemon-reload
systemctl restart docker


## 用 kubeadm 初始化集群（仅在主节点跑）
```
# 初始化集群控制台 Control plane
# 失败了可以用 kubeadm reset 重置
kubeadm init --image-repository=registry.aliyuncs.com/google_containers

# 记得把 kubeadm join xxx 保存起来
# 忘记了重新获取：kubeadm token create --print-join-command

# 复制授权文件，以便 kubectl 可以有权限访问集群
# 如果你其他节点需要访问集群，需要从主节点复制这个文件过去其他节点
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# 在其他机器上创建 ~/.kube/config 文件也能通过 kubectl 访问到集群
```

## 把工作节点加入集群（只在工作节点跑）
```
kubeadm join 172.16.32.10:6443 --token xxx --discovery-token-ca-cert-hash xxx

```

## 安装网络插件，否则 node 是 NotReady 状态（主节点跑）,可能要等几分钟
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

```



 