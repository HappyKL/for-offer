# 知识图谱（尚硅谷）

## 1.介绍说明

### 前世今生

### 组件说明

![image-20200214162349831](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214162349831.png)

![image-20200214162436997](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214162436997.png)

apiserver：所有服务的访问统一入口

controllermanager：维持副本期望数量

scheduler：选择合适的节点进行分配任务

etcd：键值对数据库，储存K8s集群所有的重要信息

kubelet：直接更容器引擎交互实现容器的生命周期管理

kube-proxy：负责写入规则至iptables，ipvs，实现服务映射访问

coredns：可以为集群中的svc创建一个域名IP的对应关系解析

Dashboard:给K8s集群提供一个B/S结构访问体系

Ingress controller：官方只能实现四层代理，ingress可以实现七层代理

federation：提供一个可以跨集群中心多k8s统一管理功能

prometheus：提供k8s集群的监控能力

elk：提供k8s集群日志统一分析接入平台



## 2.基础概念

### Pod概念

Pod中容器共享网络栈和存储卷，首先启动一个pause容器

### 控制器类型

rc，rs，deployment，statefulSet，DameonSet，HPA，Job，CronJob

### 网络通讯模式

同一个Pod内的多个容器：lo

各个Pod之间：Overlay Network（flannel）

Pod与Service之间的通讯：各节点的Iptables



flannel：

![image-20200214163646442](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214163646442.png)

Etcd的作用：

存储管理Flannel可分配的IP地址段资源

监控Etcd中每个Pod的实际地址，建立维护Pod节点路由表



不通情况下的网络通信方式：

![image-20200214163848286](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214163848286.png)



组件通讯示意图：（只有节点网络是真实网络）

![image-20200214164017216](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214164017216.png)

## 3.K8s安装

### 搭建K8s集群

## 4.资源清单

### 资源

![image-20200214164350380](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214164350380.png)

![image-20200214164406373](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214164406373.png)

### 资源清单语法（yaml）

### 编写Pod

### Pod生命周期

![image-20200214164508255](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214164508255.png)

InitC：运行MainC之前的一些准备，比如拷贝文件等

探针：readiness（就绪检测：刚启动时，可能容器还无法访问，但显示running，此时显示未ready）   liveness（存活检测：定时检查容器状态，如果检测失败，则会按照重启策略进行后续操作）

postStart：在开启容器之前的操作

preStop：在容器退出前的操作



init模版

![image-20200214165937191](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214165937191.png)

liveness模版

![image-20200214170030773](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214170030773.png)

readiness模版

![image-20200214170102266](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214170102266.png)

start和stop模版

![image-20200214170246183](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214170246183.png)



## 5.资源控制器

### 掌握各种控制器特点以及使用方式

rc，rs，deployment，statefulSet，DameonSet，HPA，Job，CronJob



Pod的分类

​	自主式Pod：Pod退出了，此类型的Pod不会被创建

​	控制器管理的Pod：在控制器的生命周期里，始终要维持Pod的副本数量



### rs

比rc：支持selector集合(通过lables)，rc已经弃用

rs模版

![image-20200214165822161](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214165822161.png)

### Deployment

提供了声明式定义方法，与RS的区别：支持滚动更新

![image-20200214171009931](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214171009931.png)

deployment模版，与RS几乎一致

![image-20200214170933818](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214170933818.png)

### DaemonSet

模版：

![image-20200214172856561](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214172856561.png)

 ### Job

![image-20200214173216217](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214173216217.png)

### CronJob

![image-20200214174238268](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200214174238268.png)



## 6.服务发现

### 掌握svc原理及其构建方式

svc通过标签匹配Pod

### 四种访问方式

clusterIP：也可以使用ClusterIP：None（可通过域名访问，serverless）

NodePort：

LoadBalancer：LB+NodePort

ExternalName：



### service代理模式

userspace

iptables

ipvs（目前正在使用）



### Ingress

service支持四层，而Ingress支持七层，本质就是通过给负载均衡器自动更新配置文件。比如Ingress-nginx，先安装Ingress-nginx-control，包括deployment和对应的service，通过官网可以进行安装；之后可以配置域名和服务的对应关系通过Ingress资源的rules字段进行定义；最终可以通过域名（如果没有配DNS，可以在hosts文件里配置域名和IP的对应关系）访问对应服务。



## 7.存储

### 掌握多种存储类型特点并且能够在不同环境中选择合适的存储方案

### ConfigMap

支持三种方式：文件导入，目录导入，字面值

使用方式：环境变量注入，可以通过注入的环境变量设置命令行参数

### Screct

三种类型

Opaque：base64 编码格式的 Secret，用来存储密码、密钥等；但数据也通过 base64 --decode 解码得到原始数据，所有加密性很弱。

`kubernetes.io/dockerconfigjson`：用来存储私有 docker registry 的认证信息。

`kubernetes.io/service-account-token`： 用于被 serviceaccount 引用。serviceaccout 创建时 Kubernetes 会默认创建对应的 secret。Pod 如果使用了 serviceaccount，对应的 secret 会自动挂载到 Pod 的 `/run/secrets/kubernetes.io/serviceaccount` 目录中，serviceaccount 用来使得 Pod 能够访问 Kubernetes API。目录下有token，ca.crt，namespace





使用 ：Opaque类型

用于环境变量，数据卷挂载



### Volume

类型很多：emptyDir , hostPath(灵活度高) ......

emptyDir：容器挂掉不会丢失数据，但Pod被删除后，emptyDir也会被删除。



### PV&PVC

静态PV

动态PV

PVC和PV绑定



以NFS为例

PV创建时可以设置到NFS

StatefulSet创建时设置有PVC模版，假设Pod副本有三个，则每个Pod都会有一个PVC，每个PVC会绑定一个PV

这样，每次StatefulSet创建后都会有稳定的存储



删除时：手动删除PVC，手动释放PV中绑定的PVC才会将PV的状态变为avalible 



### StatefulSet

有序部署：通过Init Containers来实现

有序删除

稳定的持久化存储：Pod重新调度后还是能访问到相同的内容

稳定的网络标识符：Pod重新调度后PodName



StatefulSet为每个Pod副本创建了一个DNS域名，这个域名是Podname.headless server name，意味着服务间通过Pod域名来通信而非Pod IP，因此当Pod所在Node发生故障时，Pod会被飘逸到其他Node上，虽然Pod Ip会发生变化，但Pod域名不会有变化



StatefulSet使用Headless服务来控制Pod的域名，这个域名的FQDN为：service neme.namespace.svc.cluster.local，其中，cluster.local指的是集群的域名

```shell
dig -t A ervice neme.namespace.svc.cluster.local @$(coreosdns.ip)
```



## 8.调度器

### 掌握调度器原理，能够根据要求把Pos调度到相应的node

### 预选+优选

预选：资源是否够用，标签是否匹配，port是否冲突，volume是否冲突等等

优选：资源消耗最小的权重高，不同资源使用率接近的权重高，已经有要使用镜像的节点权重越高

### 亲和性

软策略prefer/硬策略required

节点亲和性：标签匹配节点，In，NotIn...

Pod亲和性：和指定标签的节点在同一个Node；

Pod反亲和性：和指定标签的节点不在同一个Node

### 污点和容忍

污点taint

effect:

noSchedule:不调度

preferNoSchedule：尽量不调度

noExecute：不调度，且驱逐已有的Pod

```shell
# 打污点
kubectl taint nodes nodeName key1=val1:effect
# 查看污点
kubectl describe node node-name
# 消除污点
kubectl taint nodes nodeName key1=val1:effect-
```



容忍toleration

```yaml
# pod.spec.toleration
- key:
	operator:
	value:
	effect:
	tolerationSeconds:
```

当不指定Key时，表示容忍所有的污点

当不指定effect时，表示容忍所有的污点作用

当多个Master存在时，防止资源浪费，可以设置master的污点作用为PreferNoSchedule

```shell
kubectl taint nodes nodeName node-role.kubernetes.io/master:PreferNoSchedule
```



### 固定节点调度

设置Pod.spec.nodeName

或者

设置Pod.spec.nodeSelector

## 9.集群安全机制

### 集群的认证，鉴权，访问控制 原理及其流程

### 机制说明

围绕保护API Server设计，使用了认证，鉴权，准入控制三步来保证

### 认证

#### 认证策略

Http Token认证：通过一个Token来识别合法用户

当客户端发起API调用请求时，需要在HTTP Header里放入Token，每一个Token对应一个用户存储在API Server能访问的文件中，从而验证是否合法。

HTTP base认证：通过用户名和密码

放入HTTP Request中的Heather Authorization域里，服务器收到后base64解码获取用户名密码，然后验证

目前通常采用的方式：**最严格的HTTPS证书认证**：基于CA根证书签名的客户端身份认证方式 (双向认证方式)，客户端服务端均有证书（证书里包含ca加密的公钥）和私钥

![image-20200309150108512](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200309150108512.png)



#### 认证节点：

1.k8s组件：kubectl , controller Manager , Scheduler , kubelet ,kube-proxy

2.管理的某些Pod：比如DashBorad



#### 安全性说明

Controller Manager，Scheduler与Api Server在同一台机器，直接启用API Server的非安全端口访问即可，--insecure-bind-address=127.0.0.1

kubectl ，kubelet，kube-proxy访问API Server都需要进行HTTPS双向认证



#### 证书颁发

手动签发：通过k8s集群的根ca签发https证书

自动签发：kubelet首次访问API Server时，使用token做认证，通过后，Controller Manager会为kubelet生成一个证书，以后的访问都是用证书做认证了



#### kubeconfig

kubeconfig文件包含集群参数（CA证书，API Server地址），客户端参数（上面生成的证书和私钥），集群context信息（集群名称，用户名），Kubernetes组件通过启动时指定不同的kubeconfig文件可以切换到不同的集群



#### ServiceAccount

Pod中的容器访问API Server，因为Pod的创建，销毁是动态的，所以要为他生成证书不可行，kurbenetes使用了Service Account解决Pod访问API Server的认证问题



#### secret和SA的关系

k8s设计了一种资源对象是secret，分为俩类：一种是用于ServiceAccount的service-account-token，另一种用于保存用户自定义保密信息的Qqaque，serviceAccount中包含三个部分：token, ca.crt,namespace

Token:用于APIserver私钥签名的JWT，用于访问api server时，server端认证

ca.crt：根证书，用于Client端验证API server发送的证书

namespace，标识这个service-account-token的作用域空间

#### 总结

![image-20200309151113817](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200309151113817.png)



### 鉴权

认证只是确认了双方可以互相通信。

鉴权是确定请求方有哪些资源的权限

#### 授权策略

API Server目前有以下几种鉴权策略，通过API Server的启动参数 -authorization-mode进行设置：

AlwaysDeny:拒绝所有请求

AlwaysAllow：允许接受所有请求

ABAC：基于属性的访问控制，表示使用用户配置的授权规则对用户请求进行匹配和控制

Wehook：通过调用外部REST服务对用户进行授权

**RBAC**（Role-Based Acess Control）基于角色，目前默认策略



#### RBAC

优势

对集群中的资源和非资源均有完整地覆盖

整个RBAC完全由几个API对象完成，同其他API对象一样，可以用kubectl或API进行操作

可以在运行时进行调整，无需重启api server

- RBAC的API资源对象说明

  RBAC引入了4个全新的资源对象：Role，ClusterROle，RoleBinding，ClusterRoleBinding

  ![image-20200221011216463](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200221011216463.png)

- ![image-20200221012217958](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200221012217958.png)

- Role/ClusterRole RoleBinding/ClusterRoleBinding

- resource子资源 pods/log

- to subject

- 举例：创建一个用户只能管理dev空间（讲了生成kubeconfig文件过程）

### 准入控制

![image-20200221015505264](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200221015505264.png)





## 10.HELM

### 类似于Linux中的yum 

### 掌握HELM原理 

#### 两个概念：chart和release

chart是创建一个应用的信息集合，包括各种Kubernetes对象的配置模版，参数定义，依赖关系，文档说明，chart是应用部署的自包含逻辑单元，可以将chart想象成apt，yum中的软件安装包

release是chart的运行实例，代表一个正在运行的应用，当chart被安装到Kubernetes集群，就生成了一个release，chart能够多次安装到同一个集群，每次安装就是一个release

#### 两个组件：Helm客户端和Tiller服务器

### HELM模版自定义  

chart.yaml      template.      Values.yaml

### 部署一些常用插件

#### 举例：使用helm部署dashboard

#### 举例：使用helm部署prometheus + 演示hpa资源对象



#### 小知识：资源设置

三种情况：

基于Pod设置（对Pod的限制）：requests/limits

基于名称空间：resourceQuota：对名称空间的限制

limitRange：如果Pod没设置，则使用名称空间下最大的限制，如果名称空间也没设置限制，则会采用limitRange资源对象的设置，以方资源耗尽出现oom等异常



#### 部署日志收集 EFK



## 11.运维

### kubeadm源码修改（修改证书年限到10年）





### Kubernetes高可用创建



![image-20200221174413250](K8s%E5%AD%A6%E4%B9%A0.assets/image-20200221174413250.png)

- k8s1.5之后etcd自动形成集群(集群已解决)
- controller，scheduler表现为主备(集群已解决)
- 只需要进行apiserver的管理(可以使用nginx+keepalive)