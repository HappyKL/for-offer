# 大米运维

## metrics

对一种采样数据的总称

### 类型

Gauges：瞬时，比如磁盘内存使用率

counters：持续增加，比如网站访问人数

histogram:



### 格式

key-value



### 采集方式

exporter

Push gateway



### 循环更新同步时间

```shell
timedatectl set-timezone Asia/Shanghai
*****ntpdate -u cn.pool.ntp.org
```



## 命令行

### 例子：cpu利用率公式

```shell
# 一分钟的增量 node_cpu是一个counter类型
increase(node_cpu{mode="idle"}[1m])
# 所有值加和
sum()
# 按不同机器分开
by(instance)

# cpu使用率
(1 - sum(increase(node_cpu{mode="idle"}[1m])) / sum(node_cpiu[1m])))*100
```



### 过滤

```shell
# 标签过滤 比如node_cpu{mode="idle"}
# 数值过滤 比如node_cpu > 4000000
```



### 六个函数

```shell
rate(counter) # 括号里放counter类型，默认每秒的速率
rate(counter[1m]) # 一分钟的平均速率

# 增量
increase(counter[1m])

# 加和
sum()

# 归类
by(instance) 
by(cluster_name)

# 前三位 
topK(3,Gauges)

# 举例：找出CPU利用率超过百分之80的有几台
count()
```



## 数据采集

- 小知识

  ```shell
  # 后台启动
  & 
  nohup
  screen # 需要先安装 
  daemonize
  ```

### prometheus的安装与使用

#### 运行

```shell
# 下载安装包后，直接./prometheus，后台运行方式见上面小知识
# 生产环境下prometheus的配置参数
--web.read-timeout=5m # 请求连接的最长等待时间
--web.max-connection=512 # 最大链接数
--storage.tsdb.retention=15d # 保存15天的数据
--storage.tsdb.path="data/" # 保存路径
--query.timeout=2m  # 慢查询终止
```

#### 配置文件yaml

```shell
scrap_interval: 15s # 监控频率

targets: [“server1:9100”] # 监控机器名，端口9100是node_exporter的默认端口

# 要记得将机器名的dns解析写入hosts文件或者存入local dns server

# 访问端口默认是node_exporter的
```

### node_exporter的安装与运行

```shell
# 与Prometheus一样，解压安装包后 ./node_exporter

# 可以使用curl localhost:9100/metrics

# prometheus 每15s 会向node_exporter的9100端口发送get请求
```



### pushgateway的安装与运行

```shell
# 被动推送 与Prometheus一样，解压后，./pushgateway

# 写脚本，通过post方式发送到pushgateway服务器

# 简单方便，通常中小型企业，采用node_exporter和db_exporter，其他采用pushgateway即可，pushgateway的短板：多个脚本同时发送到pushgateway服务器可能会造成进程死掉，另外，如果脚本出错，pushgateway仍旧会把错误的数据发送给prometheus.  但经大米老师的经验分析，第一个短板其实基本不会出现，最多就是接收慢一点，不会死掉；第二个短板如果自己仔细点也没啥问题。
```



### 编写exporter

```shell
# 大米有一个例子可以多读读！！！

# 编写一定要小心，避免出现资源泄露，这种后台程序频繁调用，会导致很严重的僵尸进程，会拖垮机器
```

## Grafana

### 安装与使用

```shell
# 安装rpm文件，双击即可安装
service grafana-server start #即可开启服务

# add data source
# new dashboard

# add metrics

# exporter/pushgateway -> prometheus_server -> grafana -> pagerduty(报警) -> 手机/短信/邮件等

# add alert
```

