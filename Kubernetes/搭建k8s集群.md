# 目标

搭建一个K8s集群，一个master，两个node



## 安装虚拟机

| 主机IP          | 操作系统     | CPU  | 内存 | 磁盘 |
| --------------- | ------------ | ---- | ---- | ---- |
| 192.168.124.49  | Ubuntu 16.04 | 4    | 8G   | 50G  |
| 192.168.124.149 | Ubuntu 16.04 | 4    | 8G   | 50G  |
| 192.168.124.231 | Ubuntu 16.04 | 4    | 8G   | 50G  |



## 准备

```shell
# 关闭防火墙
ufw disable
# 关闭Swap
swapoff -a
# 如果可以的话设置科学上网
```

### 安装过程

#### 1.所有主机安装docker

```shell
apt-get install -y docker.io
```

#### 2.所有主机安装kubeadm，kubelet，kubectl

首先添加Kubernetes源

- 如果可以科学上网，则执行以下语句获取Kubernetes源

```shell
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
```

然后安装kubelet，kubectl，kubeadm

```shell
sudo apt-get install -y kubelet kubeadm kubectl
```



- 如果不能科学上网，那就添加阿里云的kubernetes源

```shell
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main
EOF

apt-get update # 忽略gpg的报错信息
```

然后使用下面命令安装kubelet，kubectl，kubeadm

```shell
apt-get install -y kubelet kubeadm kubectl --allow-unauthenticated
```



#### 3.kubeadm init

```shell
kubeadm init --pod-network-cidr=10.244.0.0/16
```

如果可以科学上网，那应该就不会出错，否则报错如下：

```shell
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-apiserver:v1.16.3: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-controller-manager:v1.16.3: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-scheduler:v1.16.3: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-proxy:v1.16.3: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/pause:3.1: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/etcd:3.3.15-0: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/coredns:1.6.2: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
```

从报错信息中可以看到哪些镜像下载不成功，可以到dockerHub上寻找相对应的组件及版本，进行下载，之后再通过打tag，修改为需要的镜像，比如以下代码：

```shell
# 下载可以拉取的相对应的组件和版本
docker pull aiotceo/kube-apiserver:v1.16.3
docker pull aiotceo/kube-controller-manager:v1.16.3
docker pull aiotceo/kube-scheduler:v1.16.3
docker pull aiotceo/kube-proxy:v1.16.3
docker pull aiotceo/pause:3.1
docker pull aiotceo/etcd:3.3.15
docker pull aiotceo/coredns:1.6.2

# 通过打tag的方式修改为所需要的镜像
docker tag aiotceo/coredns:1.6.2 k8s.gcr.io/coredns:1.6.2
docker tag aiotceo/etcd:3.3.15 k8s.gcr.io/etcd:3.3.15-0
docker tag aiotceo/pause:3.1 k8s.gcr.io/pause:3.1
docker tag aiotceo/kube-proxy:v1.16.3 k8s.gcr.io/kube-proxy:v1.16.3
docker tag aiotceo/kube-scheduler:v1.16.3 k8s.gcr.io/kube-scheduler:v1.16.3
docker tag aiotceo/kube-controller-manager:v1.16.3 k8s.gcr.io/kube-controller-manager:v1.16.3
docker tag aiotceo/kube-apiserver:v1.16.3 k8s.gcr.io/kube-apiserver:v1.16.3

# 然后重新init，即可
kubeadm init --pod-network-cidr=10.244.0.0/16
```



以下是运行init成功后的结果

```shell
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.124.49:6443 --token uv17vd.q3ber8i5knxg4h0x \
    --discovery-token-ca-cert-hash sha256:c55bd70d346d809e1079565cc1fc1a05f001671cc9f2d02c55bbbc4a00bcc2a3
```



#### 4.使用集群

从结果可以看到，想要使用集群，需要执行以下命令，执行结束后才可以使用kubectl

```shell
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

现在可以通过以下命令检测node和pod运行状态：

```shell
kubectl get node # 检查node是否ready
# 检查所有pod是否正常
kubectl get pod --all-namespaces -o wide
#如果pod处于非running状态，则查看该pod：
kubectl describe pod xxxxx -n kube-system
# 如果是因为镜像无法下载导致镜像启动失败，则手动在node上pull image，然后在master上删掉对应的pod，k8s会重新调度pod
# 删除pod命令
kubectl delete pod xxxx -n kube-system
```



现在查看node信息的话，应该会显示notReady，是因为还没有安装网络插件

#### 5.安装网络插件

以安装Weave插件为例，执行以下命令：

```shell
sysctl net.bridge.bridge-nf-call-iptables=1

kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

执行结束后，master应该会显示ready状态



如果整个安装失败的话，可以重置，重新安装，即重新kubeadm init

```shell
kubeadm reset 
```



#### 6.可以添加node节点

##### 6.1 kubeadm join

根据master执行kubeadm init的结果，想要加入集群，可以执行以下命令：

```shell
kubeadm join 192.168.124.49:6443 --token uv17vd.q3ber8i5knxg4h0x \
    --discovery-token-ca-cert-hash sha256:c55bd70d346d809e1079565cc1fc1a05f001671cc9f2d02c55bbbc4a00bcc2a3
```

##### 6.2 查看node和pod信息

执行结束后应该是不能使用kubectl查看node和pod信息的，如图：

![image-20191120223733199](%E6%90%AD%E5%BB%BAk8s%E9%9B%86%E7%BE%A4.assets/image-20191120223733199.png)

需要执行以下操作：

拷贝master的/etc/kubernetes/admin.conf到slave的/etc/kubernetes/目录下

 ```shell
# 从master传到node节点
scp  /etc/kubernetes/admin.conf likangvm3@192.168.124.231:/tmp/

# 在node节点上配置kubeconfig环境变量
cp /tmp/admin.conf /etc/kubernetes/
export KUBECONFIG=/etc/kubernetes/admin.conf 
 ```

或者

```shell
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source ~/.bash_profile
```

##### 6.3 查看pod不成功的原因，大多是因为镜像拉取不下来

现在可以通过以下命令检测node和pod运行状态：

```shell
kubectl get node # 检查node是否ready
# 检查所有pod是否正常
kubectl get pod --all-namespaces -o wide
#如果pod处于非running状态，则查看该pod：
kubectl describe pod xxxxx -n kube-system
# 如果是因为镜像无法下载导致镜像启动失败，则手动在node上pull image，然后在master上删掉对应的pod，k8s会重新调度pod
# 删除pod命令
kubectl delete pod xxxx -n kube-system
```



如果是因为镜像拉取不下来，还按照之前的方法，打tag：

```shell
docker pull aiotceo/kube-proxy:v1.16.3
docker pull aiotceo/pause:3.1

docker tag aiotceo/pause:3.1 k8s.gcr.io/pause:3.1
docker tag aiotceo/kube-proxy:v1.16.3 k8s.gcr.io/kube-proxy:v1.16.3
```



#### 7.验证

```shell
kubectl get nodes

# NAME           STATUS   ROLES    AGE   VERSION
# likangvm1-pc   Ready    master   11h   v1.16.3
# likangvm2-pc   Ready    <none>   11h   v1.16.3
# likangvm3-pc   Ready    <none>   24m   v1.16.3

kubectl get pods --all-namespaces

# NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
# kube-system   coredns-5644d7b6d9-9dprc               1/1     Running   0          11h
# kube-system   coredns-5644d7b6d9-ljv5w               1/1     Running   0          11h
# kube-system   etcd-likangvm1-pc                      1/1     Running   0          11h
# kube-system   kube-apiserver-likangvm1-pc            1/1     Running   0          11h
# kube-system   kube-controller-manager-likangvm1-pc   1/1     Running   0          11h
# kube-system   kube-proxy-qpvtb                       1/1     Running   0          25m
# kube-system   kube-proxy-v2xnb                       1/1     Running   0          11h
# kube-system   kube-proxy-wkxzg                       1/1     Running   0          11h
# kube-system   kube-scheduler-likangvm1-pc            1/1     Running   0          11h
# kube-system   weave-net-6nj4c                        2/2     Running   0          25m
# kube-system   weave-net-lm6dh                        2/2     Running   0          37m
# kube-system   weave-net-vwnc2                        2/2     Running   0          37m
```

截图如下：

![image-20191121144548803](%E6%90%AD%E5%BB%BAk8s%E9%9B%86%E7%BE%A4.assets/image-20191121144548803.png)



#### 遇到的问题

![image-20191120223405712](%E6%90%AD%E5%BB%BAk8s%E9%9B%86%E7%BE%A4.assets/image-20191120223405712.png)

需要关闭Swap，执行以下命令：

```shell
swapoff -a
```



##### 查看日志

```shell
journalctl -f -u kubelet.service
```

