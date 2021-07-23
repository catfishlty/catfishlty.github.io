---
title: Kubernetes初探（五）
description: 部署用户手册应用
date: 2021-07-20T13:18:00+0800
lastmod: '2021-07-23T10:35:00+0800'
image: kubernates.png
slug: k8s-basic-5
categories:
    - k8s-basic
    - katacoda
tags:
    - K8S
    - 容器
    - Katacoda
    - YAML
---

> Katacoda在线课：[Deploy Guestbook Web App Example](https://www.katacoda.com/courses/kubernetes/guestbook)
> 
> 本系列教程希望能通过交互式学习网站与传统方式结合，更高效、容易的学习知识。
> 本系列教程将使用 [Katacoda在线学习平台](https://www.katacoda.com) 完成学习。

本场景说明了如何使用 *Kubernetes* 和 *Docker* 启动简单的多层 *Web* 应用。留言簿示例应用程序通过调用* JavaScript API* 将访客的笔记存储在 *Redis* 中。 *Redis* 包含一个 master（用于存储）和一组 *slave* 复制集。

### 核心概念

在此场景中将涵盖以下核心概念。这些是理解 *Kubernetes* 的基础。

- Pods
- Replication Controllers
- Services
- NodePorts

## 启动 Kubernetes

首先，我们需要一个正在运行的 *Kubernetes* 集群。详细信息在 [Launch Kubernetes cluster](https://www.katacoda.com/courses/kubernetes/launch-cluster) 

### 任务

使用初始化程序脚本启动单节点集群。初始化脚本将启动 *API*、*Master*、*Proxy* 和 *DNS Discovery*。 *Web App* 使用 *DNS Discovery* 来查找 *Redis slave* 来存储数据。

```bash
launch.sh
```

### 健康检查

使用 `kubectl cluster-info` 和 `kubectl get nodes`命令来检查部署的集群的节点健康信息。

```bash
controlplane $ kubectl cluster-info
Kubernetes master is running at https://172.17.0.81:6443
KubeDNS is running at https://172.17.0.81:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
controlplane $ kubectl get nodes
NAME           STATUS   ROLES    AGE     VERSION
controlplane   Ready    master   2m15s   v1.14.0
node01         Ready    <none>   114s    v1.14.0
```

如果节点返回 *NotReady*，则它仍在等待。请等待几秒钟后再重新启动。

## Redis Master - Replication Controller

启动应用的第一步是启动*Redis Master*。 *Kubernetes* 服务部署至少有两个部分，分别为 *Replication Controller* 和 *Service*。

*Replication Controller* 定义了运行的实例数量、要使用的 *Docker* 镜像以及服务标识名称。其他选项可用于配置和发现。使用上面的编辑器查看 *YAML* 定义。

如果 *Redis* 出现故障，*Replication Controller* 将在活动节点上重新启动它。

### 创建 *Replication Controller*

**redis-master-controller.yaml**
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  replicas: 1
  selector:
    name: redis-master
  template:
    metadata:
      labels:
        name: redis-master
    spec:
      containers:
      - name: master
        image: redis:3.0.7-alpine
        ports:
        - containerPort: 6379

```

在本示例中，*YAML* 使用官方 *redis* 设置端口 *6379* 运行了一个名为 *redis-master* 的 redis 服务器。

*kubectl create* 命令采用 *YAML* 定义并指示 *master* 启动控制器。

```bash
controlplane $ kubectl create -f redis-master-controller.yaml
replicationcontroller/redis-master created
```

### 查看运行的组件

上面的命令创建了一个 *Replication Controller* 。

```bash
controlplane $ kubectl get rc
NAME           DESIRED   CURRENT   READY   AGE
redis-master   1         1         1       2m9s
```

所有容器都被描述为 *Pod*。 *Pod* 是构成特定应用程序（例如 *Redis*）的容器集合。可以使用 *kubectl* 查看 *Pod* 信息。

```bash
controlplane $ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
redis-master-qkfxw   1/1     Running   0          2m54s
```

## Redis Master - Service

第二步是 *Service*。 *Kubernetes* *Service* 是一种命名负载均衡器，它将流量代理到一个或多个容器。即使容器位于不同的节点上，代理也能工作。

服务代理在集群内通信，很少将端口暴露给外部接口。

当启动服务时，似乎无法使用 *curl* 或 *netcat* 进行连接，除非=将其作为 *Kubernetes* 的一部分启动。推荐的方法是使用 *LoadBalancer* 服务来处理外部通信。

**redis-master-service.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  ports:
    # the port that this service should serve on
  - port: 6379
    targetPort: 6379
  selector:
    name: redis-master
```

### 创建 Service

YAML 定义了 *Service* 的名称*redis-master*，以及应该被代理的端口。

```bash
controlplane $ kubectl create -f redis-master-service.yaml
service/redis-master created
```

### 查看 Service 列表与详情
bash
controlplane $ kubectl get services
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP    10m
redis-master   ClusterIP   10.100.28.180   <none>        6379/TCP   43s

controlplane $ kubectl describe services redis-master
Name:              redis-master
Namespace:         default
Labels:            name=redis-master
Annotations:       <none>
Selector:          name=redis-master
Type:              ClusterIP
IP:                10.100.28.180
Port:              <unset>  6379/TCP
TargetPort:        6379/TCP
Endpoints:         10.32.0.193:6379
Session Affinity:  None
Events:            <none>
```

## Redis Slave Pods

在本示例中，我们将运行 *Redis Slaves*，它会从 *master* 复制数据。有关 *Redis* 复制的更多详细信息，请访问 [http://redis.io/topics/replication](http://redis.io/topics/replication)

如前所述，*Controller* 定义了 *Service* 的运行方式。在这个例子中，我们需要确定 *Service* 是如何发现其他 *Pod*。 *YAML* 将 *GET_HOSTS_FROM* 属性表示为 *DNS*。您可以将其更改为在 *YAML* 中使用环境变量，但这会引入创建顺序依赖关系，因为需要运行服务才能定义环境变量。

**redis-slave-controller.yaml**
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-slave
  labels:
    name: redis-slave
spec:
  replicas: 2
  selector:
    name: redis-slave
  template:
    metadata:
      labels:
        name: redis-slave
    spec:
      containers:
      - name: worker
        image: gcr.io/google_samples/gb-redisslave:v1
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access an environment variable to find the master
          # service's host, comment out the 'value: dns' line above, and
          # uncomment the line below.
          # value: env
        ports:
        - containerPort: 6379
```

### 启动 Redis Slave Controller

在这种情况下，我们将使用镜像 *kubernetes/redis-slave:v2* 启动 *Pod* 的两个实例。它将通过 *DNS* 链接到 *redis-master*。

```bash
controlplane $ kubectl create -f redis-slave-controller.yaml
replicationcontroller/redis-slave created
```

### 列出 *Replication Controller*

```bash
NAME           DESIRED   CURRENT   READY   AGE
redis-master   1         1         1       11m
redis-slave    2         2         2       26s
```

## Redis Slave Service

和以前一样，我们需要启动一个与 *redis-slave* 通信的`Service`来让`Slave`能够接收传入的请求。

因为我们有两个复制的 *Pod*，该服务还将在两个节点之间提供负载平衡。

**redis-slave-service.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    name: redis-slave
spec:
  ports:
    # the port that this service should serve on
  - port: 6379
  selector:
    name: redis-slave
```

### Start Redis Slave Service

```bash
controlplane $ kubectl create -f redis-slave-service.yaml
service/redis-slave created
```

```bash
controlplane $ kubectl get services
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP    2m20s
redis-master   ClusterIP   10.100.144.249   <none>        6379/TCP   46s
redis-slave    ClusterIP   10.105.27.32     <none>        6379/TCP   16s
```

## 前端 Replicated Pods

启动数据服务后，我们现在可以部署 *Web* 应用。部署 *Web* 应用的模式与我们之前部署的 *Pod* 相同。

### 启动前端

*YAML* 定义了一个名为 *frontend* 的服务，该服务使用 *gcr.io/google_samples/gb-frontend:v3* 的镜像。*Replication Controller* 将确保三个 *Pod* 始终存在。

**frontend-controller.yaml**
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: frontend
  labels:
    name: frontend
spec:
  replicas: 3
  selector:
    name: frontend
  template:
    metadata:
      labels:
        name: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access environment variables to find service host
          # info, comment out the 'value: dns' line above, and uncomment the
          # line below.
          # value: env
        ports:
        - containerPort: 80
```

```bash
controlplane $ kubectl create -f frontend-controller.yaml
replicationcontroller/frontend created
```

### 列出 Controllers 和 Pods

```bash
controlplane $ kubectl get rc
NAME           DESIRED   CURRENT   READY   AGE
frontend       3         3         3       34s
redis-master   1         1         1       8m20s
redis-slave    2         2         2       8m6s
```

```bash
controlplane $ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
frontend-88lvf       1/1     Running   0          42s
frontend-phhhv       1/1     Running   0          42s
frontend-rq8tt       1/1     Running   0          42s
redis-master-7h9t2   1/1     Running   0          8m28s
redis-slave-gnhfh    1/1     Running   0          8m14s
redis-slave-mtn2r    1/1     Running   0          8m14s
```

### PHP 代码

*PHP* 代码使用 *HTTP* 和 *JSON* 与 *Redis* 通信。当设置一个值时，请求转到 *redis-master*，而读取的数据来自 *redis-slave* 节点。

## 用户手册前端 Service

为了能够访问前端，我们需要启动一个 *Service* 来配置代理。

### 启动代理

*YAML* 将服务定义为 *NodePort* 。 *NodePort* 允许您设置在整个集群中共享的特点端口，这就像 Docker 中的 *-p 80:80*。

在这种情况下，我们定义我们的 *Web* 应用在端口 *80* 上运行，在 *30080* 上开放服务。

**frontend-service.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    name: frontend
spec:
  # if your cluster supports it, uncomment the following to automatically create
  # an external load-balanced IP for the frontend service.
  # type: LoadBalancer
  type: NodePort
  ports:
    # the port that this service should serve on
    - port: 80
      nodePort: 30080
  selector:
    name: frontend
```

```bash
controlplane $ kubectl create -f frontend-service.yaml
service/frontend created
```

```bash
controlplane $ kubectl get services
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
frontend       NodePort    10.98.211.171    <none>        80:30080/TCP   3s
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP        13m
redis-master   ClusterIP   10.100.144.249   <none>        6379/TCP       11m
redis-slave    ClusterIP   10.105.27.32     <none>        6379/TCP       11m
```

我们将在未来的场景中讨论 *NodePort* 。

## 访问用户手册前端

定义了所有 *Controller* 和 *Service* 后，*Kubernetes* 将开始将它们作为 *Pod* 启动。根据实际发生的情况，*Pod* 可以具有不同的状态。例如，如果 *Docker* 映像仍在下载无法启动，则 *Pod* 将处于 *pending* 状态。准备就绪后，状态将会变更为 *running*。

### 查看 Pods 状态

可以使用下列命令查看所有 *Pod* 的状态。

```bash
controlplane $ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
frontend-88lvf       1/1     Running   0          9m3s
frontend-phhhv       1/1     Running   0          9m3s
frontend-rq8tt       1/1     Running   0          9m3s
redis-master-7h9t2   1/1     Running   0          16m
redis-slave-gnhfh    1/1     Running   0          16m
redis-slave-mtn2r    1/1     Running   0          16m
```

### 查找 NodePort

如果您没有分配 *众所周知*（well-known） 的 *NodePort*，那么 *Kubernetes* 将随机分配一个可用端口。您可以使用 *kubectl* 找到分配的 *NodePort*。

```bash
controlplane $ kubectl describe service frontend | grep NodePort
Type:                     NodePort
NodePort:                 <unset>  30080/TCP
```

### 查看 UI

一旦 *Pod* 处于 *running* 状态，您将能够通过端口 *30080* 查看 *UI*。使用 *URL* 查看页面 [https://2886795308-30080-kitek05.environments.katacoda.com](https://2886795308-30080-kitek05.environments.katacoda.com/)

*在背后*（Under the covers） *PHP* 服务通过 *DNS* 发现 *Redis* 实例。您现在已经在 *Kubernetes* 上部署了一个有效的多层应用程序。

