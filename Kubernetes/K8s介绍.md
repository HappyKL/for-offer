

## 作用

容器的自动化复制和部署，随时扩展和收缩容器规模，提供负载均衡



方便容器升级



提供容器弹性，失效就替换



# 1.K8s基本概念

## .1 Master 

- 组件：API Server，Controller Manager，Scheduler，Etcd

##  .2 Node

- 组件 ：kubelet，kube-proxy

- 查看Node信息

```shell
# 可查看Node的基本信息：名称，标签，创建时间等
# 可查看Node的当前状态：Node启动后会做一系列的自检工作，比如磁盘空间是否不足(DiskPressure),内存是否不足（MemoryPressure），网络是否正常（NetworkUnavailable），PID资源是否充足（PIDPressure），在一切正常时设置Node为Ready状态，该状态表示节点处于正常状态，Master可以在上面调度新的任务了（比如启动Pod）
# Node的主机地址和主机名
# Node上的资源数量：描述Node可用的系统资源，比如CPU，内存数量，最大可调度Pod数量等
# Node可分配的资源量：描述Node当前可分配的资源量
# 主机系统信息：包括主机ID，系统UUID，Linux kernel版本号，操作系统类型与版本，Docker版本号，kubelet与kube-proxy的版本号等
# 当前运行的Pod列表概要信息
# 已分配的资源使用概要信息：例如资源申请的最低最大允许使用量占系统总量的百分比
# Node相关的Event信息
kubectl describe node node-name
```

## .3 Pod 

- 根容器：Pause容器

  每个Pod都有一个特殊的被称为“根容器”的Pause容器，除了Pause容器，Pod还包含一个或多个紧密相关的用户业务容器

  为何Pod要设置Pause容器：

  - 在一组容器作为一个单元的情况下，我们难以简单的对“整体”进行判断及有效行动。引入与业务无关并且不易死亡的**Pause容器作为Pod的根容器，以它的状态代表整个容器组的状态**，就简单巧妙的解决了这个难题
  - Pod里的**多个业务容器共享Pause容器的IP，共享Pause容器挂接的Volume**，这样既简化了业务容器间的通信问题，也解决了他们之间的文件共享问题

- 网络

  每个Pod都分配了唯一的IP地址，被称为Pod IP，一个Pod里面的多个容器共享那个Pod IP地址，K8s要求底层网络支持集群内任意两个Pod之间的TCP/IP直接通信，这通常采用虚拟二层网络技术来实现，例如Flannel等，也就是说，**在K8s中，一个Pod里的容器与另外主机上的Pod容器能够直接通信**。

- 类型

  - 普通Pod：一旦创建会**存放到etcd中，随后Master会调度到某个Node上进行绑定，随后该Pod被对应的Node上的kubelet进程实例化成一组相关的Docker容器并启动**。当Pod里某个容器停止时，K8s会自动检测到这个问题并且重新启动这个Pod（重启Pod里的所有容器），如果Pod所在的Node宕机，就会将这个Node上所有的Pod重新调度到其他节点。
  - 静态Pod：不存放在etcd中，**存放在某个具体的Node上的一个具体文件中，并且只在此Node上启动，运行**

- Endpoint

  Pod的IP加上容器端口（containerPort），组成了新概念-Endpoint，它代表此Pod里的一个服务进程的对外通信地址。一个Pod也存在具有多个Endpoint的情况，比如当我们把Tomcat定义为一个Pod时，可以对外暴露管理端口与服务器端口这两个Endpoint。

- Pod Volume

  可以用分布式文件系统GlusterFS实现后端存储功能，Pod Volume是被定义在Pod上，然后被各个容器挂载到自己的文件系统中的

- 使用Event排查故障

  Event记录了事件的最早产生时间，最后重现时间，重复次数，发起者，类型，以及导致此事件的原因等众多信息。Event通常会被关联到某个具体的资源对象中，比如Pod，当Pod无法创建时，可以通过以下命令来查看信息，来定位故障原因

  ```shell
  kubectl describe pod xxxx
  ```

- Pod的资源设置

  当前可设置限额的计算资源有CPU和Memory两种，CPU资源单位为CPU(Core)的数量，是一个绝对值而非相对值，CPU通畅与千分之一的CPU配额为最小单位，用m表示，通常一个容器配额被定义为100～300m，即占用0.1～0.3个cpu；Memory配额单位时内存字节数

  - Requests：该资源的最小申请量，系统必须满足要求。通常设置Requests符合容器平时的工作负载情况下的资源需求
  - Limits：该资源最大允许使用量，不能被突破，当容器试图使用超过这个量的资源时，可能会被K8s杀掉并重启。通常设置Limits为峰值负载情况下资源占用的最大量

  比如以下配置

  ```yaml
  spec:
    containers:
    - name: db
      image: mysql
      resources:
        resuests:
          memory: "63Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"
  ```

## .4 Label

- 一个Label是一个key=value的键值对，key和value由用户自己指定

- Label可以附加到各种资源对象上，例如Node，Pod，Service，RC等，一个资源对象可以定义多个Label。Label可以在对象定义时添加，也可以在对象创建后动态添加

- Label Selector有两种方式：基于等式和基于集合

- Label Selector作用

- - kube-controller进程通过在资源对象RC上定义的Label Selector来筛选要监控的Pod副本数量，使Pod副本数量始终符合预期设定的全自动控制流程

  - kube-proxy进程通过Service的Label Selector来选择对应的Pod，自动建立每个Service到对应的Pod的强求路由表，从而实现Service的智能负载均衡机制
  - 通过对某些Node定义的特定的Label，并且在Pod定义文件中使用NodeSelector这种标签调度策略，kube-scheduler进程可以实现Pod定向调度的特性

## .5 Replication Controller (RC)



node是用于部署应用容器的服务器

Pod是基本的操作单元，也是应用运行的载体。整个Kubernetes系统都是围绕着Pod展开的，比如如何部署运行Pod，如何保证Pod的数量，如何访问Pod等

Deployment定义了Pod部署的信息

若干个Pod组成一个Service，对外提供服务



