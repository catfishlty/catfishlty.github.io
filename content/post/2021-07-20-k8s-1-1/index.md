---
title: Kubernetes初探（一） - 启动一个单节点集群
description: K8S 在线课
date: 2021-07-20T10:21:00+0800
lastmod: '2021-07-20T10:21:00+0800'
image: kubernetes-horizontal-color.png
slug: k8s-basic-1
categories:
    - k8s-basic
    - katacoda
tags:
    - K8S
    - 容器
    - Katacoda
---

# Kubernetes初探（一） - 启动一个单节点集群
> Katacoda在线课：https://www.katacoda.com/courses/kubernetes/launch-single-node-cluster
> 
> 本系列教程希望能通过交互式学习网站与传统方式结合，更高效、容易的学习知识。
> 本系列教程将使用 [Katacoda在线学习平台](https://www.katacoda.com) 完成学习。

`Minikube` 是一个可以轻松在本地运行 `Kubernetes` 的工具。 `Minikube` 是一个在本地上计算机的虚拟机内运行一个单节点 `Kubernetes` 集群，便于用户能够完成日常开发工作，同时也能够让新用户快速了解`Kubernetes`。 

详情见： https://github.com/kubernetes/minikube

## 步骤 1 - 启动`Minikube`

`Minikube`已经安装并配置到环境中。通过运行 `minikube version` 命令检查它是否已正确安装。

``` bash
$ minikube version

minikube version: v1.8.1
commit: cbda04cf6bbe65e987ae52bb393c10099ab62014
```

通过运行 `minikube start` 命令启动集群。

``` bash
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

现在，您的在线终端中有一个正在运行的 `Kubernetes` 集群。 `Minikube`会启动一个虚拟机为`K8S`集群提供运行环境。

## 步骤 2 - 集群信息

用户使用 `kubectl` 客户端与集群交互。该工具用于管理 `Kubernetes` 和在集群上运行的应用程序.

通过`kubectl cluster-info`命令查看集群详情信息和健康状态。

```bash
$ kubectl cluster-info
Kubernetes master is running at https://172.17.0.10:8443
KubeDNS is running at https://172.17.0.10:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

通过`kubectl get nodes`查看集群中的各节点信息。

```bash
$ kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   9m15s   v1.17.3
```

如果节点被标记为 **NotReady** 则它仍在启动组件阶段。

此命令显示可用于托管我们的应用程序的所有节点。现在我们只有一个节点，可以看到它的状态是`Ready`。

## 步骤 3 - 部署容器

现在可以通过 `Kubernetes` 集群来部署容器。

`kubectl run`命令能够将容器部署到集群中。

```bash
$ kubectl create deployment first-deployment --image=katacoda/docker-http-server
deployment.apps/first-deployment created
```

部署状态可以通过`Pods`的运行状态得知。

```bash
$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
first-deployment-666c48b44-n7qjz   1/1     Running   0          60s
```

容器运行后，可以根据需求使用不同的网络选项公开暴露接口。其中的一种解决方案是 `NodePort`，它为容器提供动态端口。

``` bash
$ kubectl expose deployment first-deployment --port=80 --type=NodePort
service/first-deployment exposed
```
下面的命令能够查询到绑定的端口，并且能够发送HTTP请求进行测试。

``` bash
$ export PORT=$(kubectl get svc first-deployment -o go-template='{{range.spec.ports}}{{if .nodePort}}{{.nodePort}}{{"\n"}}{{end}}{{end}}')
$ echo "Accessing host01:$PORT"
Accessing host01:32492
$ curl host01:$PORT
<h1>This request was processed by host: first-deployment-666c48b44-n7qjz</h1>
```

结果显示的是处理请求的容器。

## 步骤 4 - 仪表盘

使用 `Minikube`命令启用仪表板
```bash
$ minikube addons enable dashboard
* The 'dashboard' addon is enabled
```
通过使用`yaml`文件来定义部署Kubernetes Dashboard。**该配置仅适用于`Katacoda`**

**/opt/kubernetes-dashboard.yaml**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/minikube-addons: dashboard
  name: kubernetes-dashboard
  selfLink: /api/v1/namespaces/kubernetes-dashboard
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: kubernetes-dashboard
  name: kubernetes-dashboard-katacoda
  namespace: kubernetes-dashboard
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 9090
    nodePort: 30000
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort
```

``` bash
$ kubectl apply -f /opt/kubernetes-dashboard.yaml
namespace/kubernetes-dashboard configured
service/kubernetes-dashboard-katacoda created
```

可以使用`Kubernetes`仪表板查看部署到集群中的应用程序。仪表板将部署在 `30000` 端口，但需要一段时间才能完成启动。

要查看 `Dashboard`启动的进度，请使用 `kubectl get pods -n kubernetes-dashboard -w` 查看 `kube-system`命名空间中的 `Pod`

```bash
$ kubectl get pods -n kubernetes-dashboard -w
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-7b64584c5c-jmkls   1/1     Running   0          6m15s
kubernetes-dashboard-79d9cd965-fcb4t         1/1     Running   0          6m14s
```

启动好后，仪表盘的在线访问地址为： https://2886795306-30000-ollie07.environments.katacoda.com/