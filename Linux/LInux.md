## 文件权限

文件类型

```shell
- # 普通文件
d # 目录
b # 块设备
c # 字符设备
l # 符号链接
f # 命名管道
s # 套接字
```

权限

```shell

r # 读 4
w # 写 2
x # 执行 1


chmod 755 fileName # 更改权限  
chmod a=rwx fileName # 更改权限 u g o a + - =

chown user1 fileName # 更改属主
chown :group1 fileName # 属组

chgrp group1 fileName # 更改属组

```

# 系统操作篇

## 17 vim正常模式

```shell
yy # 复制一行
3yy # 复制3行

dd # 剪切一行
5dd # 剪切5行
p # 粘贴

u # 撤销
ctrl + r # 重做

```



# 系统管理篇

## 26 网络管理

```shell
# 两套工具 net tools && iproute
# net tools
ifconfig
route
netstat
# iproute
ip
ss

# 修改网络接口名称为eth0
vim /etc/default/grub # 设置GRUB_CMDLINE_LINUX中添加参数：biosdevname=0 net.ifname=0
grub2-mkconfig -o /boot/grub2/grub.cfg # 让修改的grub文件生效
reboot # 重启即可修改成功
```

## 27 查看网络配置

```shell
ifconfig eht0 # 查看指定网卡信息
mii-tool eth0 # 查看网卡的物理状态
route -n # 查看路由 -n 通过ip显示，不用显示域名
```



## 28 修改网络配置

```shell
# 修改网卡
ifconfig eth0 10.21.5.222 netmask 255.255.255.0
ifup eth0
ifdown eth0

# 修改路由
route add default gw <网关IP>
route add -host <特定ip> gw <网关IP>
route add -net <指定网段> netmask <子网掩码> gw <网关IP>

# 通过ip命令也可以实现以上命令
```

## 29 网络故障排除

```shell
ping www.baidu.com
traceroute www.baidu.com
mtr
nslookup www.baidu.com
telnet www.baidu.com 80
tcpdump -i and -n host 10.0.0.1
netstat -tnlp # -t tcp -n 显示ip -l listen -p pid
ss -tnlp
```

## 30 网络管理和网络配置文件

```shell
# 有两种网络管理方式：network和NetworkManager，network是以前的方式，下面以这种方式执行命令

service network status # 查看网络管理状态

# /etc/sysconfig/newwork-scripts 网卡文件 可配置静态网路
service network restart

# 修改主机名
hostname name # 临时修改
hostnamectl set-hostname name # 可以永久修改，需要配置hosts文件，否则下一次开机会有些服务超时（有些服务依赖主机名）
vim /etc/hosts # 添加 127.0.0.1 name
```



## 32.使用rpm命令安装软件包

```shell
rpm 
-q vim-common # 查询已安装的软件 -qa 所有的已安装软件 -q vim-common 查询是否安装vim-common
-i vim-common.xxx.xxx.xxx.rpm # 安装
-e vim-common # 卸载

# rpm文件格式：文件名.版本号.系统版本.平台.rpm

# rpm需要自行解决软件依赖关系
```



### 33.使用yum包管理安装软件包

```shell
# yum可以自行解决软件依赖关系

# 配置yum源为国内的，下载速度快

# 备份原有的源
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup. 
# 将国内的源下载至本机路径
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 更新缓存
yum makecache 

# yum命令
yum 
install # 安装
remove # 移除
list # 查找 list可以列出所有安装的包 list name 查看是否安装某个包
update # 更新 update可以更新所有安装的包 update name 更新某个包


```



## 34.通过源代码编译安装包

```shell
# 其他方式安装：二进制安装  源代码编译安装

wget https://xxxxxxx.tar.gz
tar -zxf xxxx.tar.gz
cd xxxxx/
./configure --prefix=/usr/local/xxxxx # 检查编译环境  --prefix 指定安装位置
make -j2 # 使用2个核编译
make install # 安装

```

