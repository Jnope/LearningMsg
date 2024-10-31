# 运行web程序
docker pull training/webapp
docker run -d -P training/webapp cmd
-d: 后台运行
-P：容器内端口随机映射至主机上
-p source:target：指定映射端口
ps：查看正在运行的容器
port id：查看指定id或名的容器

## 日志
docker logs -f id/name
