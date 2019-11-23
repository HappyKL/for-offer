

## 作用

容器的自动化复制和部署，随时扩展和收缩容器规模，提供负载均衡



方便容器升级



提供容器弹性，失效就替换



## K8s重要的资源对象

- Master： API Server，Controller Manager，Scheduler

- Node： kubelet，kube-proxy，docker

  通过以下命令可以查看node信息

  ```shell
  kubectl describe node node-name
  ```

  

-  Pod

node是用于部署应用容器的服务器

Pod是基本的操作单元，也是应用运行的载体。整个Kubernetes系统都是围绕着Pod展开的，比如如何部署运行Pod，如何保证Pod的数量，如何访问Pod等

Deployment定义了Pod部署的信息

若干个Pod组成一个Service，对外提供服务



