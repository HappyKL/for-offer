# 安全防御

b站视频 尚硅谷 linux运维

## 攻击常用手段

拒绝服务：DOS，DDOS

已知漏洞

口令破解

欺骗用户

## 常见防御

**基础类防火墙**：书写规则，比如开放80/443端口

IDS类，入侵检测系统：就像监控，事发时不会阻止，事发后可以查看分析

IPS类，入侵防御系统：根据消息内容进行相应阻止

主动安全类：比IPS更专，比如（waf）Web application firewall

## 基础类防火墙

### 定义

工作在主机边缘处或者网络边缘处对数据报文进行检测，并且能够根据事先定义好的规则，对数据报文进行相应处理的模块

### 分类

- 构造

  硬件：深信服，华为

  软件：iptables（内核态：netfilter，应用层：iptables）

- 工作机制

  包过滤：SIP ， DIP，SPORT ，DPORT

  应用层：URL ，HOSTNAME

## iptables原理

### 四表五链

五链：input output prerouting postrouting forward

四表：

raw表：确定是否对该数据包进行状态跟踪

mangle表：为数据包设置标记

nat表：修改数据包中的源，目标ip地址或端口

filter表：queuing是否放行该数据包

![image-20200221215307574](Linux%E5%AE%89%E5%85%A8%E9%98%B2%E5%BE%A1.assets/image-20200221215307574.png)

### 防火墙顺序



![image-20200221215912792](Linux%E5%AE%89%E5%85%A8%E9%98%B2%E5%BE%A1.assets/image-20200221215912792.png)



![image-20200221220039552](Linux%E5%AE%89%E5%85%A8%E9%98%B2%E5%BE%A1.assets/image-20200221220039552.png)

### 语法规则

iptables [-t 表名] 选项 [链名] [条件] [-j 控制类型]

- 注意事项

  不指定表名时，默认为filter表

  不指定链名时，默认指表内的所有链

  除非设置链的默认策略，否则必须指定匹配条件

  **选项，链名，控制类型使用大写字母，其余均为小写**

- 控制类型

  ACCEPT（接收）

  DROP（丢弃）

  REJECT（拒绝，必要时给出提示）

  LOG（记录日志信息，然后传给下一条规则继续匹配）

  SNAT（修改数据包源地址）

  DNAT（修改数据包目的地址）

  REDIRECT（重定向）

- 选项

  -A 链的末尾追加一条规则

  -I 链的开头插入一条规则

  -L 列出所有的规则条目

  -n 以数字形式显示地址，端口信息

  -v 以更详细的方式显示规则信息（可以通过-v选项查看过滤的包数量,字节数）

  --line-numbers 查看规则时显示序号

  -D 删除链内指定序号（或内容）的一条规则

  -F 清空当前表的规则

  -P 为指定的链设置默认规则

- 举例

  iptables [-t 表名] 选项 [链名] [条件] [-j 控制类型]

  ```shell
  iptables -t filter -A INPUT -p tcp -j ACCEPT
  
  iptabls -A INPUT 2 -p icmp -j ACCEPT
  
  iptables -nL --line-numbers
  
  iptables -D INPUT 4 
  
  iptables -F
  
  iptables -A INPUT -p tcp --dport 22 -j ACCEPT # 开启ssh端口
  iptables -I INPUT -p tcp --dport 80 -j ACCEPT # 端口开启http，http端口访问比ssh多，所以使用-I，插到前面
  iptbles -P INPUT DROP # 默认策略只能是ACCEPT和DROP
  
  ```

- 小知识

  将匹配最多的规则放到最前面（可以通过-v选项查看过滤的包数量）

- 规则匹配

  - 通用匹配

    协议匹配：-p 协议名

    地址匹配：-s 源地址  -d 目的地址

    接口匹配：-i 入站网卡  -o 出站网卡

  - 隐含匹配

    端口匹配：--sport 源端口 --dport目的端口

    ICMP类型匹配：  --icmp-type ICMP类型 

  - 显式匹配

    -m multiport --dport 25,80,110

    -m mac --mac-source 00:0c:29....

    -m iprance -src-range 192.168.4.21-192.168.4.28

    -m state --state NEW,ESTABLISHED,RELATED(连接状态)

- NAT表

  SNAT（路由器其实就是配的这个规则）

  ```shell
  iptabels -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j SNAT --to-source 218.29.30.31
  
  # 下面更常用，上面的规则需要自己指定地址
  iptabels -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE
  ```

  DNAT (典型应用场景：企业局域网内的服务需要公网让大家访问)

  ```shell
  iptables -t nat -A PREROUTING -d 218.29.30.31 -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.6
  
  
  ```

  

### 脚本

- 备份

  ```shell
  service iptables save  # 持久作用
  
  iptables-save > 1.iptables # 备份
  ```

- 还原

  ```shell
  iptables-restore < 1.iptables
  ```

  

- centos7的默认防火墙为firewalld，如何将firewalld修改为iptable

  ```shell
  systemctl stop firewalld
  systemctl disable firewalld
  rpm -e --nodeps firewalld
  yum -y install iptables-services
  systemctl start iptables
  systemctl enable iptables
  ```

  

## SeLinux

### 前世今生

- linux护城河

  iptables

  tcp wrappers

  acl  access control list

  selinux

- 已成为内核的一部分

### 安全上下文

可以说selinux是基于令牌的访问控制

- 组成

  用户:角色:类型，在传统的selinux或类型匹配中，只要类型一致就可以访问

- 操作

```shell
# 将selinux开启
vim /etc/selinux/config
# 修改SELINUX=enforcing
reboot # 将selinux加入到内核需要重启

# 临时设置
setenforce 0 # 关闭
setenforce 1 # 开启
getenforce # 查看
```

