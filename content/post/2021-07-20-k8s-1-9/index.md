---
title: Kubernetes初探（九）
description: 在Kubernetes上运行Stateful服务
date: 2021-07-23T14:23:00+0800
lastmod: '2021-07-23T14:51:00+0800'
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

> Katacoda在线课：[Running Stateful Services on Kubernetes](https://www.katacoda.com/courses/kubernetes/storage-introduction)
> 
> 本系列教程希望能通过交互式学习网站与传统方式结合，更高效、容易的学习知识。
> 本系列教程将使用 [Katacoda在线学习平台](https://www.katacoda.com) 完成学习。

## 部署 NFS 服务器

*NFS* 是一种允许节点通过网络读 / 写数据的协议。该协议的工作原理是让主节点运行 *NFS* 守护程序并存储数据。此主节点使某些目录可通过网络使用。

客户端访问通过驱动器挂载共享的主服务器。从应用程序的角度来看，它们正在写入本地磁盘。在背后，*NFS* 协议将其写入主服务器。

### 任务

在此场景中，出于演示和学习目的，*NFS* 服务器的角色由自定义容器处理。容器通过 NFS 提供目录并将数据存储在容器内。在生产环境中，建议配置专用的 *NFS Server*。

使用 `docker run -d --net=host \ --privileged --name nfs-server \ katacoda/contained-nfs-server:centos7 \ /exports/data-0001 /exports/data-0002` 命令启动 *NFS* 服务器

```bash
controlplane $ docker run -d --net=host \
>    --privileged --name nfs-server \
>    katacoda/contained-nfs-server:centos7 \
>    /exports/data-0001 /exports/data-0002
Unable to find image 'katacoda/contained-nfs-server:centos7' locally
centos7: Pulling from katacoda/contained-nfs-server
8d30e94188e7: Pull complete 
2b2b27f1f462: Pull complete 
133e63cf95fe: Pull complete 
Digest: sha256:5f2ea4737fe27f26be5b5cabaa23e24180079a4dce8d5db235492ec48c5552d1
Status: Downloaded newer image for katacoda/contained-nfs-server:centos7
65699a0a96bfc489fe2141a815ef12b03917f9bb667340b1be68dfe838d14bf3
```

*NFS* 服务器公开两个目录，*data-0001* 和 *data-0002*。在接下来的步骤中，这将用于存储数据。

## 部署 Persistent Volume

为了让 Kubernetes 了解可用的 *NFS* 共享，它需要一个 *PersistentVolume* 配置。 *PersistentVolume* 支持不同的数据存储协议，例如 *AWS EBS* 卷、*GCE* 存储、*OpenStack Cinder*、*Glusterfs* 和 *NFS*。该配置提供了存储和 *API* 之间的抽象，从而实现了一致的体验。

在使用 *NFS* 的情况下，一个 *PersistentVolume* 与一个 *NFS* 目录相关联。当容器使用完卷后，数据可以 **保留** 以备将来使用，也可以 **回收**，这意味着所有数据都将被删除。该策略由 *persistentVolumeReclaimPolicy* 选项定义。

*YAML* 文件结构：
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: <friendly-name>
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: <server-name>
    path: <shared-path>
```

该规范定义了关于 *PersistentVolume* 的额外元数据，包括有多少可用空间以及它是否具有读 / 写访问权限。

### 任务

使用 `cat nfs-0001.yaml nfs-0002.yaml` 查看 *YAML* 文件的内容。

**nfs-0001.yaml**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-0001
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 172.17.0.45
    path: /exports/data-0001
```

**nfs-0002.yaml**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-0002
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 172.17.0.45
    path: /exports/data-0002
```

创建两个新的 *PersistentVolume* 定义并指向两个可用的 NFS 共享。

```bash
controlplane $ kubectl create -f nfs-0001.yaml
persistentvolume/nfs-0001 created
controlplane $ kubectl create -f nfs-0002.yaml
persistentvolume/nfs-0002 created
```

创建后，使用 `kubectl get pv` 命令查看集群中的所有 *PersistentVolumes*

```bash
controlplane $ kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
nfs-0001   2Gi        RWO,RWX        Recycle          Available                                   66s
nfs-0002   5Gi        RWO,RWX        Recycle          Available                                   61s
```

## 部署 Persistent Volume Claim

一旦 *PersistentVolume* 可用，应用程序就可以声明该卷供其使用。该声明旨在阻止应用程序意外写入同一卷并导致冲突和数据损坏。

*Persistent Volume Claim* 指定了卷的要求。这包括所需的读/写访问和存储空间。一个例子如下：

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-mysql
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

## 任务

使用 `cat pvc-mysql.yaml pvc-http.yaml` 查看文件的内容

**pvc-mysql.yaml**

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-mysql
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

**pvc-http.yaml**

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-http
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

为两个不同的应用程序创建两个声明。 *MySQL Pod* 将使用一个声明，另一个由 *HTTP* 服务器使用。

```bash
controlplane $ kubectl create -f pvc-mysql.yaml
persistentvolumeclaim/claim-mysql created
controlplane $ kubectl create -f pvc-http.yaml
persistentvolumeclaim/claim-http created
```

创建后，使用 `kubectl get pvc` 查看集群中的所有 *PersistentVolumesClaims*，将打印出声明映射到的卷。

```bash
controlplane $ kubectl get pvc
NAME          STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
claim-http    Bound    nfs-0001   2Gi        RWO,RWX                       53s
claim-mysql   Bound    nfs-0002   5Gi        RWO,RWX                       56s
```

## Volume 的使用

定义部署后，它可以将自己分配给先前的声明。以下代码段定义了目录 */var/lib/mysql/data* 的卷挂载，该目录映射到存储 *mysql-persistent-storage*。名为 *mysql-persistent-storage* 的存储映射到名为 *claim-mysql* 的声明中。

```yaml
  spec:
      volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql/data
  volumes:
    - name: mysql-persistent-storage
      persistentVolumeClaim:
        claimName: claim-mysql
```

## 任务

使用以下命令查看 *Pod* 的定义。

```bash
controlplane $ cat pod-mysql.yaml pod-www.yaml
```

**pod-mysql.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    name: mysql
spec:
  containers:
  - name: mysql
    image: openshift/mysql-55-centos7
    env:
      - name: MYSQL_ROOT_PASSWORD
        value: yourpassword
      - name: MYSQL_USER
        value: wp_user
      - name: MYSQL_PASSWORD
        value: wp_pass
      - name: MYSQL_DATABASE
        value: wp_db
    ports:
      - containerPort: 3306
        name: mysql
    volumeMounts:
      - name: mysql-persistent-storage
        mountPath: /var/lib/mysql/data
  volumes:
    - name: mysql-persistent-storage
      persistentVolumeClaim:
        claimName: claim-mysql
```

**pod-http.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: www
  labels:
    name: www
spec:
  containers:
  - name: www
    image: nginx:alpine
    ports:
      - containerPort: 80
        name: www
    volumeMounts:
      - name: www-persistent-storage
        mountPath: /usr/share/nginx/html
  volumes:
    - name: www-persistent-storage
      persistentVolumeClaim:
        claimName: claim-http
```

启动两个具有 *Persistent Volume Claims* 的新 *Pod*。当 *Pod* 开始允许应用程序像本地目录一样读/写时，卷被映射到正确的目录。

```bash
controlplane $ kubectl create -f pod-mysql.yaml
pod/mysql created
controlplane $ kubectl create -f pod-www.yaml
pod/www created
```

可以使用 `kubectl get pods` 开始查看 Pod 的状态

```bash
controlplane $ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
mysql   1/1     Running   0          27s
www     1/1     Running   0          25s
```

如果 *Persistent Volume Claim* 未分配给 *Persistent Volume*，则 *Pod* 将处于 *Pending* 模式，直到它变为可用。在下一步中，我们将向卷读/写数据。

## 数据读写

我们的 *Pod* 现在可以进行读/写。 *MySQL* 将所有数据库更改存储到 *NFS* 服务器，而 *HTTP* 服务器将从 *NFS* 驱动器提供静态服务。升级、重新启动或将容器移动到不同的机器时，数据仍然可以访问。

要测试 *HTTP 服务器*，请编写一个“Hello World”*index.html* 主页。在这种情况下，我们知道 *HTTP* 目录将基于 *data-0001*，因为卷定义没有驱动足够的空间来满足 *MySQL* 大小要求。

```bash
controlplane $ docker exec -it nfs-server bash -c "echo 'Hello World' > /exports/data-0001/index.html"
```

根据 *Pod* 的 *IP*，访问 *Pod* 时，应该返回预期的响应。

```bash
controlplane $ ip=$(kubectl get pod www -o yaml |grep podIP | awk '{split($0,a,":"); print a[2]}'); echo $ip
10.32.0.6
controlplane $ curl $ip
Hello World
```

### 更新数据

当 *NFS* 共享上的数据发生变化时，*Pod* 会读取新更新的数据。

```bash
controlplane $ docker exec -it nfs-server bash -c "echo 'Hello NFS World' > /exports/data-0001/index.html"
controlplane $ curl $ip
Hello NFS World
```

## 重建 Pod

因为使用远程 *NFS* 服务器存储数据，如果 *Pod* 或主机宕机，那么数据仍然可用。

### 任务

删除 *Pod* 将导致它删除对任何持久卷的声明。新 *Pod* 可以获取并重新使用 *NFS* 共享。

**pod-www2.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: www2
  labels:
    name: www2
spec:
  containers:
  - name: www2
    image: nginx:alpine
    ports:
      - containerPort: 80
        name: www2
    volumeMounts:
      - name: www-persistent-storage
        mountPath: /usr/share/nginx/html
  volumes:
    - name: www-persistent-storage
      persistentVolumeClaim:
        claimName: claim-http
```

```bash
controlplane $ kubectl delete pod www
pod "www" deleted
controlplane $ kubectl create -f pod-www2.yaml
pod/www2 created
controlplane $ ip=$(kubectl get pod www2 -o yaml |grep podIP | awk '{split($0,a,":"); print a[2]}'); curl $ip
Hello NFS World
```

应用程序现在使用远程 *NFS* 进行数据存储。根据具体要求，这种相同的方法适用于其他存储引擎，例如 *GlusterFS*、*AWS EBS*、*GCE* 存储或 *OpenStack Cinder*。
