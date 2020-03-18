# 深入剖析Kubernetes

### 08 重新认识Docker容器



#### 数据卷Volume

- 允许将宿主机上指定的目录或者文件，挂载到容器里面进行读取和修改操作

- docker中，支持两种Volume，可以把宿主机目录挂载进容器的/test目录当中

  ```shell
  docker run -v /test ... #docker默认在宿主机上创建一个临时目录/var/lib/docker/volumes/[VOLUME_ID]/_data，挂载到容器的/test目录
  docker run -v /home:/test #docker直接把宿主机的/home目录挂载到容器/test目录
  ```

- 在rootfs准备好之后，在执行chroot之前，把volume指定的宿主机目录挂载到容器目录即可



### 14. 深入解析Pod对象(一)：基本概念

- 个人理解：讲了Pod和Container的核心字段

#### 重要字段

- 凡是调度，网络，存储，以及安全相关的属性，基本都属于Pod级别

- NodeSelector:一个供用户将Pod与Node进行绑定的字段

  ```yaml
  apiVersion: v1
  kind: Pod
  ...
  spec:
   nodeSelector:
     disktype: ssd
  ```

  - 意味着该Pod会运行在携带“disktype:ssd"标签的节点上，否则，它将调度失败

- NodeName：一旦Pod这个字段被赋值，k8s就会认为这个Pod赢你经过了调度，调度的结果就是赋值的节点名字

- HostAliases：定义了Pod的hosts文件里的内容

  ```yaml
  apiVersion: v1
  kind: Pod
  ...
  spec:
    hostAliases:
    - ip: "10.1.2.3"
      hostnames:
      - "foo.remote"
      - "bar.remote"
  ...
  ```

  - 在k8s中，如果要设置hosts文件里的内容，一定要通过这种方法，否则，如果直接修改了hosts文件的话，在Pod被删除重建之后，kubelet会自动覆盖掉被修改的内容

- 凡是跟容器的Linux Namespace相关的属性，也一定是Pod级别的

  - shareProcessNamespace

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      shareProcessNamespace: true
      containers:
      - name: nginx
        image: nginx
      - name: shell
        image: busybox
        stdin: true
        tty: true
    ```

    - 意味着这个Pod里的容器共享PID Namespace

- 凡是Pod中的容器要共享宿主机的Namespace，也一定是Pod级别的定义

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
  spec:
    hostNetwork: true
    hostIPC: true
    hostPID: true
    containers:
    - name: nginx
      image: nginx
    - name: shell
      image: busybox
      stdin: true
      tty: true
  ```

  - 在这个Pod里，我定义了共享宿主机的Network，IPC，和PID Namespace。这意味着，这个Pod里所有的容器，会直接使用宿主机的网络，直接与宿主机进行IPC通信

- container

  - ImageOullPolicy：always，never，ifnotpresent

  - lifecycle

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: lifecycle-demo
    spec:
      containers:
      - name: lifecycle-demo-container
        image: nginx
        lifecycle:
          postStart:
            exec:
              command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
          preStop:
            exec:
              command: ["/usr/sbin/nginx","-s","quit"]
    ```

#### pod对象的生命周期

- Pod生命周期的变化，主要体现在Pod API对象的status部分，pod.status.phase，就是pod的当前状态
  - Pending：Pod的yaml文件已经提交给了k8s，API对象已经被创建并保存在ETCD当中，但这个Pod里有些容器因为某些原因不能被顺利创建，比如调度不成功
  - Running：Pod已经调度成功，包含的容器都已经创建成功
  - succeeded：Pod里所有容器都正常执行完毕，并且已经退出
  - Unknown：异常状态，Pod的状态不能持续的被Kubelet汇报给kube-apiserver，这很有可能主从节点之间通信发生了问题

### 15.深入解析Pod对象（二）：使用进阶



#### Secret

- Secret可以将Pod想要访问的加密数据存放到Etcd中，然后通过在Pod的容器里挂载Volume的方式，访问到这些Secret里保存的信息了

#### ConfigMap



#### Downward API

让Pod里的容器能够直接获取到这个Pod API对象本身的信息

#### ServiceAccountToken

用于



#### 容器健康检查和恢复机制





### 18 深入理解StatefulSet (一) ：拓扑状态（只看了headless）

#### StatefulSet

- 定义：实例之间不对等，实例对外部数据有依赖关系的应用，本称为“有状态应用”（Stateful Application）

- StatefulSet把真实世界的应用状态抽象为两种情况：
  - 拓扑状态：Pod实例启动必须按照某系顺序；当被删除后，重新创建时也必须按照这个顺序。并且新创建出来的Pod，必须和Pod的网络标识一样，这样原先的访问者才能使用同样的方法访问到这个新Pod
  - 存储状态：多个Pod实例绑定了不同的存储数据，Pod第一次读取到的数据和隔了十分钟之后再次读取到的数据，应该是同一份，哪怕在此期间Pod A被重新创建过。典型的例子，就是一个数据库应用的多个存储实例

- 核心功能：通过某种方式记录这些状态，然后在Pod被重新创建时能够为新Pod恢复这些状态

#### 工作原理

- Headless Service

  - Service是如何被访问的呢？
    - VIP：当访问VIP时，将请求转到该service所代理的某一个Pod上
    - DNS方式：比如，访问"my-svc.my-namespace.svc.cluster.local"这条DNS记录时，就可以访问到my-svc的Service所代理的某一个Pod
      - Normal Service：访问my-svc.my-namespace.svc.cluster.local解析到的就是my-svc这个service的VIP，其实还是VIP的方式
      - Headless Service：访问my-svc.my-namespace.svc.cluster.local解析到的，直接就是my-svc代理的某一个Pod的IP地址
      - 区别：Headless Service不需要分配一个VIP，而是可以直接以DNS记录的方式解析出被代理的Pod的IP地址

  - Headless Service的定义方式

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      ports:
      - port: 80
        name: web
      clusterIP: None
      selector:
        app: nginx
    ```

    当按照这样的方式创建一个Headless Service之后，它所代理的所有Pod的IP地址就会被绑定一个这样格式的DNS记录：

    ```text
    <pod-name>.<svc-name>.<namespace>.svc.cluster.local
    ```

    这个DNS记录，就是k8s为Pod分配的唯一“可解析身份”，因此只要知道Pod名字和它对应的service名字，就可以访问到Pod的IP地址

- StatefulSet如何使用这个DNS记录来维持Pod的拓扑状态呢？

  ```yaml
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: web
  spec:
    serviceName: "nginx"
    replicas: 2
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.9.1
          ports:
          - containerPort: 80
            name: web
  ```

  - 与deployment的唯一区别就是多了一个serviceName=nginx字段，这个字段就是告诉StatefulSet控制器，在执行控制循环的时候，请使用nginx这个Headless Service来保证Pod的“可解析身份”
  - 



## 容器持久化

### 28 PV、PVC、StorageClass

- 个人理解：用户请求创建Pod后，k8s发现这个Pod声明的PVC，然后就靠PersistentVolumeController帮它找一个PV来配对；如果没有现成的PV，就会找对应的storageClass，然后会新建一个PV，然后和PVC绑定
- 个人理解：创建PV需要经过两阶段处理变成宿主机上持久化Volume
  - 第一阶段Attach，需要由master节点上的AttachDetachController负责，为宿主机挂载远程磁盘
  - 第二阶段Mount，运行在kubelet组件内部，把挂载的磁盘mount到宿主机目录，运行在独立的goroutine，不会阻塞kubelet主循环
  - 完成这两步后，kubelet把Volume目录通过CRI里的Mounts参数，传递给Docker，然后就可以为Pod里的容器挂载这个“持久化”的Volume了



#### PV

- 该API对象定义的是一个持久化存储在宿主机上的目录，比如一个NFS的挂载目录

- 通常PV对象由运维人员事先创建在k8s集群里待用

- 例子

  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: nfs
  spec:
    storageClassName: manual
    capacity:
      storage: 1Gi
    accessModes:
      - ReadWriteMany
    nfs:
      server: 10.244.1.4
      path: "/"
  ```

#### PVC

- Pod所希望使用的持久化存储的属性

- 由开发人员创建，或者以PVC模板的方式成为StatefulSet的一部分，然后由StatefulSet控制器负责创建带编号的PVC

- 例子

  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: nfs
  spec:
    accessModes:
      - ReadWriteMany
    storageClassName: manual
    resources:
      requests:
        storage: 1Gi
  ```

- 用户创建的PVC要真正被容器使用起来，就必须先和某个符合条件的PV进行绑定，检查的条件包括两部分：

  - PV和PVC的spec字段，比如，PV的存储大小，必须满足PVC的要求
  - PV和PVC的storageClassName字段必须一样

- 将PV和PVC绑定之后，Pod就能像hostPath等常规类型的Volume一样使用这个PVC了

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      role: web-frontend
  spec:
    containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
      volumeMounts:
          - name: nfs
            mountPath: "/usr/share/nginx/html"
    volumes:
    - name: nfs
      persistentVolumeClaim:
        claimName: nfs
  ```

  - Pod需要做的就是在volumes字段里声明自己要使用的PVC名字。
  - Pod创建时，kubelet就会把这个PVC对应的PV，也就是一个NFS类型的Volume，挂载在这个Pod容器内的目录里

#### PV和PVC的绑定

- 存在专门处理持久化存储的控制器，叫做Volume Controller，这个Volume Controller维护着多个控制循环，有一个循环，就是撮合PV和PVC，他的名字叫做PersistentVolumeController
- PersistentVolumeController会不断查看当前每一个PVC，是不是已经处于绑定状态，如果不是，它就会遍历所有的，可用的PV，并尝试与PVC进行绑定
- 绑定其实就是将这个PV的名字填写在PVC对象spec.volumeName字段上

#### PV变成容器的持久化存储

- 容器的Volume：将一个宿主机上的目录，跟一个容器里的目录绑定挂载在了一起

- 持久化的Volume：宿主机上的目录，具备持久性。不会因为容器的删除而被清理掉，也不会跟当前宿主机绑定

- 持久化Volume的实现：往往依赖于一个远程存储服务，比如：远程文件存储（比如，NFS、GlusterFS）等

- PV具体原理：两阶段处理

  - Attach：为虚拟机挂载远程磁盘

    ```shell
    $ gcloud compute instances attach-disk <虚拟机名字> --disk <远程磁盘名字>
    ```

  - Mount：将磁盘设备格式化并挂载到Volume宿主机目录

    ```shell
    # 通过lsblk命令获取磁盘设备ID
    $ sudo lsblk
    # 格式化成ext4格式
    $ sudo mkfs.ext4 -m 0 -F -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/<磁盘设备ID>
    # 挂载到挂载点
    $ sudo mkdir -p /var/lib/kubelet/pods/<Pod的ID>/volumes/kubernetes.io~<Volume类型>/<Volume名字>
    ```

- 挂载到容器:kubelet把Volume目录通过CRI里的Mounts参数，传递给Docker，然后就可以为Pod里的容器挂载这个“持久化”的Volume了，相当于执行了如下命令：

  ```shell
  $ docker run -v /var/lib/kubelet/pods/<Pod的ID>/volumes/kubernetes.io~<Volume类型>/<Volume名字>:/<容器内的目标目录> 我的镜像 ...
  ```

- 删除PV时，需要Unmount和Dettach两个阶段，执行反向操作即可
- PV的处理流程与Pod以及容器的启动流程没有耦合，只要kubelet在向Docker发起CRI请求之前，确保“持久化”的宿主机目录已经处理完毕即可

#### PV两阶段处理流程

- pv两阶段处理流程是靠kubelet主控循环(kubelet sync loop)之外的两个控制循环来实现的
- Attach
  - 由Volume Controller负责维护，控制循环名字叫：AttachDetachController，作用就是不断检查每一个Pod对应的PV，和这个Pod所在宿主机之间的挂载情况，从而决定是否需要对这个PV进行Attach(Dettach)操作
  - Volume Controller是kube-controller-manager的一部分，因此AttachDetachController也是运行在Master节点上的
- Mount
  - 发生在Pod对应的宿主机上，必须是kubelet组件的一部分，这个控制循环的名字，叫做：VolumeManagerReconciler，运行起来之后，是一个独立于Kubelet主循环的Goroutine

#### StorageClass

- 需求
  - 运维人员创建PV，大规模集群里开发人员可能会创建很多PVC，对应着需要很多PV，手动创建很难
  - 动态创建PV（Dynamic Provisioning）机制工作的核心，在于一个名叫StorageClass的API对象
- StorageClass对象
  - 作用：创建PV模板
  - 定义
    - PV属性，比如存储类型，Volume大小等
    - 创建这种PV需要用到的存储插件，比如，Ceph等
  - k8s根据用户提交的PVC，找到一个对应的StorageClass，然后调用StorageClass声明的存储插件，创建出需要的PV

### 48 Prometheus，Metrics Server与kubernetes监控体系

- 个人理解-Prometheus监控体系：exporter和pushgateway采集数据，alertManager报警管理，使用PromQL对监控数据可视化
- 个人理解-Metrics数据来源：分为三类：NodeExporter，各个组件的Metrics API，核心监控数据(容器，Pod，Service等)。容器数据由cAdvisor提供，该服务在Kubelet启动时启用；其他核心监控数据通过Metrics Server提供的k8s API来获取
- 个人理解-Metrics Server：通过Aggregator机制，让api server和Metrics server共同成为aggregator代理的后端，aggregator可以根据URL选择具体的API后端代理服务器，通过这种方式，可以很方便的扩展K8sAPI了

#### prometheus

- 示意图

  ![image-20200311111019903](%E6%9E%81%E5%AE%A2%E6%97%B6%E9%97%B4k8s.assets/image-20200311111019903.png)

- Metrics数据来源

  - 宿主机的监控数据：由Pormetheus维护的Node Exporter工具，以DaemonSet方式运行在宿主机上，可以给Primetheus采集的数据不仅仅是节点负载，CPU，内存，磁盘以及网络这样的常规信息，可以参考https://github.com/prometheus/node_exporter#enabled-by-default

  - k8s的API Server，kubelet等组件的/metrics API，除了常规的cpu，内存等信息，还包括各个组件的核心监控指标，比如，对于API Server来说，它就会在/metrics API里，暴露出各个Controller的工作队列的长度，请求的QPS，和延迟数据等

  - k8s核心监控数据：Pod，Node，容器，service等主要k8s核心概念的Metrics

    - 容器相关的Metrics来自kubelet内置的cAdvisor服务，kubelet启动后，cAdvisor服务也随之启动，提供的信息可以细化到每个容器的PCU，文件系统，内存，网络等资源的使用情况

    - k8s的核心监控数据主要是用k8s的扩展能力，Metrics Server

      - 需求：Metrics Server是用来取代Heapster这个项目的，它把Heapster可以监控的数据通过标准的k8s API暴露出来，Metrics信息与Heapster完成解耦，Heapster慢慢退出舞台

      - 使用：标准的k8s API，比如以下可以返回一个Pod的监控数据：

        ```text
        http://127.0.0.1:8001/apis/metrics.k8s.io/v1beta1/namespaces/<namespace-name>/pods/<pod-name>
        ```

        这些数据，是从kubelet的summary API (即<kubelet_ip>:<kubelet_port>/stats/summary)采集而来，summary API返回的信息，即包括cAdvisor的监控数据，也包括k8s本身汇总的信息

      - 原理：Metrics Server并不是kube-apiserver的一部分，而是通过Aggregator这种插件机制，在独立部署的情况下同kube-apiserver一起统一对外服务的

        ![image-20200311112919106](%E6%9E%81%E5%AE%A2%E6%97%B6%E9%97%B4k8s.assets/image-20200311112919106.png)

        在这个机制下，你还可以添加更多的后端给这个 kube-aggregator。所以 kube-aggregator 其实就是一个根据 URL 选择具体的 API 后端的代理服务器。通过这种方式，我们就可以很方便地扩展 Kubernetes 的 API 了

      - Aggergator模式开启(略)

#### Prometheus使用

- 将Pormetheus Operator在k8s中部署起来
- 把Metrics源配置起来，让Prometheus自己去进行采集即可

### 49 Custom Metrics: 让Auto Scaling不再“食之无味”

- 个人理解-custom metrics：Custom Metrics APIServer与Metrics Server一样，通过Aggregator APIServer扩展机制实现，来提供API。Custom Metrics APIServer其实就是一个Prometheus的Adapter
- 个人理解-使用HPA+Custom Metrics实现自动扩展：通过HPA来设置扩展策略，metrics字段指定依据指标；Custom Metrics通过Prometheus会获取到指标，Prometheus通过ServiceMonitor来监控指定应用，指定应用要提供Metrics接口

#### Custom Metrics

- 需求：Auto Scaling，自动水平扩展，这个功能往往要依据某项资源的情况，比如CPU，内存等，但有时往往需要一虚自定义的监控指标，比如某个应用的等待队列长度等

  - 凭借强大的API扩展能力，Custom Metrics已经成为k8s一项标准能力
  - K8s的自动扩展器组件Horizontal Pod Autoscaler（HPA），也可以直接使用Custom Metrics来执行用户指定的扩展策略

- 扩展原理：与Metrics Server一样，也是借助Aggregator APIServer扩展机制来实现的，当Custom Metrics APIServer启动之后，k8s里会出现一个叫做custom.metrics.k8s.io的API，当访问这个URL时，Aggregator就会把请求转发给Custom Metrics APIServer

- 工作原理

  - Custom Metrics APIServer的实现，其实就是一个Prometheus项目的Adaptor

  - 比如，现在要实现一个根据指定 Pod 收到的 HTTP 请求数量来进行 Auto Scaling 的 Custom Metrics，这个 Metrics 就可以通过访问如下所示的自定义监控 URL 获取到：

    ```text
    https://<apiserver_ip>/apis/custom-metrics.metrics.k8s.io/v1beta1/namespaces/default/pods/sample-metrics-app/http_requests 
    ```

    - 当你访问这个 URL 的时候，Custom Metrics APIServer 就会去 Prometheus 里查询名叫 sample-metrics-app 这个 Pod 的 http_requests 指标的值，然后按照固定的格式返回给访问者
    - 最普遍的做法，就是让 Pod 里的应用本身暴露出一个 /metrics API，然后在这个 API 里返回自己收到的 HTTP 的请求的数量。所以说，接下来 HPA 只需要定时访问前面提到的自定义监控 URL，然后根据这些值计算是否要执行 Scaling 即可

#### 实例讲解

- 1.部署Prometheus项目

  - 使用Prometheus Operator完成

    ```yaml
    $ kubectl apply -f demos/monitoring/prometheus-operator.yaml
    clusterrole "prometheus-operator" created
    serviceaccount "prometheus-operator" created
    clusterrolebinding "prometheus-operator" created
    deployment "prometheus-operator" created
    
    $ kubectl apply -f demos/monitoring/sample-prometheus-instance.yaml
    clusterrole "prometheus" created
    serviceaccount "prometheus" created
    clusterrolebinding "prometheus" created
    prometheus "sample-metrics-prom" created
    service "sample-metrics-prom" created
    ```

- 2.把Custom Metrics APIServer部署起来

  ```yaml
  $ kubectl apply -f demos/monitoring/custom-metrics.yaml
  namespace "custom-metrics" created
  serviceaccount "custom-metrics-apiserver" created
  clusterrolebinding "custom-metrics:system:auth-delegator" created
  rolebinding "custom-metrics-auth-reader" created
  clusterrole "custom-metrics-read" created
  clusterrolebinding "custom-metrics-read" created
  deployment "custom-metrics-apiserver" created
  service "api" created
  apiservice "v1beta1.custom-metrics.metrics.k8s.io" created
  clusterrole "custom-metrics-server-resources" created
  clusterrolebinding "hpa-controller-custom-metrics" created
  ```

- 3.为Custom Methrics APIServer创建对应的ClusterRoleBinding，以能够使用curl来直接访问Custom Metrics的API

  ```yaml
  $ kubectl create clusterrolebinding allowall-cm --clusterrole custom-metrics-server-resources --user system:anonymous
  clusterrolebinding "allowall-cm" created
  ```

- 4.把待监控的应用和 HPA 部署起来

  ```shell
  $ kubectl apply -f demos/monitoring/sample-metrics-app.yaml
  deployment "sample-metrics-app" created
  service "sample-metrics-app" created
  servicemonitor "sample-metrics-app" created
  horizontalpodautoscaler "sample-metrics-app-hpa" created
  ingress "sample-metrics-app" created
  ```

  HPA的配置：

  ```yaml
  kind: HorizontalPodAutoscaler
  apiVersion: autoscaling/v2beta1
  metadata:
    name: sample-metrics-app-hpa
  spec:
    scaleTargetRef:
      apiVersion: apps/v1
      kind: Deployment
      name: sample-metrics-app
    minReplicas: 2
    maxReplicas: 10
    metrics:
    - type: Object
      object:
        target:
          kind: Service
          name: sample-metrics-app
        metricName: http_requests
        targetValue: 100
  ```

  - HPA的配置，就是设置Auto Scaling规则的地方

  - scaleTargetRef字段，指定了被监控的对象是名叫sample-metrics-app的Deployment，也就是我们上面部署的被监控应用，并且它最小实例数目是2，最大实例数目是10

  - metrics字段，指定了HPA进行scale的依据，是名叫http_requests的Metrics，获取这个Metrics的途径，是访问名叫sample-metrics-app的service

  - 有了这些字段的定义，HPA就可以通过以下URL发起请求来获取Custom Metrics的值：

    ```text
    https://<apiserver_ip>/apis/custom-metrics.metrics.k8s.io/v1beta1/namespaces/default/services/sample-metrics-app/http_requests
    ```

- 5.通过名叫hey的测试工具为我们的应用增加一些访问压力：

  ```shell
  $ # Install hey
  $ docker run -it -v /usr/local/bin:/go/bin golang:1.8 go get github.com/rakyll/hey
  
  $ export APP_ENDPOINT=$(kubectl get svc sample-metrics-app -o template --template {{.spec.clusterIP}}); echo ${APP_ENDPOINT}
  $ hey -n 50000 -c 1000 http://${APP_ENDPOINT}
  ```

  访问应用Service的Custom Metrics URL，可以看到返回应用收到的HTTP请求数量了：

  ```shell
  $ curl -sSLk https://<apiserver_ip>/apis/custom-metrics.metrics.k8s.io/v1beta1/namespaces/default/services/sample-metrics-app/http_requests
  {
    "kind": "MetricValueList",
    "apiVersion": "custom-metrics.metrics.k8s.io/v1beta1",
    "metadata": {
      "selfLink": "/apis/custom-metrics.metrics.k8s.io/v1beta1/namespaces/default/services/sample-metrics-app/http_requests"
    },
    "items": [
      {
        "describedObject": {
          "kind": "Service",
          "name": "sample-metrics-app",
          "apiVersion": "/__internal"
        },
        "metricName": "http_requests",
        "timestamp": "2018-11-30T20:56:34Z",
        "value": "501484m"
      }
    ]
  }
  ```

  - 返回的value值其实是通过prometheus收集到的值

  - Prometheus项目是如何知道采集哪些Pod的Metrics API作为监控指标的来源呢？

    - 在创建应用时有一呃ServiceMonitor对象也被创建出来
    - 这个serviceMonitor对象就是Prometheus Operator项目用来监控Pod的一个配置文件

    ```yaml
    apiVersion: monitoring.coreos.com/v1
    kind: ServiceMonitor
    metadata:
      name: sample-metrics-app
      labels:
        service-monitor: sample-metrics-app
    spec:
      selector:
        matchLabels:
          app: sample-metrics-app
      endpoints:
      - port: web
    ```

    - 可以看到通过selector来指定被监控Pod

- 这时候查看Pod数量的话，可以看到HPA开始增加Pod数目了