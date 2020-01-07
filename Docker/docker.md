#  1基础知识

## 1.1 linux namespace

- Namespace的API主要使用以下三个系统调用：
  - clone()：创建新进程，根据系统调用参数来判断哪些类型的Namespace被创建，而且它们的子进程也会被包含到这些Namespace中
  - setns()：将进程加入到Namespace中
  - unshare()：将进程移出某个Namespace

- Linux共实现了六种namespace。

### 1.1.1 Mount Namespace

- 系统调用参数 CLONE_NEWNS
- 隔离各个进程看到的挂载点视图，在不同的Mount Namespace中看到的文件系统层次不一样，在Mount Namespace中调用mount(),unmount()仅仅会影响当前Namespace内的文件系统，而对全局文件系统没影响
- 该namespace是Linux实现的第一个namespace类型

### 1.1.2 UTS Namespace

- 系统调用参数 CLONE_NEWUTS
- 隔离nodename和domainname两个系统标识，在每个UTS Namespace中可以设置自己的hostname

### 1.1.3 IPC Namespace

- 系统调用参数 CLONE_NEWIPC
- 隔离System V IPC和POSIX message queues，每个IPC Namespace中都有自己的System V IPC和POSIX message queue

### 1.1.4 PID Namespace

- 系统调用参数 CLONE_NEWPID
- 隔离进程PID，同一个进程在不同的PID Namespace里拥有不同的PID

### 1.1.5 Network Namespace

- 系统调用参数 CLONE_NEWNET
- 用来隔离网络设备，IP地址端口等网络栈

### 1.1.6 Uer Namespace

- 系统调用参数 CLONE_NEWUSER
- 隔离用户的用户组ID



## 1.2 Linux Cgroups

### 1.2.1 定义

- Linux Cgroups提供了对一组进程及将来子进程的资源限制，控制和统计的能力，这些资源包括CPU,内存，存储，网络等
- Cgroups中的三个组件
  - cgroup 是对进程分组管理的一种机制，一个cgroup包含一组进程，并可以在这个cgroup上增加Linux subsystem的各种参数配置，将一组进程和一组subsystem的系统参数关联起来
  - subsystem是一组资源控制的模块，一般包含如下几项：
    - blkio 设置对块设备输入输出的访问控制
    - cpu设置cgroup中进程的CPU被调度的策略
    - cpuacct可以统计cgroup中进程的CPU占用
    - cpuset在多核机器上设置cgroup中进程可以使用的CPU
    - devices控制cgroup中进程对设备的访问
    - ······
  - hierarchy的功能是把一组cgroup串成一个树状的结构，一个这样的树便是一个hierarchy，通过这种树状结构，Cgroups可以做到继承。
  - 三个组件的关系（略）

- kernel接口

  - 创建并挂载一个hierarchy

    ```shell
    mkdir cgroup-test
    # 挂载
    mount -t cgroup -o none,name=cgroup-test cgroup-test ./cgroup-test
    # 查看
    ls ./cgroup-test
    # cgroup.clone_children  cgroup.procs  cgroup.sane_behavior  notify_on_release  release_agent  tasks
    ```

  - 在创建好的hierarchy上cgroup根节点上扩展出两个cgroup

    ```shell
    mkdir cgroup-1
    mkdir cgroup-2
    
    tree
    
    # 结果显示如下
    .
    ├── cgroup-1
    │   ├── cgroup.clone_children
    │   ├── cgroup.procs
    │   ├── notify_on_release
    │   └── tasks
    ├── cgroup-2
    │   ├── cgroup.clone_children
    │   ├── cgroup.procs
    │   ├── notify_on_release
    │   └── tasks
    ├── cgroup.clone_children
    ├── cgroup.procs
    ├── cgroup.sane_behavior
    ├── notify_on_release
    ├── release_agent
    └── tasks
    
    ```

    

  - 在cgroup中添加和移动进程

    ```shell
    echo $$
    
    ```

    

  - 通过subsystem限制cgroup中进程的资源