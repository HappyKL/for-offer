# docker

## 安装

```shell
apt-get install docker.io
```

## Docker Hub账号

 向dockerHub网站注册一个账号，可以建立自己的docker仓库，我的账户名dddddddddocker

## 加速器配置

```shell
# 配置镜像加速器(阿里云加速器配置),登陆阿里云，查看容器镜像服务
# 针对Docker客户端版本大于 1.10.0 的用户

# 您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ubn8mq0w.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 基本操作

```shell
# 版本查看
docker version

# 查看镜像
docker images

# 拉取镜像
docker pull mysql:5.6
# 镜像查询可到docker hub中

# 删除镜像
docker rmi nginx

# 容器运行，端口映射，挂存储卷，设置环境变量（可到docker hub上查看有哪些环境变量），
docker run -p 3306:3306 --name mymysql -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6

# 查看当前运行的容器
docker ps
docker ps -a
# 进入容器
docker exec -it container_id /bin/bash

# 查看日志
docker logs -f container_id
# 停止容器
docker stop container_id
# 删除容器
docker rm container_id

# 仓库登陆
docker login

# 镜像构建
docker build -t <your_username>/repo-name:1.0 . # 在当前文件夹下寻找Dockerfile文件
docker build -t aaaa:1.0 -f dockerfilePath 

# 镜像打tag
docker tag mysql:5.6 myregistry/mymysql:1.0

# 发布镜像
docker push myregistry/mymysql:1.0

# 镜像打包
docker image save nginx:latest >nginx.tar
```



## Dockerfile 

```shell
FROM ubutnu:16.04

RUN

# 拷贝文件
ADD

WORKDIR

CMD

EXPOSE


```

参考网址： https://github.com/solochen84/ph 



# docker-compose 多容器APP

```shell
# docker-compose.yaml

build #本地创建对象
command  #覆盖缺省命令
depends on  #连接容器
image  #pull镜像
ports  # 暴露端口
volumes # 卷

up  # 启动服务
stop # 停止服务
rm # 删除服务中的各个容器
logs # 观察各个容器的日志
ps # 列出服务相关的容器

```



