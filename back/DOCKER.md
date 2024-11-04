# 运行web程序
docker pull training/webapp  
docker run -d -P training/webapp cmd  
| 命令 | 说明 |
| :--- | :--- |
| run -d | 后台运行 |
| run -P | 容器内端口随机映射至主机上 |
| run -p port:dockerPort | 将dockerPort绑定至主机port |
| run -p ip:port:dockerPort | 将dockerPort(默认tcp, 将dockerPort/tcp)绑定至ip:port |
| docker ps | 查看正在运行的容器 |
| docker port name/id [dockerPort] | 查看指定id或名的容器的端口信息 |

## 日志
docker logs -f id/name

## 容器互联
### 创建
创建Docker网络 ```docker network create -d bridge bridge_name```  
-d：指定网络类型bridge, overlay
### 连接
``` shell
docker run -itd --name test1 --network bridge_name ubuntu /bin/bash
docker run -itd --name test2 --network bridge_name ubuntu /bin/bash
```
连接后各个容器内可以访问另一个容器: 在test2中 ping test1

# Dockerfile
FROM: 基础镜像--如ngnix  
RUN：执行命令--RUN xxx或RUN ["文件", "参数1", "参数2"]两种格式；每次RUN会新建层  
\ 换行
&& 命令连接符，不会导致新建层  
| Dockerfile 指令	| 说明 | 示例 |
| :--- | :--- | :--- |
| FROM |	指定基础镜像，用于后续的指令构建。 | FROM nginx:v3 |
| MAINTAINER |	指定Dockerfile的作者/维护者。（已弃用，推荐使用LABEL指令） |  |
| LABEL |	添加镜像的元数据，使用键值对的形式。 |  |
| RUN |	在构建过程中在镜像中执行命令。 | RUN yum -y install wget （docker build时执行） |
| CMD |	指定容器创建时的默认命令。（可以被覆盖） | CMD ["<可执行文件或命令>","<param1>","<param2>",...]  docker run时执行|
| ENTRYPOINT |	设置容器创建时的主要命令 |  |
| EXPOSE |	声明容器运行时监听的特定网络端口。 | |
| ENV |	在容器内部设置环境变量。 | |
| ADD |	将文件、目录或远程URL复制到镜像中。 | |
| COPY |	将文件或目录复制到镜像中。 | COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"] |
| VOLUME |	为容器创建挂载点或声明卷。 | |
| WORKDIR |	设置后续指令的工作目录。 | |
| USER |	指定后续指令的用户上下文。 | |
| ARG |	定义在构建过程中传递给构建器的变量，可使用 "docker build" 命令设置。 | |
| ONBUILD |	当该镜像被用作另一个构建过程的基础时，添加触发器。 | |
| STOPSIGNAL |	设置发送给容器以退出的系统调用信号。 | |
| HEALTHCHECK |	定义周期性检查容器健康状态的命令。 | |
| SHELL | 覆盖Docker中默认的shell，用于RUN、CMD和ENTRYPOINT指令。 | |

# Compose
## docker-compose.yml
``` yml
version: '3'
services:
  web:
    build: .
    depends_on:
      - redis
    ports:
      - "5001:5000"
  redis:
    image: "redis:alpha"
```
web: web服务从当前目录Dockerfile构建镜像，将容器5000端口绑定至主机5001  
redis: redis使用公共redis镜像  
depends_on: 依赖  
## 启动
docker-compose up -d

# Machine
docker-machine ls 查看虚拟机  
docker-machine create --driver virtualbox name 创建机器 --driver指定驱动类型  
docker-machine cmd name 对name机器操作xxx命令  
|cmd|说明|
|:---|:---|
|ip|查看ip|
|stop|停止机器|
|start|启动机器|
|ssh|进入机器|

# 集群管理Swarm
