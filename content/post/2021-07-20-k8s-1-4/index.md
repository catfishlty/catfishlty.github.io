---
title: Kubernetes初探（四）
description: 使用 Yaml 启动容器
date: 2021-07-20T11:39:00+0800
lastmod: '2021-07-21T17:52:00+0800'
image: kubernates.png
slug: k8s-basic-4
categories:
    - k8s-basic
    - katacoda
tags:
    - K8S
    - 容器
    - Katacoda
    - YAML
---

> Katacoda在线课：[Deploy Containers Using YAML](https://www.katacoda.com/courses/kubernetes/creating-kubernetes-yaml-definitions)
> 
> 本系列教程希望能通过交互式学习网站与传统方式结合，更高效、容易的学习知识。
> 本系列教程将使用 [Katacoda在线学习平台](https://www.katacoda.com) 完成学习。

在此场景中，您将学习如何使用 *Kubectl* 创建和启动 *Deployment*、*Replication Controller*，并通过编写 *yaml* 定义使用服务开暴露它们。


*YAML* 定义了计划部署的 *Kubernetes* 对象。可以以更新对象并将其重新部署到集群的方式来更改配置。

## 创建 Deployment

*Deployment* 对象是最常见的 *Kubernetes* 对象之一。*Deployment* 对象定义了所需的容器规范，以及 *Kubernetes* 其他部分用来发现和连接到应用程序的名称和标签。

### 任务

将以下定义复制到编辑器的*YAML*文件中。该 *YAML* 定义了如何使用在端口 *80* 上运行的应用，该应用使用 *Docker* 映像 *katacoda/docker-http-server* ，启动名为 *webapp1* 。

**deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp1
  template:
    metadata:
      labels:
        app: webapp1
    spec:
      containers:
      - name: webapp1
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
```

使用 `kubectl create -f deployment.yaml` 命令向集群部署。

```bash
$ kubectl create -f deployment.yaml
deployment.apps/webapp1 created
```

由于它是一个 *Deployment* 对象，因此可以通过 `kubectl get deployment` 获取所有已部署的 *Deployment* 对象的列表。

```bash
$ kubectl get deployment
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
webapp1   1/1     1            1           10s
```

可以使用 `kubectl describe deployment webapp1` 输出单个部署的详细信息。

```bash
$ kubectl describe deployment webapp1
Name:                   webapp1
Namespace:              default
CreationTimestamp:      Wed, 21 Jul 2021 09:31:47 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=webapp1
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=webapp1
  Containers:
   webapp1:
    Image:        katacoda/docker-http-server:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   webapp1-6b54fb89d9 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  49s   deployment-controller  Scaled up replica set webapp1-6b54fb89d9 to 1
```

## 创建Service

*Kubernetes* 具有强大的网络功能，可以控制应用程序的通信方式。这些网络配置也可以通过 *YAML* 定义。

### 任务

将 *Service* 定义复制到编辑器。该 *Service* 选择带有标签 *webapp1* 的所有应用程序。当部署多个副本或实例时，它们将根据此公共标签自动进行负载平衡。该 *Service* 通过 *NodePort* 的网络连接方式部署。

**service.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp1-svc
  labels:
    app: webapp1
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
  selector:
    app: webapp1
```

所有 *Kubernetes* 对象都一致使用 *kubectl* 完成部署。

使用 `kubectl create -f service.yaml` 命令部署 *Service*

```bash
$ kubectl create -f service.yaml
service/webapp1-svc created
```

和以往一样，使用 `kubectl get svc` 可以查看所有已部署的 *Service* 对象的信息。通过 `kubectl describe svc webapp1-svc` 命令可以查看该 *Service* 对象更多的配置信息。

```bash
$ kubectl get svc
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP        6m38s
webapp1-svc   NodePort    10.111.56.111   <none>        80:30080/TCP   20s
```

```bash
$ kubectl describe svc webapp1-svc
Name:                     webapp1-svc
Namespace:                default
Labels:                   app=webapp1
Annotations:              <none>
Selector:                 app=webapp1
Type:                     NodePort
IP:                       10.111.56.111
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30080/TCP
Endpoints:                172.18.0.4:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

```
$ curl host01:30080
<h1>This request was processed by host: webapp1-6b54fb89d9-5rzpv</h1>
```

## 缩放 Deployment

由于 *Deployment* 需要不同的配置，因此可以更改 *YAML* 的详细信息。这遵循基础设施即代码的思维方式。清单应在源代码控制下，并且保证生产环境的配置和源代码控制中的配置一致。

### 任务

更新 *deployment.yaml* 文件以增加运行的实例数。例如，该文件应如下所示：

```yaml
replicas: 1  
# -->  
replicas: 4
```

使用 *kubectl apply* 提交对现有定义的更新。要扩展副本数量，请使用 `kubectl apply -f deployment.yaml` 部署更新的 *YAML* 文件

```bash
$ kubectl apply -f deployment.yaml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/webapp1 configured
```

集群的状态将立即更新，可通过 `kubectl get deployment` 命令 查看

```bash
$ kubectl get deployment
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
webapp1   4/4     4            4           15m
```

根据定义将有额外的 `Pods` 被调度到集群中， 使用 `kubectl get pods` 查看 *Pod* 信息

```bash
$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
webapp1-6b54fb89d9-5rzpv   1/1     Running   0          16m
webapp1-6b54fb89d9-ktvz4   1/1     Running   0          2m15s
webapp1-6b54fb89d9-kzdjw   1/1     Running   0          2m15s
webapp1-6b54fb89d9-x8vtv   1/1     Running   0          2m15s
```

由于所有 `Pod` 具有相同的标签选择器，因此将由已部署的 *Service* *NodePort* 为这些 *Pod* 提供负载平衡能力。

向目标端口发出多个请求，可以通过返回信息发现，请求是由多个容器进行处理的。

```bash
$ curl host01:30080
<h1>This request was processed by host: webapp1-6b54fb89d9-5rzpv</h1>
$ curl host01:30080
<h1>This request was processed by host: webapp1-6b54fb89d9-kzdjw</h1>
$ curl host01:30080
<h1>This request was processed by host: webapp1-6b54fb89d9-kzdjw</h1>
$ curl host01:30080
<h1>This request was processed by host: webapp1-6b54fb89d9-x8vtv</h1>
$ curl host01:30080
<h1>This request was processed by host: webapp1-6b54fb89d9-ktvz4</h1>
```

其他 *Kubernetes* 网络细节和对象定义将在未来的其他场景中介绍。