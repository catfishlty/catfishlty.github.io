---
title: Kubernetes初探（三）
description: 使用 Kubectl 启动容器
date: 2021-07-20T11:27:00+0800
lastmod: '2021-07-21T16:29:00+0800'
image: kubernates.png
slug: k8s-basic-3
categories:
    - k8s-basic
    - katacoda
tags:
    - K8S
    - 容器
    - Katacoda
---

> Katacoda在线课：[Kubectl Run Containers](https://www.katacoda.com/courses/kubernetes/kubectl-run-containers)
> 
> 本系列教程希望能通过交互式学习网站与传统方式结合，更高效、容易的学习知识。
> 本系列教程将使用 [Katacoda在线学习平台](https://www.katacoda.com) 完成学习。

在此场景中，将学习如何使用 `Kubectl` 创建和启动`Deployments`, `Replication Controllers`并通过`Services`对外开放接口，而无需编写 *yaml* 定义。这样便可以快速将容器启动到集群上。

## 启动集群

首先，我们需要启动一个 Kubernetes 集群。

执行以下命令启动集群并下载`Kubectl CLI`。

```bash
$ minikube start --wait=false
* minikube v1.8.1 on Ubuntu 18.04
* Using the none driver based on user configuration
* Running on localhost (CPUs=2, Memory=2460MB, Disk=145651MB) ...
* OS release is Ubuntu 18.04.4 LTS
* Preparing Kubernetes v1.17.3 on Docker 19.03.6 ...
  - kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
* Launching Kubernetes ... 
* Enabling addons: default-storageclass, storage-provisioner
* Configuring local host environment ...
* Done! kubectl is now configured to use "minikube"
```

通过`kubectl get nodes`命令来检查节点是否准备就绪。

```bash
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   19s   v1.17.3
```

## Kubectl Run

Run 命令根据指定的参数（例如映像或副本）创建部署。该部署发布给Kubernetes 主节点，启动所需 Pod 和容器的 。 *Kubectl run* 类似于 *docker run*，但*Kubectl run* 是在集群中运行的。

命令的格式为 `kubectl run <name of deployment> <properties>`

### 任务
以下命令将启动一个名为 *http* 的部署，它将启动一个基于 Docker 镜像 *katacoda/docker-http-server:latest* 的容器。

```bash
$ kubectl run http --image=katacoda/docker-http-server:latest --replicas=1
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/http created
```

使用*kubectl*查看各部署的状态。

```bash
$ kubectl get deployments
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
http   1/1     1            1           2m7s
```

可以通过 *describe* 查看 *Kubernetes* 创建了哪些内容。

```bash
$ kubectl describe deployment http
Name:                   http
Namespace:              default
CreationTimestamp:      Wed, 21 Jul 2021 08:04:55 +0000
Labels:                 run=http
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=http
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=http
  Containers:
   http:
    Image:        katacoda/docker-http-server:latest
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   http-774bb756bb (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  4m16s  deployment-controller  Scaled up replica set http-774bb756bb to 1
```

描述中包含有多少副本可用、指定的标签以及与部署相关的事件等信息。这些事件将突出显示可能发生的任何问题和错误。

在下一步中，我们将开放正在运行的服务。

## Kubectl Expose

创建部署后，我们可以使用 *kubectl* 创建一个服务，该服务在特定端口上开放 *Pod*。

通过 *kubectl expose* 开放新部署的 *http* 部署。该命令允许定义服务的不同参数以及如何开放该部署。

### 任务

使用以下命令将主机上的容器端口 *80* 公开 *8000* 绑定到主机的 *external-ip*。

```bash
$ kubectl expose deployment http --external-ip="172.17.0.44" --port=8000 --target-port=80
service/http exposed
```

接下来就能 ping 主机查看 HTTP 服务返回的结果。

```bash
$ curl http://172.17.0.44:8000
<h1>This request was processed by host: http-774bb756bb-m6tjp</h1>
```

## Kubectl Run and Expose

可以单独使用命令 *kubectl run* 来创建部署并将其开放。

### 任务

使用命令创建在端口 *8001* 上开放的第二个 http 服务。

```bash
$ kubectl run httpexposed --image=katacoda/docker-http-server:latest --replicas=1 --port=80 --hostport=8001
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/httpexposed created
```
接下来可以使用 `curl http://172.17.0.41:8001` 来访问该服务。

``` bash
$ curl http://172.17.0.44:8001
<h1>This request was processed by host: httpexposed-68cb8c8d4-r7qtt</h1>
```

在服务内，通过 Docker 端口映射开放了 Pod。因此，您将看不到使用 `kubectl get svc` 列出的服务

```bash
$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
http         ClusterIP   10.97.103.237   172.17.0.44   8000/TCP   4m47s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    44m
```

可以通过 `docker ps | grep httpexposed`命令查看详细信息。

```bash
$ docker ps | grep httpexposed
1690c6682f93        katacoda/docker-http-server   "/app"                   2 minutes ago       Up 2 minutes                               k8s_httpexposed_httpexposed-68cb8c8d4-r7qtt_default_6fb23c14-277e-4298-b325-c52ef4eb09a9_0
d9ae0076a0d4        k8s.gcr.io/pause:3.1          "/pause"                 2 minutes ago       Up 2 minutes        0.0.0.0:8001->80/tcp   k8s_POD_httpexposed-68cb8c8d4-r7qtt_default_6fb23c14-277e-4298-b325-c52ef4eb09a9_0
```

### 暂停容器

运行上面的命令，你会注意到端口暴露在 Pod 上，而不是 http 容器本身。 Pause 容器负责为 Pod 定义网络。 Pod 中的其他容器共享相同的网络命名空间。允许多个容器通过同一网络接口进行通信，提高了网络性能。

## 容器扩展

随着我们的部署运行，我们现在可以使用 *kubectl* 来扩展副本的数量。

扩展部署将请求 Kubernetes 启动额外的 Pod。然后，这些 Pod 将使用开放服务自动进行负载平衡。

### 任务

*kubectl scale* 命令能够为特定 `Deployment`或 `Replication Controller` 调整运行的 Pod 数量。

```bash
$ kubectl scale --replicas=3 deployment http
deployment.apps/http scaled
```

列出所有 pod，能够看到三个正在运行的 *http* 部署。

``` bash
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
http-774bb756bb-m6tjp         1/1     Running   0          20m
http-774bb756bb-q29v8         1/1     Running   0          45s
http-774bb756bb-x2xxv         1/1     Running   0          45s
httpexposed-68cb8c8d4-r7qtt   1/1     Running   0          9m3s
```

一旦每个 Pod 启动，就会被添加到负载均衡器服务中。通过`describe`，可以查看包含的端点和与之关联的 Pod。
Once each Pod starts it will be added to the load balancer service. By describing the service you can view the endpoint and the associated Pods which are included.

```bash
$ kubectl describe svc http
Name:              http
Namespace:         default
Labels:            run=http
Annotations:       <none>
Selector:          run=http
Type:              ClusterIP
IP:                10.97.103.237
External IPs:      172.17.0.44
Port:              <unset>  8000/TCP
TargetPort:        80/TCP
Endpoints:         172.18.0.4:80,172.18.0.6:80,172.18.0.7:80
Session Affinity:  None
Events:            <none>
```

向服务发出多次请求，可以通过返回信息看出请求将在不同的节点处理。

```bash
$ curl http://172.17.0.44:8000
<h1>This request was processed by host: http-774bb756bb-x2xxv</h1>
$ curl http://172.17.0.44:8000
<h1>This request was processed by host: http-774bb756bb-q29v8</h1>
$ curl http://172.17.0.44:8000
<h1>This request was processed by host: http-774bb756bb-q29v8</h1>
$ curl http://172.17.0.44:8000
<h1>This request was processed by host: http-774bb756bb-x2xxv</h1>
$ curl http://172.17.0.44:8000
<h1>This request was processed by host: http-774bb756bb-q29v8</h1>
$ curl http://172.17.0.44:8000
<h1>This request was processed by host: http-774bb756bb-q29v8</h1>
$ curl http://172.17.0.44:8000
<h1>This request was processed by host: http-774bb756bb-m6tjp</h1>
$ curl http://172.17.0.44:8000
<h1>This request was processed by host: http-774bb756bb-x2xxv</h1>
```
