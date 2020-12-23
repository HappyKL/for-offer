## helm

chart：一个helm程序包

repository：charts仓库，其实就是一个https/http服务器

release：特定的chart部署于目标集群中的一个实例

 

chart	-> Config(value.yaml) -> release



程序框架：

​	helm ： 客户端，管理本地的chart仓库，管理chart，与tiller服务器交互，发送chart，实例安装查询卸载等

​    tiller：服务器，接收helm发送来的charts，与Config合并生成release



命令

```shell
# release管理
helm install --name xxx stable/memcached # 安装
helm delete xxx # 删除
helm upgrade # 更新release
helm rollback # 回滚
helm list # 列出已有的release
helm history 

# chart管理
helm inspect stable/memcached # 查看chart详细信息
helm init # 安装服务器tiller
helm search  # 搜寻相关chart


## install失败报错及解决
Error: a release named es-exporter already exists.
Run: helm ls --all es-exporter; to check the status of the release
Or run: helm del --purge es-exporter; to delete it

```



chart文件结构

```shell
$ helm create xxx # 创建chart基本文件结构
$ tree xxx/
xxx/
	chart.yaml # chart名字，版本，维护者等信息
	template/ # 模版信息
		deployment.yaml 
		helpers.tpl
		ingress.yaml
		service.yaml
		NOTES.txt # 提示给用户的使用信息
	charts/  # 其他依赖chart
	values.yaml # 自定义配置信息
	
$ helm package xxx/ # 打包
$ helm serve # 开启本地chart仓库服务器

```

### helm搭建efk



