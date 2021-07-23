---
title: Kubernetes初探（十）
description: 使用 Kubernetes 管理 Secrets 和密码
date: 2021-07-23T14:54:00+0800
lastmod: '2021-07-23T15:24:00+0800'
image: kubernates.png
slug: k8s-basic-10
categories:
    - k8s-basic
    - katacoda
tags:
    - K8S
    - 容器
    - Katacoda
    - YAML
---

> Katacoda在线课：[Use Kubernetes To Manage Secrets And Passwords](https://www.katacoda.com/courses/kubernetes/managing-secrets)
> 
> 本系列教程希望能通过交互式学习网站与传统方式结合，更高效、容易的学习知识。
> 本系列教程将使用 [Katacoda在线学习平台](https://www.katacoda.com) 完成学习。

在此场景中，您将了解如何使用 *Kubernetes* 管理 *Secrets* 。 *Kubernetes* 允许您创建通过环境变量或作为卷挂载到 *Pod* 的 *Secrets* 。

只允许 *Secrets* （例如 SSL 证书或密码）通过基础架构团队以安全的方式进行管理，而不是将密码存储在应用程序的部署工件中。

## 启动 Kubernetes

首先，我们需要启动一个 *Kubernetes* 集群。

执行以下命令启动集群组件并下载 *Kubectl CLI*。

```bash
controlplane $ launch.sh
Waiting for Kubernetes to start...
Kubernetes started
```

## 创建 Secrets

*Kubernetes* 要求将 *Secrets* 编码为 *Base64* 的字符串。

使用命令行工具，我们可以创建 *Base64* 字符串并将它们存储为变量在文件中使用。 

```bash
controlplane $ username=$(echo -n "admin" | base64)
controlplane $ password=$(echo -n "a62fjbd37942dcs" | base64)
```

 *Secrets* 是使用 *YAML* 定义的。下面我们将使用上面定义的变量，并为它们提供我们的应用程序可以使用的标签。这将创建一个可以通过名称访问的键/值秘密的集合。

```bash
echo "apiVersion: v1
kind: Secret
metadata:
  name: test-secret
type: Opaque
data:
  username: $username
  password: $password" >> secret.yaml
```

**secret.yaml**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
type: Opaque
data:
  username: YWRtaW4=
  password: YTYyZmpiZDM3OTQyZGNz
```

这个 *YAML* 文件中定义可以与 *Kubectl* 一起使用来创建我们的 *Secrets* 。在启动需要访问密钥的 *Pod* 时，我们将通过名称引用集合。

## 任务: 创建 Secret

使用 *kubectl* 创建 *Secret*

```bash
controlplane $ kubectl create -f secret.yaml
secret/test-secret created
```

以下命令允许您查看定义的所有*Secret*集合。

```bash
controlplane $ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-z8shh   kubernetes.io/service-account-token   3      14m
test-secret           Opaque                                2      26s
```

在下一步中，我们将通过 *Pod* 使用这些 *Secret* 。

## 通过环境变量使用 *Secret*

在文件 *secret-env.yaml* 中，我们定义了一个有先前创建的 *Secret* 为环境变量的 *Pod* 。

使用 `cat secret-env.yaml` 查看文件

**secret-env.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
    - name: mycontainer
      image: alpine:latest
      command: ["sleep", "9999"]
      env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: test-secret
              key: username
        - name: SECRET_PASSWORD
          valueFrom:
            secretKeyRef:
              name: test-secret
              key: password
  restartPolicy: Never
```

为了填充环境变量，我们定义了名称，在本例中为 *SECRET_USERNAME*，以及 *Secret* 集合的名称和包含数据的密钥。

结构如下所示：

```yaml
- name: SECRET_USERNAME
valueFrom:
 secretKeyRef:
   name: test-secret
   key: username
```

### 任务

使用 `kubectl create -f secret-env.yaml` 启动 *Pod* 。

```bash
controlplane $ kubectl create -f secret-env.yaml
pod/secret-env-pod created
```

*Pod* 启动后，将输出填充的环境变量。 

```bash
controlplane $ kubectl exec -it secret-env-pod env | grep SECRET_
SECRET_USERNAME=admin
SECRET_PASSWORD=a62fjbd37942dcs
```

*Kubernetes* 在填充环境变量时解码 base64 值。您应该会看到我们定义的原始用户名/密码组合。这些变量现在可用于访问 API、数据库等。

您可以使用 `kubectl get pods` 检查 *Pod* 的状态。

```bash
controlplane $ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
secret-env-pod   1/1     Running   0          34s
```

在下一步中，我们将把 *Secret* 挂载为文件。

## 通过文件使用 *Secret*

使用环境变量在内存中存储 *Secret* 可能会导致它们意外泄漏。推荐的方法是将它们作为卷安装。

*Pod* 定义可以使用 `cat secret-pod.yaml` 查看。

**secret-pod.yaml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-vol-pod
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: test-secret
  containers:
    - name: test-container
      image: alpine:latest
      command: ["sleep", "9999"]
      volumeMounts:
          - name: secret-volume
            mountPath: /etc/secret-volume
```

要将 *Secret* 安装为卷，我们首先定义一个具有名称的卷，在本例中为 *Secret* 卷，并为其提供我们存储的 *Secret* 。

```yaml
volumes:
 - name: secret-volume
   secret:
     secretName: test-secret
```

当我们定义容器时，我们将创建的卷挂载到特定目录。应用程序将从该路径读取*Secret* 作为文件。

```yaml
volumeMounts:
 - name: secret-volume
   mountPath: /etc/secret-volume
```

### 任务

使用 `kubectl create -f secret-pod.yaml` 创建我们的新 *Pod* 。

```bash
controlplane $ kubectl create -f secret-pod.yaml
pod/secret-vol-pod created
```

启动后，您可以与安装的*Secret* 进行交互。例如，您可以列出所有可用的 *Secret* ，就好像它们是常规数据一样。

```bash
controlplane $ kubectl exec -it secret-vol-pod ls /etc/secret-volume
password  username
```

读取文件允许我们访问解码的 *Secret* 值。要访问我们使用的用户名

```bash
controlplane $ kubectl exec -it secret-vol-pod cat /etc/secret-volume/username
admin
```

对于密码，我们会读取密码文件。

```bash
admincontrolplane $ kubectl exec -it secret-vol-pod cat /etc/secret-volume/password
a62fjbd37942dcs
```
