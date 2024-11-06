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
| deployment | pod控制器 |
| controlle |控制器，管理pod|
| service |pod对外入口，维护同类pod|
| label |同类pod相同标签|
| namespace |命名空间，隔离pod运行环境|

### 基础命令
| 命令 | 说明 |
| :--- | :--- |
| kubectl explain kind[.property] | 查看资源属性或子属性 |
| kubectl api-versions | 查看可用版本 |

### 资源属性
#### 一级属性
| 属性 | 说明 |
| :--- | :--- |
| apiVersion | 版本，需要能用kubectl api-versions查询到 |
| kind | 资源类型 |
| metadata | 元数据 |
| spec | 配置详情|

#### metadata
| metadata | 说明 |
| :--- | :--- |
| name | 资源名 |
| namespace | 所属命名空间 |
| labels | 标签 |

#### spec
| spec | 说明 |
| :--- | :--- |
| containers | 容器列表 |
| nodeName | 将pod调度到指定node |
| nodeSelector | 将pod调度到指定标签node |
| hostNetwork | 是否使用主机网络模式，默认false |
| volumes | pod上挂载的存储信息 |
| restartPolicy | 重启策略, Always, ifNotPresent, Never |

##### containers
| spec.containers | 说明 |
| :--- | :--- |
| name | 容器名称 |
| iamge | 容器镜像地址 |
| imagePullPolicy | 拉取策略 |
| command | 启动命令列表 |
| args | 启动参数列表 |
| env | 环境变量 -name/value |
| ports | 端口列表 -name/containerPort/protocol |
| resources | 资源限制、请求设置, limits上限/requests下限，cpu(cpu限制)/memory内存限制 |
command和args用于覆盖Dockerfile，有command则使用command，否则使用Dockerfile

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
| kubectl get ns [name -o xxx] | 查看namespace |
| kubectl describe ns name | 查看namespace详情 |
| kubectl create ns name | 创建ns |
| kubectl delete ns name | 删除ns |
### yaml配置
``` yml
# ns.yaml
apiVersion: v1 # api版本
kind: Namespace # 资源类型
metadata: # 资源对象信息
  name: nsn # 资源名
```
创建 kubectl create -f ns.yaml  
删除 kubectl delete -f ns.yaml

## pod
容器存在于pod中
### 创建pod
kubectl run podName [参数]: kubectl run pdn --image=nginx:latest --port=8080 --namespace nsn 
| 参数 | 说明 | 示例 |
|:---|:---|:---|
| image | pod镜像 | --image=nginx:v3 |
| port | 指定端口 | --port=9090 |
| namespace| 指定ns | --namespace nsName 或 -n nsName |
### 查看pod信息
查看基本信息 kubectl get pods -n nsn  
查看详细信息 kubectl describe pod pdn -n nsn  
### 访问pod
获取pod ip：kubectl get pods -n nsn -o wide  
访问pod：curl ip  
### 删除pod
查看pod控制器：kubectl get deploy -n nsn  
删除pod控制器：kubectl delete deploy -n nsn  
删除pod：kubectl delete pod pdn -n nsn  
### yaml配置
``` yml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pdn
  namespace: nsn # 所属命名空间
spec: # 对象行为和状态
  containers: # 容器
  - image: nginx:latest # 镜像
    name: nginx # 容器名
    ports: # 端口信息
    - name: nginx-port # 端口名
      containerPort: 80 # 端口号
      protocol: TCP # 协议
```
创建：kubectl create -f pod.yaml  
删除：kubectl delete -f pod.yaml  

## label
标签以key-value形式附加到对象上
### 选择器
基于灯饰的选择器：key = val, key != val  
基于集合的选择器：key in (val1, val2), key not in (val1, val2)  
### 增加标签
kubectl label pod pdn key1=val1 -n nsn
### 更新标签
kubectl label pod pdn key1=val2 -n nsn --overwrite
### 查看标签
kubectl get pod pdn key=val -n nsn --show-labels
### 筛选标签
kubectl get pod pdn key=val -n nsn -l key1=val1 --show-labels
### 删除标签
kubectl label pod pdn key1- -n nsn
### yaml配置
``` yml
# label.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pdn
  namespace: nsn
  labels: # 标签
    version: "3.0" # 标签键值
    env: "test"
spec:
  containers:
  - image: nginx:latest
    name: pod
    ports:
    - name: nginx-port
      containerPort: 80
      protocol: TCP
```
更新 kubectl apply -f label.yaml

## deployment--pod控制器
### 创建deploy
kubectl create deployment deployName [参数]: kubectl create deployment dyn --image=nginx:latest --port=8080 --replicas=2 -n nsn
| 参数 | 说明 | 示例 |
|:---|:---|:---|
| image | pod镜像 | --image=nginx:v3 |
| port | 指定端口 | --port=9090 |
| replicas | pod数量 | --replicas=2 |
| namespace| 指定ns | --namespace nsName 或 -n nsName |
### 查看deploy
kubectl get deploy -n nsn  
详细信息 kubectl describe deploy dyn -n nsn  
### 删除deploy
kubectl delete deploy dyn -n nsn
### yaml配置
``` yml
# deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dyn
  namespace: nsn
spec:
  replicas: 3 # pod数
  selector: # 选择器
    matchLabels: # 选择器信息
      run: nginx # 选择器key-val
  template: # pod模板
    metadata:
      labels: # pod标签
        run: nginx
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
```
创建：kubectl create -f deploy.yaml  
删除：kubectl delete -f deploy.yaml  

## service
同类pod的对外接口--可用于服务发现、负载均衡
### 创建service
集群内部可访问 kubectl expose deploy dyn --name=svcn  --type=ClusterIP --port=8080 --target-port=80 -n nsn  
集群外部可访问 kubectl deploy dyn --name=svcn  --type=NodePort --port=8080 --target-port=80 -n nsn  
### 查看service
kubectl get svc svcn -n nsn -o wide
### 删除service
kubectl delete svc svcn -n nsn
### yaml配置
``` yml
# svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: svcn
  namespace: nsn
spec:
  clusterIP: 10.109.179.231 # 固定svc的内网ip
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  type: ClusterIP # service类型，ClusterIP内部NodePort外部
```
创建：kubectl create -f svc.yaml  
删除：kubectl delete -f svc.yaml  

# 竞品
## Swarm

## Mesos
