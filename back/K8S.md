# Kubernetes 分布式
https://www.cnblogs.com/timelesszhuang/p/k8s.html
## 优点
自我修复、弹性伸缩、负载均衡、版本回退、存储编排、服务发现  
## 基础
|概念|说明|
|:---|:---|
| master |集群控制节点，至少一个|
| node |工作节点|
| pod |最小控制单元，其中包含1-多个容器|
| controlle |控制器，管理pod|
| service |pod对外入口，维护同类pod|
| label |同类pod相同标签|
| namespace |命名空间，隔离pod运行环境|
### 部署
需要集群节点时间精确一致
``` shell
# chronyd同步时间
systemctl start chronyd
systemctl enable chronyd

# 配置会见同步服务器
```
#### kubeadm
kubeadm init创建master节点 -> kubeadm join master-ip:port 将node加入集群

#### 二进制包  

## namespace
| 命令 | 说明 |
|:---|:---|
| kubectl get ns | 查看所有namespace |
| node |工作节点|
| pod |最小控制单元，其中包含1-多个容器|
| controlle |控制器，管理pod|
| service |pod对外入口，维护同类pod|
| label |同类pod相同标签|
| namespace |命名空间，隔离pod运行环境|

# 竞品
## Swarm

## Mesos
