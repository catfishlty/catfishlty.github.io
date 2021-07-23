---
title: Kubernetes初探（八）
description: Liveness 和 Readiness 健康检查
date: 2021-07-23T13:07:00+0800
lastmod: '2021-07-23T14:22:00+0800'
image: kubernates.png
slug: k8s-basic-8
categories:
    - k8s-basic
    - katacoda
tags:
    - K8S
    - 容器
    - Katacoda
    - YAML
---

> Katacoda在线课：[Liveness and Readiness Healthchecks](https://www.katacoda.com/courses/kubernetes/liveness-readiness-healthchecks)
> 
> 本系列教程希望能通过交互式学习网站与传统方式结合，更高效、容易的学习知识。
> 本系列教程将使用 [Katacoda在线学习平台](https://www.katacoda.com) 完成学习。

在此场景中，您将了解 *Kubernetes* 如何使用 *Readiness and Liveness Probes* 检查容器运行状况。

*Readiness Probe* 检查应用是否准备好开始处理流量。此探针解决了容器已启动的问题，但该进程仍在预热和配置自身，这意味着它尚未准备好接收流量。

*Liveness Probe* 确保应用程序健康并能够处理请求。

## 启动集群

首先，我们需要启动一个 *Kubernetes* 集群。

执行以下命令启动集群组件并下载 *Kubectl CLI*

```bash
controlplane $ launch.sh
Waiting for Kubernetes to start...
Kubernetes started
```

集群启动后，使用 `kubectl apply -f deploy.yaml` 部署演示应用程序。

**deploy.yaml**
```yaml
kind: List
apiVersion: v1
items:
- kind: ReplicationController
  apiVersion: v1
  metadata:
    name: frontend
    labels:
      name: frontend
  spec:
    replicas: 1
    selector:
      name: frontend
    template:
      metadata:
        labels:
          name: frontend
      spec:
        containers:
        - name: frontend
          image: katacoda/docker-http-server:health
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 1
            timeoutSeconds: 1
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 1
            timeoutSeconds: 1
- kind: ReplicationController
  apiVersion: v1
  metadata:
    name: bad-frontend
    labels:
      name: bad-frontend
  spec:
    replicas: 1
    selector:
      name: bad-frontend
    template:
      metadata:
        labels:
          name: bad-frontend
      spec:
        containers:
        - name: bad-frontend
          image: katacoda/docker-http-server:unhealthy
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 1
            timeoutSeconds: 1
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 1
            timeoutSeconds: 1
- kind: Service
  apiVersion: v1
  metadata:
    labels:
      app: frontend
      kubernetes.io/cluster-service: "true"
    name: frontend
  spec:
    type: NodePort
    ports:
    - port: 80
      nodePort: 30080
    selector:
      app: frontend
```

```bash
controlplane $ kubectl apply -f deploy.yaml
replicationcontroller/frontend created
replicationcontroller/bad-frontend created
service/frontend created
```

## Readiness Probe

在部署集群时，还部署了两个 *Pod* 来演示健康检查。您可以使用 `cat deploy.yaml` 查看部署。

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 1
  timeoutSeconds: 1
```

可以根据您的应用程序更改设置来调用不同的端点，例如 */ping*。

### 获取状态

第一个 *Pod* *bad-frontend* 是一个 *HTTP* 服务，它总是返回 500 错误，表明它没有正确启动。您可以使用 `kubectl get pods --selector="name=bad-frontend"` 命令查看 Pod 的状态。

```bash
controlplane $ kubectl get pods --selector="name=bad-frontend"
NAME                 READY   STATUS    RESTARTS   AGE
bad-frontend-jk5z2   0/1     Running   2          78s
```

*Kubectl* 将返回使用我们的特定标签部署的 *Pod*。因为健康检查失败，它会显示没有容器为准备就绪的状态，同时它还将显示容器的重启尝试次数。


要了解有关失败原因的更多详细信息，请对该 *Pod* 使用描述命令。

```bash
controlplane $ pod=$(kubectl get pods --selector="name=bad-frontend" --output=jsonpath={.items..metadata.name})
controlplane $ kubectl describe pod $pod
Name:               bad-frontend-jk5z2
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               controlplane/172.17.0.67
Start Time:         Fri, 23 Jul 2021 05:54:59 +0000
Labels:             name=bad-frontend
Annotations:        <none>
Status:             Running
IP:                 10.32.0.6
Controlled By:      ReplicationController/bad-frontend
Containers:
  bad-frontend:
    Container ID:   docker://811c3fa6d76c13f4b5bc7a6b2d6f514292e9673be46572a79b8f2d3d5e36bc62
    Image:          katacoda/docker-http-server:unhealthy
    Image ID:       docker-pullable://katacoda/docker-http-server@sha256:bea95c69c299c690103c39ebb3159c39c5061fee1dad13aa1b0625e0c6b52f22
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    2
      Started:      Fri, 23 Jul 2021 05:57:07 +0000
      Finished:     Fri, 23 Jul 2021 05:57:37 +0000
    Ready:          False
    Restart Count:  4
    Liveness:       http-get http://:80/ delay=1s timeout=1s period=10s #success=1 #failure=3
    Readiness:      http-get http://:80/ delay=1s timeout=1s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-5n24z (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  default-token-5n24z:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-5n24z
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                   From                   Message
  ----     ------     ----                  ----                   -------
  Normal   Scheduled  2m53s                 default-scheduler      Successfully assigned default/bad-frontend-jk5z2 to controlplane
  Normal   Pulling    2m52s                 kubelet, controlplane  Pulling image "katacoda/docker-http-server:unhealthy"
  Normal   Pulled     2m45s                 kubelet, controlplane  Successfully pulled image "katacoda/docker-http-server:unhealthy"
  Normal   Created    105s (x3 over 2m45s)  kubelet, controlplane  Created container bad-frontend
  Normal   Started    105s (x3 over 2m45s)  kubelet, controlplane  Started container bad-frontend
  Warning  Unhealthy  105s (x6 over 2m35s)  kubelet, controlplane  Liveness probe failed: HTTP probe failed with statuscode: 500
  Normal   Killing    105s (x2 over 2m15s)  kubelet, controlplane  Container bad-frontend failed liveness probe, will be restarted
  Normal   Pulled     105s (x2 over 2m15s)  kubelet, controlplane  Container image "katacoda/docker-http-server:unhealthy" already present on machine
  Warning  Unhealthy  103s (x7 over 2m43s)  kubelet, controlplane  Readiness probe failed: HTTP probe failed with statuscode: 500
```

### Readiness

我们的第二个 Pod，*frontend*，在启动时返回 *OK* 状态。

```bash
controlplane $ kubectl get pods --selector="name=frontend"
NAME             READY   STATUS    RESTARTS   AGE
frontend-54czd   1/1     Running   0          4m9s
```

#### Liveness Probe

由于我们的第二个 *Pod* 当前处于健康状态，我们可以模拟发生的故障。

目前，该 *Pod* 应该没有发生崩溃。

```bash
controlplane $ kubectl get pods --selector="name=frontend"
NAME             READY   STATUS    RESTARTS   AGE
frontend-54czd   1/1     Running   0          7m17s
```

### 让服务崩溃

*HTTP* 服务器有一个额外的端点，这将导致它返回 *500* 错误。使用 *kubectl exec* 可以调用端点。

```bash
controlplane $ pod=$(kubectl get pods --selector="name=frontend" --output=jsonpath={.items..metadata.name})
controlplane $ kubectl exec $pod -- /usr/bin/curl -s localhost/unhealthy
```

## Liveness

*Kubernetes* 将根据配置执行 *Liveness Probe*。如果探测器失败，*Kubernetes* 将销毁并重新创建失败的容器。执行上面的命令使服务崩溃并观察 *Kubernetes* 自动恢复它。

```bash
controlplane $ kubectl get pods --selector="name=frontend"
NAME             READY   STATUS    RESTARTS   AGE
frontend-54czd   1/1     Running   1          9m42s
```

检查可能需要一些时间才能检测到。