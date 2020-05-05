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



### 12.第一个容器化应用



## 容器编排与k8s作业管理

### 13.为什么我们需要Pod

- 个人理解：为什么：Pod是一组超亲密关系的容器(频繁相互调用和文件交换等等)，解决了成组调度的问题，同时容器设计模式有很大的用途，比如sidecar，举了两个实例，web服务器和war包以及容器日志收集，原理都是因为pod之间容器共享volume
- 个人理解：Pod实现原理：多个容器共享Infra容器(首先启动)的network namespace，多个容器共享volume(volume设计在pod级别，所有容器分别挂载该volume即可)

#### 13.1成组调度

- Mesos 
  - 资源囤积，所有设置了Affinity约束的任务都到达时，进行统一调度
  - 调度效率低下，可能会出现死锁
- Omega
  - 使用乐观调度，先不管冲突，如果有问题通过回滚解决
  - 比较复杂
- k8s
  - 以Pod为最小调度单位

#### 13.2容器设计模式

- Pod实现原理

  - Pod里所有容器共享同一个Network Namespace,并且可以声明共享同一个Volume

  - 引入中间容器Infra，这个容器首先被创建，创建的容器会共享Infra容器的网络ns

    - Infra容器占用很少的资源，使用的是k8s.gcr.io/pause

  - Volume的共享：将volume定义在pod层级即可，用户容器设置挂载目录即可

    ```yaml
    
    apiVersion: v1
    kind: Pod
    metadata:
      name: two-containers
    spec:
      restartPolicy: Never
      volumes:
      - name: shared-data
        hostPath:      
          path: /data
      containers:
      - name: nginx-container
        image: nginx
        volumeMounts:
        - name: shared-data
          mountPath: /usr/share/nginx/html
      - name: debian-container
        image: debian
        volumeMounts:
        - name: shared-data
          mountPath: /pod-data
        command: ["/bin/sh"]
        args: ["-c", "echo Hello from the debian container > /pod-data/index.html"]
    ```

- 容器设计模式

  - 实例1：web服务器和war包

    - 原始方案

      - 打包在一个容器
        - 缺点：更新war包内容时需要重新发布镜像
      - 只发布web服务器，需声明volume，该volume中含有war包
        - 缺点：需要维护一套分布式存储系统，否则不能在其他机器上适用

    - 新方案

      - 两个容器一个Pod，共享一个volume，war包镜像使用initcontainer先启动，将war包复制到指定目录下
      - initcontainer容器结束后，web服务器容器启动

      ```yaml
      
      apiVersion: v1
      kind: Pod
      metadata:
        name: javaweb-2
      spec:
        initContainers:
        - image: geektime/sample:v2
          name: war
          command: ["cp", "/sample.war", "/app"]
          volumeMounts:
          - mountPath: /app
            name: app-volume
        containers:
        - image: geektime/tomcat:7.0
          name: tomcat
          command: ["sh","-c","/root/apache-tomcat-7.0.42-v2/bin/start.sh"]
          volumeMounts:
          - mountPath: /root/apache-tomcat-7.0.42-v2/webapps
            name: app-volume
          ports:
          - containerPort: 8080
            hostPort: 8001 
        volumes:
        - name: app-volume
          emptyDir: {}
      ```

    - 这种"组合"方式就是常用的一种模式"sidecar"，在一个Pod中，启动一个辅助容器完成一些独立于主容器之外的工作

  - 实例2:容器日志收集

    - 原理：与第一个例子一样，sidecar容器和主容器共享volume
    - 主容器写日志到固定目录，sidecar容器不断从目录里读取日志文件，转发到mongodb活着es中存储起来即可

  - 实例3:Istio项目  微服务治理

    - 原理：共享网络ns
    - 略(下一讲)

### 14. 深入解析Pod对象(一)：基本概念

- 个人理解：讲了Pod的核心字段，凡是调度，网络，存储，安全相关的都会在pod上定义，凡是涉及pod的ns的都是pod上定义，另外，containers也是pod的核心字段。containers的属性除了image，command，workingdir等基础属性，还有imagepullpolicy，lifecycle(poststart,prestop)等
- 个人理解：讲了Pod对象的运行状态pending，running，succed，faild，unknown，另外还简单提到一些更细节的condition：PodScheduled、Ready、Initialized，以及 Unschedulable

#### 14.1Pod重要字段

- 凡是调度，网络，存储，以及安全相关的属性，基本都属于Pod级别，比如Pod运行在哪台服务器上(调度)，配置Pod的网卡（Pod的网络定义），Pod的磁盘（Pod的存储），配置防火墙（Pod的安全定义）等

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

- containers属性

  - 基础属性：image, command,workingdir,ports,volumemounts等

  - 镜像拉取策略：ImageOullPolicy：always，never，ifnotpresent

  - lifecycle：poststart，prestop
  
    - Poststart：在容器启动后，立刻执行一个指定的操作。需要明确的是，postStart 定义的操作，虽然是在 Docker 容器 ENTRYPOINT 执行之后，但它并不严格保证顺序。也就是说，在 postStart 启动时，ENTRYPOINT 有可能还没有结束
      - 如果 postStart 执行超时或者错误，Kubernetes 会在该 Pod 的 Events 中报出该容器启动失败的错误信息，导致 Pod 也处于失败的状态
    - prestop：容器被杀死之前（比如，收到了 SIGKILL 信号）。而需要明确的是，preStop 操作的执行，是同步的。所以，它会阻塞当前的容器杀死流程，直到这个 Hook 定义操作完成之后，才允许容器被杀死
    
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

#### 14.2pod对象的生命周期

- Pod生命周期的变化，主要体现在Pod API对象的status部分，pod.status.phase，就是pod的当前状态
  - Pending：Pod的yaml文件已经提交给了k8s，API对象已经被创建并保存在ETCD当中，但这个Pod里有些容器因为某些原因不能被顺利创建，比如调度不成功
  - Running：Pod已经调度成功，包含的容器都已经创建成功
  - succeeded：Pod里所有容器都正常执行完毕，并且已经退出
  - Failed：Pod 里至少有一个容器以不正常的状态（非 0 的返回码）退出。得想办法 Debug 这个容器的应用，比如查看 Pod 的 Events 和日志
  - Unknown：异常状态，Pod的状态不能持续的被Kubelet汇报给kube-apiserver，这很有可能主从节点之间通信发生了问题
- Conditions：Pod 对象的 Status 字段，还可以再细分出一组 Conditions。这些细分状态的值包括：PodScheduled、Ready、Initialized，以及 Unschedulable。它们主要用于描述造成当前 Status 的具体原因是什么
  - 比如，Pod 当前的 Status 是 Pending，对应的 Condition 是 Unschedulable，这就意味着它的调度出现了问题。
  - 再比如，Ready 意味着 Pod 不仅已经正常启动（Running 状态），而且已经可以对外提供服务了

### 15.深入解析Pod对象（二）：使用进阶

- 个人理解：讲解了secret，configmap，downward API，serviceAccountToken四种projected volume。
- 个人理解：讲解了健康检查livenessProbe和readinessProbe，Pod恢复机制restartPolicy，always，OnFailure，never，以及restartPolicy 和 Pod 里容器的状态，以及 Pod 状态的对应关系
- 个人理解：PodPreset，自动给Pod填充某些字段

#### 15.1Secret

- Secret可以将Pod想要访问的加密数据存放到Etcd中，然后通过在Pod的容器里挂载Volume的方式，访问到这些Secret里保存的信息了

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: test-projected-volume 
  spec:
    containers:
    - name: test-secret-volume
      image: busybox
      args:
      - sleep
      - "86400"
      volumeMounts:
      - name: mysql-cred
        mountPath: "/projected-volume"
        readOnly: true
    volumes:
    - name: mysql-cred
      projected:
        sources:
        - secret:
            name: user
        - secret:
            name: pass
  ```

  - Volume不是常见的emptyDir或者hostPath类型，而是projected类型，数据来源sources是来自user和pass的Secret对象，对应数据库用户名和密码

- secret对象可以由以下两种方式生成

    - kubectl create secret指令

      ```shell
      $ cat ./username.txt
      admin
      $ cat ./password.txt
      c1oudc0w!
      
      $ kubectl create secret generic user --from-file=./username.txt
      $ kubectl create secret generic pass --from-file=./password.txt
      ```

    - kubectl apply -f yaml文件

      ```yaml
      apiVersion: v1
      kind: Secret
      metadata:
        name: mysecret
      type: Opaque
      data:
        user: YWRtaW4=
        pass: MWYyZDFlMmU2N2Rm
      ```

      - 注意：secret对象要求数据必须经过Base64转码

        ```shell
        $ echo -n 'admin' | base64
        YWRtaW4=
        $ echo -n '1f2d1e2e67df' | base64
        MWYyZDFlMmU2N2Rm
        
        
        # 解码
        $ echo -n 'YWRtaW4=' | base64 -d
        admin
        ```

      - 这里内容仅仅是做了转码，而并没有被加密，在生产环境中，需要k8s开启secret的加密插件，增强数据安全性，具体可以见secret相关章节


- 像这样由secret投射到Pod里的volume，是会动态更新的，一旦etcd内容更新，volume里的文件内容也会被更新，由kubelet组件定时维护这些volume
  - 注意：更新可能会有一定的延时。所以在编写应用程序时，在发起数据库连接的代码处写好重试和超时的逻辑，绝对是个好习惯

#### 15.2ConfigMap

- 与secret类似，区别在于ConfigMap存储不需要加密的配置信息

  ```shell
  # .properties文件的内容
  $ cat example/ui.properties
  color.good=purple
  color.bad=yellow
  allow.textmode=true
  how.nice.to.look=fairlyNice
  
  # 从.properties文件创建ConfigMap
  $ kubectl create configmap ui-config --from-file=example/ui.properties
  
  # 查看这个ConfigMap里保存的信息(data)
  $ kubectl get configmaps ui-config -o yaml
  apiVersion: v1
  data:
    ui.properties: |
      color.good=purple
      color.bad=yellow
      allow.textmode=true
      how.nice.to.look=fairlyNice
  kind: ConfigMap
  metadata:
    name: ui-config
    ...
  ```

  

#### 15.3Downward API

- 作用：让Pod里的容器能够直接获取到这个Pod API对象本身的信息

- 例子

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: test-downwardapi-volume
    labels:
      zone: us-est-coast
      cluster: test-cluster1
      rack: rack-22
  spec:
    containers:
      - name: client-container
        image: k8s.gcr.io/busybox
        command: ["sh", "-c"]
        args:
        - while true; do
            if [[ -e /etc/podinfo/labels ]]; then
              echo -en '\n\n'; cat /etc/podinfo/labels; fi;
            sleep 5;
          done;
        volumeMounts:
          - name: podinfo
            mountPath: /etc/podinfo
            readOnly: false
    volumes:
      - name: podinfo
        projected:
          sources:
          - downwardAPI:
              items:
                - path: "labels"
                  fieldRef:
                    fieldPath: metadata.labels
  ```

  - 当前 Pod 的 Labels 字段的值，就会被 Kubernetes 自动挂载成为容器里的 /etc/podinfo/labels 文件

- 支持字段

  ```text
  1. 使用fieldRef可以声明使用:
  spec.nodeName - 宿主机名字
  status.hostIP - 宿主机IP
  metadata.name - Pod的名字
  metadata.namespace - Pod的Namespace
  status.podIP - Pod的IP
  spec.serviceAccountName - Pod的Service Account的名字
  metadata.uid - Pod的UID
  metadata.labels['<KEY>'] - 指定<KEY>的Label值
  metadata.annotations['<KEY>'] - 指定<KEY>的Annotation值
  metadata.labels - Pod的所有Label
  metadata.annotations - Pod的所有Annotation
  
  2. 使用resourceFieldRef可以声明使用:
  容器的CPU limit
  容器的CPU request
  容器的memory limit
  容器的memory request
  ```

  - Downward API 能够获取到的信息，一定是 Pod 里的容器进程启动之前就能够确定下来的信息。而如果你想要获取 Pod 容器运行后才会出现的信息，比如，容器进程的 PID，那就肯定不能使用 Downward API 了，而应该考虑在 Pod 里定义一个 sidecar 容器

#### 15.4ServiceAccountToken（特殊的secret）

- service account
  - 作用：Kubernetes 系统内置的一种“服务账户”，它是 Kubernetes 进行权限分配的对象
  - **ServiceAccountToken**：Service Account 的授权信息和文件，实际上保存在它所绑定的一个特殊的 Secret 对象里的。这个特殊的 Secret 对象，就叫作 ServiceAccountToken
    - 任何运行在 Kubernetes 集群上的应用，都必须使用这个 ServiceAccountToken 里保存的授权信息，也就是 Token，才可以合法地访问 API Server

- ServiceAccountToken

  - 默认账户：为了方便使用，Kubernetes 已经提供了一个默认“服务账户”（default Service Account）。并且，任何一个运行在 Kubernetes 里的 Pod，都可以直接使用这个默认的 Service Account，而无需显示地声明挂载它

    - Projected Volume机制：每个Pod里都会有一个类型是 Secret、名为 default-token-xxxx 的 Volume，然后自动挂载到每个容器固定目录上

      ```shell
      $ kubectl describe pod nginx-deployment-5c678cfb6d-lg9lw
      Containers:
      ...
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-s8rbq (ro)
      Volumes:
        default-token-s8rbq:
        Type:       Secret (a volume populated by a Secret)
        SecretName:  default-token-s8rbq
        Optional:    false
      ```

      - 容器可以在挂载目录里访问到授权信息和文件

        ```shell
        $ ls /var/run/secrets/kubernetes.io/serviceaccount 
        ca.crt namespace  token
        ```

      - 应用程序只要直接加载这些授权文件，就可以访问并操作k8s API了

        - 如果使用的是 Kubernetes 官方的 Client 包（k8s.io/client-go）的话，它还可以自动加载这个目录下的文件，你不需要做任何配置或者编码操作

    - InClusterConfig：这种把 Kubernetes 客户端以容器的方式运行在集群里，然后使用 default Service Account 自动授权的方式，被称作“InClusterConfig”，也是我最推荐的进行 Kubernetes API 编程的授权方式

    - 当然，考虑到自动挂载默认 ServiceAccountToken 的潜在风险，Kubernetes 允许你设置默认不为 Pod 里的容器自动挂载这个 Volume

  - 自定义 Service Account：可对应不同的权限设置。这样，我们的 Pod 里的容器就可以通过挂载这些 Service Account 对应的 ServiceAccountToken，来使用这些自定义的授权信息。在后面讲解为 Kubernetes 开发插件的时候，我们将会实践到这个操作

#### 15.5容器健康检查和恢复机制 <没看明白有疑问>

- 健康检查探针：Pod 里的容器定义一个健康检查“探针”，kubelet 会根据 探针 的返回值决定这个容器的状态，而不是直接以容器进行是否运行（来自 Docker 返回的信息）作为依据。这种机制，是生产环境中保证应用健康存活的重要手段

- 例子

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      test: liveness
    name: test-liveness-exec
  spec:
    containers:
    - name: liveness
      image: busybox
      args:
      - /bin/sh
      - -c
      - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
      livenessProbe:
        exec:
          command:
          - cat
          - /tmp/healthy
        initialDelaySeconds: 5
        periodSeconds: 5
  ```

  - 我们定义了一个这样的 livenessProbe（健康检查）。它的类型是 exec，这意味着，它会在容器启动后，在容器里面执行一条我们指定的命令，比如：“cat /tmp/healthy”。这时，如果这个文件存在，这条命令的返回值就是 0，Pod 就会认为这个容器不仅已经启动，而且是健康的。这个健康检查，在容器启动 5 s 后开始执行（initialDelaySeconds: 5），每 5 s 执行一次（periodSeconds: 5）

- 具体操作过程

  - 创建以上pod

    ```shell
    $ kubectl create -f test-liveness-exec.yaml
    ```

  - 查看pod状态

    ```shell
    $ kubectl get pod
    NAME                READY     STATUS    RESTARTS   AGE
    test-liveness-exec   1/1       Running   0          10s
    ```

  - 30s后，再查看pod的events

    ```shell
    $ kubectl describe pod test-liveness-exec
    
    FirstSeen LastSeen    Count   From            SubobjectPath           Type        Reason      Message
    --------- --------    -----   ----            -------------           --------    ------      -------
    2s        2s      1   {kubelet worker0}   spec.containers{liveness}   Warning     Unhealthy   Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
    ```

    - 显然，这个健康检查探查到 /tmp/healthy 已经不存在了，所以它报告容器是不健康的

  - 再次查看pod状态

    ```shell
    $ kubectl get pod test-liveness-exec
    NAME           READY     STATUS    RESTARTS   AGE
    liveness-exec   1/1       Running   1          1m
    ```

    - pod并没有进入failed状态，而是保持了running状态
    - restarts显示重启1次，这个异常的容器已经被 Kubernetes 重启了。在这个过程中，Pod 保持 Running 状态不变
      - 注意：Kubernetes 中并没有 Docker 的 Stop 语义。所以虽然是 Restart（重启），但实际却是重新创建了容器

- 原因解析

  - Pod恢复机制：也叫 restartPolicy。它是 Pod 的 Spec 部分的一个标准字段（pod.spec.restartPolicy），默认值是 Always，即：任何时候这个容器发生了异常，它一定会被重新创建

    - 注意点
      - Pod 的恢复过程，永远都是发生在当前节点上，而不会跑到别的节点上去。事实上，一旦一个 Pod 与一个节点（Node）绑定，除非这个绑定发生了变化（pod.spec.node 字段被修改），否则它永远都不会离开这个节点。这也就意味着，如果这个宿主机宕机了，这个 Pod 也不会主动迁移到其他节点上去
      - 如果你想让 Pod 出现在其他的可用节点上，就必须使用 Deployment 这样的“控制器”来管理 Pod，哪怕你只需要一个 Pod 副本**(一个单 Pod 的 Deployment 与一个 Pod 最主要的区别**) 《疑问》

  - 用户可自己设置 restartPolicy，改变 Pod 的恢复策略

    - Always：在任何情况下，只要容器不在运行状态，就自动重启容器
    - OnFailure: 只在容器 异常时才自动重启容器
    - Never: 从来不重启容器

  - 可根据应用运行特性，合理设置三种恢复策略

    - 一个 Pod，它只计算 1+1=2，计算完成输出结果后退出，变成 Succeeded 状态。这时，你如果再用 restartPolicy=Always 强制重启这个 Pod 的容器，就没有任何意义了
    - 如果你要关心这个容器退出后的上下文环境，比如容器退出后的日志、文件和目录，就需要将 restartPolicy 设置为 Never。因为一旦容器被自动重新创建，这些内容就有可能丢失掉了（被垃圾回收了）

  -  restartPolicy 和 Pod 里容器的状态，以及 Pod 状态的对应关系

    - 官网链接：https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#example-states

    - 两个基本设计原则

      - 只要 Pod 的 restartPolicy 指定的策略允许重启异常的容器（比如：Always），那么这个 Pod 就会保持 Running 状态，并进行容器重启

      - 对于包含多个容器的 Pod，只有它里面所有的容器都进入异常状态后，Pod 才会进入 Failed 状态。在此之前，Pod 都是 Running 状态。此时，Pod 的 READY 字段会显示正常容器的个数，比如：

        ```shell
        $ kubectl get pod test-liveness-exec
        NAME           READY     STATUS    RESTARTS   AGE
        liveness-exec   0/1       Running   1          1m
        ```

- livenessProbe 也可以定义为发起 HTTP 或者 TCP 请求的方式，定义格式如下：

  ```yaml
  ...
  livenessProbe:
       httpGet:
         path: /healthz
         port: 8080
         httpHeaders:
         - name: X-Custom-Header
           value: Awesome
         initialDelaySeconds: 3
         periodSeconds: 3
  ```

  ```yaml
  
      ...
      livenessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 20
  ```

  - Pod 其实可以暴露一个健康检查 URL（比如 /healthz），或者直接让健康检查去检测应用的监听端口。这两种配置方法，在 Web 服务类的应用中非常常用

- readinessProbe字段：readinessProbe 检查结果的成功与否，决定的这个 Pod 是不是能被通过 Service 的方式访问到，而并不影响 Pod 的生命周期。这部分内容会在service章节重点介绍


#### 15.6PodPreset

- 作用：自动填充Pod字段的对象

- 举例PodPreset

  ```yaml
apiVersion: settings.k8s.io/v1alpha1
  kind: PodPreset
  metadata:
    name: allow-database
  spec:
    selector:
      matchLabels:
        role: frontend
    env:
      - name: DB_PORT
        value: "6379"
    volumeMounts:
      - mountPath: /cache
        name: cache-volume
    volumes:
      - name: cache-volume
        emptyDir: {}
  ```
  
  ```yaml
apiVersion: v1
  kind: Pod
  metadata:
    name: website
    labels:
      app: website
      role: frontend
  spec:
    containers:
      - name: website
        image: nginx
        ports:
          - containerPort: 80
  ```
  
  ```shell
$ kubectl create -f preset.yaml
  $ kubectl create -f pod.yaml
  ```
  
  ```shell
$ kubectl get pod website -o yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: website
    labels:
      app: website
      role: frontend
    annotations:
      podpreset.admission.kubernetes.io/podpreset-allow-database: "resource version"
  spec:
    containers:
      - name: website
        image: nginx
        volumeMounts:
          - mountPath: /cache
            name: cache-volume
        ports:
          - containerPort: 80
        env:
          - name: DB_PORT
            value: "6379"
    volumes:
      - name: cache-volume
        emptyDir: {}
  ```
  
  - 这个 Pod 里多了新添加的 labels、env、volumes 和 volumeMount 的定义，它们的配置跟 PodPreset 的内容一样。此外，这个 Pod 还被自动加上了一个 annotation 表示这个 Pod 对象被 PodPreset 改动过
- PodPreset 里定义的内容，只会在 Pod API 对象被创建之前追加在这个对象本身上，而不会影响任何 Pod 的控制器的定义
    - 比如：我们现在提交的是一个 nginx-deployment，那么这个 Deployment 对象本身是永远不会被 PodPreset 改变的，被修改的只是这个 Deployment 创建出来的所有 Pod。这一点请务必区分清楚
  - 如果你定义了同时作用于一个 Pod 对象的多个 PodPreset，Kubernetes 项目会帮你合并（Merge）这两个 PodPreset 要做的修改

### 16.谈谈“控制器”模型

- 个人理解：控制器就是不断调谐的过程，即期望状态和实际状态保持一致。调谐的最终结果，往往都是对被控制对象的某种写操作，比如增加Pod，删除已有Pod

- 控制器模型的实现
  - Deployment 控制器从 Etcd 中获取到所有携带了“app: nginx”标签的 Pod，然后统计它们的数量，这就是实际状态
  - Deployment 对象的 Replicas 字段的值就是期望状态
  - Deployment 控制器将两个状态做比较，然后根据比较结果，确定是创建 Pod，还是删除已有的 Pod

- ![image-20200423004605945](%E6%9E%81%E5%AE%A2%E6%97%B6%E9%97%B4k8s.assets/image-20200423004605945.png)

### 17.经典PaaS的记忆：作业副本与水平扩展

- 个人理解：Deployment控制ReplicaSet (版本)，ReplicaSet控制Pod（副本数）
- 个人理解：deployment对象可通过字段设置滚动升级时的扩张和收缩副本数，也可设置历史版本(保存的历史ReplicaSet数量)

#### 17.1Deployment的完整实现

- 重要功能：可以实现Pod的水平扩展/收缩和滚动更新

- ReplicaSet

  - Deployment控制器实际操纵的就是ReplicaSet对象，而不是Pod对象
  - Deployment所管理的Pod，它的ownerReference就是ReplicaSet

- Deployment和ReplicaSet关系

  <img src="%E6%9E%81%E5%AE%A2%E6%97%B6%E9%97%B4k8s.assets/image-20200423005741409.png" alt="image-20200423005741409" style="zoom:50%;" />

- deployment通过控制器模式，操作ReplicaSet的个数和属性，实现“水平扩展/收缩”和“滚动更新”这两个编排动作；ReplicaSet通过控制器模式保证Pod的个数永远等于指定的个数

- kubectl get deployment

  ```yaml
  $ kubectl get deployments
  NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  nginx-deployment   3         0         0            0           1s
  ```

  - DESIRED：用户期望的Pod副本个数 (spec.replicas的值)
  - CURRENT：当前处于Running状态的Pod的个数
  - UP-TO-DATE：当前处于最新版本的Pod的个数，所谓最新版本是指Pod的spec部分与Deployment里Pod模版里定义的完全一致
  - AVAILABLE：当前已经可用的Pod的个数，即：既是Running状态，又是最新版本，并且已经处于Ready（健康检查正确）状态的Pod的个数

- kubectl rollout status 实时查看Deployment对象的状态变化

  ```shell
  $ kubectl rollout status deployment/nginx-deployment
  Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
  deployment.apps/nginx-deployment successfully rolled out
  ```

#### 17.2滚动更新过程

（通过kubectl edit deployment 修改容器镜像版本）

  - 查看Deployment的Events

    ```shell
    $ kubectl describe deployment nginx-deployment
    ...
    Events:
      Type    Reason             Age   From                   Message
      ----    ------             ----  ----                   -------
    ...
      Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1764197365 to 1
      Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-3167673210 to 2
      Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1764197365 to 2
      Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-3167673210 to 1
      Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1764197365 to 3
      Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-3167673210 to 0
    ```

    - 首先，当你修改了 Deployment 里的 Pod 定义之后，Deployment Controller 会使用这个修改后的 Pod 模板，创建一个新的 ReplicaSet（hash=1764197365），这个新的 ReplicaSet 的初始 Pod 副本数是：0
    - 然后，在 Age=24 s 的位置，Deployment Controller 开始将这个新的 ReplicaSet 所控制的 Pod 副本数从 0 个变成 1 个，即：“水平扩展”出一个副本
    - 紧接着，在 Age=22 s 的位置，Deployment Controller 又将旧的 ReplicaSet（hash=3167673210）所控制的旧 Pod 副本数减少一个，即：“水平收缩”成两个副本
    - 如此交替进行，新 ReplicaSet 管理的 Pod 副本数，从 0 个变成 1 个，再变成 2 个，最后变成 3 个。而旧的 ReplicaSet 管理的 Pod 副本数则从 3 个变成 2 个，再变成 1 个，最后变成 0 个。这样，就完成了这一组 Pod 的版本升级过程

  - 查看rs最终状态

    ```shell
    $ kubectl get rs
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-1764197365   3         3         3       6s
    nginx-deployment-3167673210   0         0         0       30s
    ```

  - 通过RollingUpdateStrategy字段可以控制滚动更新过程中最多可以创建几个新的Pod，以及最多可以删除几个旧的Pod

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    spec:
    ...
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 1
    ```

    - maxSurge 指定的是除了 DESIRED 数量之外，在一次“滚动”中，Deployment 控制器还可以创建多少个新 Pod；而 maxUnavailable 指的是，在一次“滚动”中，Deployment 控制器可以删除多少个旧 Pod
    - 这两个配置还可以用前面我们介绍的百分比形式来表示，比如：maxUnavailable=50%，指的是我们最多可以一次删除“50%*DESIRED 数量”个 Pod

  - 关系图

    <img src="%E6%9E%81%E5%AE%A2%E6%97%B6%E9%97%B4k8s.assets/image-20200423011828754.png" alt="image-20200423011828754" style="zoom:50%;" />

#### 17.3回滚：滚动升级的方式完成降级

  - 当升级失败时，比如更新版本时写入了错误的镜像，docker hub上没有该对象，则水平收缩会自动停止

  - 回滚到上一版本

    ```shell
    $ kubectl rollout undo deployment/nginx-deployment
    deployment.extensions/nginx-deployment
    ```

  - 回滚到之前的版本

    ```shell
    # 查看滚动更新历史版本
    $ kubectl rollout history deployment/nginx-deployment
    deployments "nginx-deployment"
    REVISION    CHANGE-CAUSE
    1           kubectl create -f nginx-deployment.yaml --record
    2           kubectl edit deployment/nginx-deployment
    3           kubectl set image deployment/nginx-deployment nginx=nginx:1.91
    
    
    # 回到指定版本号
    $ kubectl rollout undo deployment/nginx-deployment --to-revision=2
    deployment.extensions/nginx-deployment
    ```

- 控制历史版本个数：Deployment对象的spec.revisionHistoryLimit字段可以设置保留的“历史版本个数”，即历史ReplicaSet的数量，如果设置为0，则不能进行回滚操作

### 18.深入理解StatefulSet (一) ：拓扑状态

- 个人理解：对于pod实例之间不对等，或者实例对外部数据有依赖关系的应用，可以使用StatefulSet来实现
- 个人理解：StatefulSet通过headless service来给pod提供了固定的访问入口(使用dns记录pod-name.svc-name.namespace.svc.cluster.local进行访问)，创建Pod时从0进行编号一次创建，保证拓扑状态一致。 StatefulSet的创建需要对应一个headless service (clusterIp=none)
- 个人理解：service通过dns可以访问到服务，有两种情况，一种是headless service，直接访问pod的dns记录；另一种是正常访问，通过svc-name.namespace.svc.cluster.local先解析出service的clusterip，然后访问clusterip即可

#### 18.1StatefulSet

- 有状态应用：实例之间有不对等关系，以及实例对外部数据有依赖关系的应用
- 定义：实例之间不对等，实例对外部数据有依赖关系的应用，称为“有状态应用”（Stateful Application）
- StatefulSet把真实世界的应用状态抽象为两种情况：
  - 拓扑状态：Pod实例启动必须按照某系顺序；当被删除后，重新创建时也必须按照这个顺序。并且新创建出来的Pod，必须和Pod的网络标识一样，这样原先的访问者才能使用同样的方法访问到这个新Pod
  - 存储状态：多个Pod实例绑定了不同的存储数据，Pod第一次读取到的数据和隔了十分钟之后再次读取到的数据，应该是同一份，哪怕在此期间Pod A被重新创建过。典型的例子，就是一个数据库应用的多个存储实例
- 核心功能：通过某种方式记录这些状态，然后在Pod被重新创建时能够为新Pod恢复这些状态

#### 18.2工作原理

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
  
  - Pod的创建过程
  
    ```shell
    $ kubectl get pods -w -l app=nginx
    NAME      READY     STATUS    RESTARTS   AGE
    web-0     0/1       Pending   0          0s
    web-0     0/1       Pending   0         0s
    web-0     0/1       ContainerCreating   0         0s
    web-0     1/1       Running   0         19s
    web-1     0/1       Pending   0         0s
    web-1     0/1       Pending   0         0s
    web-1     0/1       ContainerCreating   0         0s
    web-1     1/1       Running   0         20s
    ```
  
    - 这些 Pod 的创建，是严格按照编号顺序进行的。比如，在 web-0 进入到 Running 状态、并且细分状态（Conditions）成为 Ready 之前，web-1 会一直处于 Pending 状态
  
  - 这些pod的hostname与pod名字相同
  
    ```shell
    $ kubectl exec web-0 -- sh -c 'hostname'
    web-0
    $ kubectl exec web-1 -- sh -c 'hostname'
    web-1
    ```
  
  - 当这些Pod被删除后，Kubernetes 会按照原先编号的顺序，创建出了两个新的 Pod。并且，Kubernetes 依然为它们分配了与原来相同的“网络身份”：web-0.nginx 和 web-1.nginx，因此，StatefulSet 就保证了 Pod 网络标识的稳定性
  
  - 只是dns记录不变，pod的ip地址可以变化，访问时采用固定不变的dns记录即可

### 19.深入理解StatefulSet (二) ：存储状态



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

### 44.Kubernetes GPU管理与Device Plugin机制





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