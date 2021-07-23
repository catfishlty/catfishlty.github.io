---
title: Kubernetes初探（二）
description: 使用 Kubeadm 启动一个多节点集群
date: 2021-07-20T11:05:00+0800
lastmod: '2021-07-20T11:05:00+0800'
image: kubernates.png
slug: k8s-basic-2
categories:
    - k8s-basic
    - katacoda
tags:
    - K8S
    - 容器
    - Katacoda
---

> Katacoda在线课：[Launch a multi-node cluster using Kubeadm](https://www.katacoda.com/courses/kubernetes/getting-started-with-kubeadm)
> 
> 本系列教程希望能通过交互式学习网站与传统方式结合，更高效、容易的学习知识。
> 本系列教程将使用 [Katacoda在线学习平台](https://www.katacoda.com) 完成学习。

在此场景中，您将学习如何使用 `Kubeadm` 启动 `Kubernetes` 集群。

`Kubeadm` 解决了TLS 加密配置、 Kubernetes 核心组件部署和额外节点集群加入的问题。启动的集群通过 RBAC 等机制开箱即用。

关于`Kubeadm`的更多信息可以参考： https://github.com/kubernetes/kubeadm

## 初始化 Master

`Kubeadm` 已经安装在节点上。软件包适用于 Ubuntu 16.04+、CentOS 7 或 HypriotOS v1.0.1+。

初始化集群的第一步是启动`Master节点`。 `Master节点` 负责运行控制平面组件、`etcd` 和 API 服务器。客户端能够与 API 通信，能够完成工作负载的调度和集群状态的管理。

### 任务

下面的命令将使用已知的`Token`简化初始化集群的步骤。

``` bash
controlplane $ kubeadm init --token=102952.1a7dd4cc8d1f4cc5 --kubernetes-version $(kubeadm version -o short)
[init] Using Kubernetes version: v1.14.0
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [controlplane localhost] and IPs [172.17.0.86 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [controlplane localhost] and IPs [172.17.0.86 127.0.0.1 ::1]
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [controlplane kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.17.0.86]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 17.503676 seconds
[upload-config] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.14" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --experimental-upload-certs
[mark-control-plane] Marking the node controlplane as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node controlplane as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 102952.1a7dd4cc8d1f4cc5
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.17.0.86:6443 --token 102952.1a7dd4cc8d1f4cc5 \
    --discovery-token-ca-cert-hash sha256:ab56a643a2d683bc1deeb483f0f946d4a774c4
```

在生产环境中，建议排除使用 `kubeadm` 生成的令牌。

需要客户端配置和证书来管理 Kubernetes 集群。这个配置是在 *kubeadm* 初始化集群时创建的。该命令将配置复制到用户主目录并设置用于 CLI 的环境变量。

``` bash
controlplane $ sudo cp /etc/kubernetes/admin.conf $HOME/
controlplane $ sudo chown $(id -u):$(id -g) $HOME/admin.conf
controlplane $ export KUBECONFIG=$HOME/admin.conf
```

## 部署容器网络接口 Container Networking Interface (CNI) 

容器网络接口 (CNI) 定义了不同节点及其工作负载是如何通信的。有多个网络提供商可用，其中一些在 [here](https://kubernetes.io/docs/admin/addons/) 中列出。

### 任务

在这个场景中，我们将使用 `WeaveWorks`的`CNI`。

**/opt/weave-kube.yaml**
可以通过`cat /opt/weave-kube.yaml`命令查看部署定义。
``` yaml
apiVersion: v1
kind: List
items:
  - apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
    rules:
      - apiGroups:
          - ''
        resources:
          - pods
          - namespaces
          - nodes
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - networking.k8s.io
        resources:
          - networkpolicies
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - ''
        resources:
          - nodes/status
        verbs:
          - patch
          - update
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
    roleRef:
      kind: ClusterRole
      name: weave-net
      apiGroup: rbac.authorization.k8s.io
    subjects:
      - kind: ServiceAccount
        name: weave-net
        namespace: kube-system
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
    rules:
      - apiGroups:
          - ''
        resourceNames:
          - weave-net
        resources:
          - configmaps
        verbs:
          - get
          - update
      - apiGroups:
          - ''
        resources:
          - configmaps
        verbs:
          - create
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
    roleRef:
      kind: Role
      name: weave-net
      apiGroup: rbac.authorization.k8s.io
    subjects:
      - kind: ServiceAccount
        name: weave-net
        namespace: kube-system
  - apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
    spec:
      minReadySeconds: 5
      selector:
        matchLabels:
          name: weave-net
      template:
        metadata:
          labels:
            name: weave-net
        spec:
          containers:
            - name: weave
              command:
                - /home/weave/launch.sh
              env:
                - name: IPALLOC_RANGE
                  value: 10.32.0.0/24
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.nodeName
              image: 'docker.io/weaveworks/weave-kube:2.6.0'
              readinessProbe:
                httpGet:
                  host: 127.0.0.1
                  path: /status
                  port: 6784
              resources:
                requests:
                  cpu: 10m
              securityContext:
                privileged: true
              volumeMounts:
                - name: weavedb
                  mountPath: /weavedb
                - name: cni-bin
                  mountPath: /host/opt
                - name: cni-bin2
                  mountPath: /host/home
                - name: cni-conf
                  mountPath: /host/etc
                - name: dbus
                  mountPath: /host/var/lib/dbus
                - name: lib-modules
                  mountPath: /lib/modules
                - name: xtables-lock
                  mountPath: /run/xtables.lock
            - name: weave-npc
              env:
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.nodeName
              image: 'docker.io/weaveworks/weave-npc:2.6.0'
              resources:
                requests:
                  cpu: 10m
              securityContext:
                privileged: true
              volumeMounts:
                - name: xtables-lock
                  mountPath: /run/xtables.lock
          hostNetwork: true
          hostPID: true
          restartPolicy: Always
          securityContext:
            seLinuxOptions: {}
          serviceAccountName: weave-net
          tolerations:
            - effect: NoSchedule
              operator: Exists
          volumes:
            - name: weavedb
              hostPath:
                path: /var/lib/weave
            - name: cni-bin
              hostPath:
                path: /opt
            - name: cni-bin2
              hostPath:
                path: /home
            - name: cni-conf
              hostPath:
                path: /etc
            - name: dbus
              hostPath:
                path: /var/lib/dbus
            - name: lib-modules
              hostPath:
                path: /lib/modules
            - name: xtables-lock
              hostPath:
                path: /run/xtables.lock
                type: FileOrCreate
      updateStrategy:
        type: RollingUpdate
```


使用`kubectl apply`命令来完成部署工作。.

``` bash
controlplane $ kubectl apply -f /opt/weave-kube.yaml
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created
```

`Weave` 现在将在集群上部署为一系列的 `Pod`。
可以通过 `kubectl get pod -n kube-system`命令查看状态。

``` bash
controlplane $ kubectl get pod -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-fb8b8dccf-gw8z6                1/1     Running   0          12m
coredns-fb8b8dccf-qzd49                1/1     Running   0          12m
etcd-controlplane                      1/1     Running   0          12m
kube-apiserver-controlplane            1/1     Running   0          11m
kube-controller-manager-controlplane   1/1     Running   0          11m
kube-proxy-2kjsl                       1/1     Running   0          13m
kube-scheduler-controlplane            1/1     Running   1          11m
weave-net-2dtbs                        2/2     Running   0          82s
```

需要安装Weave在你的集群中时，可以在 https://www.weave.works/docs/net/latest/kube-addon/  找到更多详细信息。

## 加入集群

一旦 `Master` 和 `CNI` 初始化完成，其他节点只要拥有正确的令牌就可以加入集群。令牌可以通过 `kubeadm token` 进行管理，例如 `kubeadm token list`。


``` bash
controlplane $ kubeadm token list
TOKEN                     TTL       EXPIRES                USAGES                   DESCRIPTION                                                EXTRA GROUPS
102952.1a7dd4cc8d1f4cc5   23h       2021-07-21T10:00:20Z   authentication,signing   The default bootstrap token generated by 'kubeadm init'.   system:bootstrappers:kubeadm:default-node-token
```

### 任务

在第二个节点上，使用主节点的 IP 地址，运行命令加入集群。

```
node01 $ kubeadm join --discovery-token-unsafe-skip-ca-verification --token=102952.1a7dd4cc8d1f4cc5 172.17.0.86:6443
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.14" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

`Node01`节点与 `Master` 节点初始化提供的命令相同。

`--discovery-token-unsafe-skip-ca-verification` 标签用于绕过 Discovery Token 验证。由于此令牌是动态生成的，因此我们无法将其包含在步骤中。在生产环境中，使用 `kubeadm init` 提供的令牌。

## 查看节点

集群现已初始化。主节点将负责管理集群，另一个工作节点将负责运行我们的容器工作负载。

### 任务

Kubernetes CLI，称为 *kubectl*，现在可以使用配置访问集群。例如，下面的命令将返回我们集群中的两个节点信息。

``` bash
controlplane $ kubectl get nodes
NAME           STATUS   ROLES    AGE   VERSION
controlplane   Ready    master   25m   v1.14.0
node01         Ready    <none>   2m    v1.14.0
```

## 部署 Pod

集群中两个节点的状态现在应该是 `Ready`。这表示接下来可以进行调度和启动部署。

使用 `Kubectl`，可以部署 `Pod`。命令总是由 `Master` 发出，每个节点只负责运行工作负载。

下面的命令是基于 `Docker` 镜像 *katacoda/docker-http-server* 创建一个 `Pod`。

```
controlplane $ kubectl create deployment http --image=katacoda/docker-http-server:latest
deployment.apps/http created
```
可以使用`kubectl get pods`查看`Pod`创建的状态

``` bash
controlplane $ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
http-7f8cbdf584-jcdrj   1/1     Running   0          70s
```

运行后，可以看到节点上`Docker` 容器的运行状态。

```
node01 $ docker ps | grep docker-http-server
d874d8c3151b        katacoda/docker-http-server   "/app"                   About a minute ago   Up About a minute                       k8s_docker-http-server_http-7f8cbdf584-jcdrj_default_2894ce10-e945-11eb-b87f-0242ac110056_0
```

## 部署仪表盘

Kubernetes 有一个基于 Web 的仪表板应用，提供对 Kubernetes 集群的查看与管理能力。

### 任务

使用 `kubectl apply -f dashboard.yaml` 命令部署仪表板。

``` bash
controlplane $ kubectl apply -f dashboard.yaml
secret/kubernetes-dashboard-certs created
serviceaccount/kubernetes-dashboard created
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
deployment.apps/kubernetes-dashboard created
service/kubernetes-dashboard created
```

仪表板将部署到 *kube-system* 命名空间中。使用 `kubectl get pods -n kube-system`命令 查看部署状态。

``` bash
controlplane $ kubectl get pods -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-fb8b8dccf-rjzxp                 1/1     Running   1          88s
coredns-fb8b8dccf-tw8cm                 1/1     Running   1          88s
etcd-controlplane                       1/1     Running   0          16s
kube-apiserver-controlplane             1/1     Running   0          23s
kube-controller-manager-controlplane    1/1     Running   1          61s
kube-proxy-6xv6k                        1/1     Running   0          84s
kube-proxy-fn84g                        1/1     Running   0          88s
kube-scheduler-controlplane             1/1     Running   2          61s
kubernetes-dashboard-5f57845f9d-jblx7   1/1     Running   0          5s
weave-net-6gcbd                         2/2     Running   1          88s
weave-net-8s6bt                         2/2     Running   1          84s
```

需要 `ServiceAccount` 才能登录。 `ClusterRoleBinding` 用于为新的 ServiceAccount (*admin-user*) 分配集群上的 *cluster-admin* 角色。

```
cat <<EOF | kubectl create -f - 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
EOF
```

`Dashboard`可以控制 Kubernetes 的所有方面。通过 `ClusterRoleBinding` 和 `RBAC`，可以根据安全要求来定义不同级别的权限。有关为仪表板创建用户的更多信息，请参考 [Dashboard documentation](https://github.com/kubernetes/dashboard/wiki/Creating-sample-user)。

创建 `ServiceAccount` 后，可以通过以下方式获取登录令牌：

```bash
controlplane $ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
Name:         attachdetach-controller-token-bltlp
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: attachdetach-controller
              kubernetes.io/service-account.uid: 0ced9bf4-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhdHRhY2hkZXRhY2gtY29udHJvbGxlci10b2tlbi1ibHRscCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhdHRhY2hkZXRhY2gtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjBjZWQ5YmY0LWU5ZDQtMTFlYi1iMjViLTAyNDJhYzExMDAwOSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphdHRhY2hkZXRhY2gtY29udHJvbGxlciJ9.x0a2m2SpBrwj8sARrvRfuct2ghrDydxYzyFmDL93hATdS_59zaueB4SrgHner8gOu_zrx9PLWcSoZNBxRblZuxJj8Di9TKUTjyDBE6txc4_0H8nseuQzljvRTbjUjEcL7fp8H1j4MrJgT4GrYU-n1gAOl6NIfk8FmpFpuUUS6G_IfIDfS60YpZRlqZGp14NuaL0RC71PsERnP6ZYnuRKTbYNfVNeURqVR4pY7XwCcQZFALaJcTf9SgwJLAAqLQFgKd9c2MYnHdnU5cnqjrYxi_b5kGBKUcRzACrHjh0uGG6ErHeo6-GbA-0NDdIMI7qKa3If0oh_GOQKmyB5cyx-Rw


Name:         bootstrap-signer-token-cstqm
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: bootstrap-signer
              kubernetes.io/service-account.uid: 0dc1b223-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJib290c3RyYXAtc2lnbmVyLXRva2VuLWNzdHFtIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImJvb3RzdHJhcC1zaWduZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwZGMxYjIyMy1lOWQ0LTExZWItYjI1Yi0wMjQyYWMxMTAwMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06Ym9vdHN0cmFwLXNpZ25lciJ9.hkciM9KpWYYPyi5MJbm67oqzuskjDB_Rj4Fz5THhtN_jJAO6go4oxz-NPparOi7kxaEU0yBLb_xLY42jOXvG_Q2Dd6gR2u9yc7nfwr67koW3lck4S0PCYSuGo-FqUndXS1a3BpT5SFAHhb52FdwEYiedevLz9C5wUDT8sVgMYSnHvEX_jwBmjrhhOohGxTNs_-_HyGMNNLfEOSbjaEtzkDFos0n3qPAJ95uaGKJ2laDsctVpHCeRx5hNCv8E7dvvR1Qp4XXDVDQrodkzkuG4LSKviYLzMLX1DM2XES3HQ7Q2uTtAICqxIDBjRlhfAPv_z02tqAbCf4fF6_CFlYK1ag


Name:         bootstrap-token-102952
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Type:  bootstrap.kubernetes.io/token

Data
====
token-secret:                    16 bytes
usage-bootstrap-authentication:  4 bytes
usage-bootstrap-signing:         4 bytes
auth-extra-groups:               47 bytes
description:                     56 bytes
expiration:                      20 bytes
token-id:                        6 bytes


Name:         certificate-controller-token-mw9rl
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: certificate-controller
              kubernetes.io/service-account.uid: 0cf891ba-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJjZXJ0aWZpY2F0ZS1jb250cm9sbGVyLXRva2VuLW13OXJsIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImNlcnRpZmljYXRlLWNvbnRyb2xsZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwY2Y4OTFiYS1lOWQ0LTExZWItYjI1Yi0wMjQyYWMxMTAwMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06Y2VydGlmaWNhdGUtY29udHJvbGxlciJ9.NIZpVmhuJUcrpbZJApUt7ODgqIPU9RN4UZcr6he83xA_66-4ORQVddj1oQQg4G0p4fSPckWfP0n7pSxahZ90VZ-NCxPeCVLThLZ563PUOIljsjfROgFLQ4eeWfRML0OHyhAgoTtfEdcHi4_IzyhSrvCOweM9oPFfdx9MdTNAG9znyQsCi4g1YNHwn7t4r8h-BouW4yRYMEOM9HVZkcDVWhmF9PDv9s235INvIAjco5JvCAeDKvw48hdSrm4RWQPmZ7yE71iBDymDF_Ntr9w12H_4FPOYcARfFOmIj1k5binKiHaOlWtfbKYGYFM4tAxqI-ErFx4lshkLNJm3ICMJSQ


Name:         clusterrole-aggregation-controller-token-xh52b
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: clusterrole-aggregation-controller
              kubernetes.io/service-account.uid: 0acea7db-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJjbHVzdGVycm9sZS1hZ2dyZWdhdGlvbi1jb250cm9sbGVyLXRva2VuLXhoNTJiIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImNsdXN0ZXJyb2xlLWFnZ3JlZ2F0aW9uLWNvbnRyb2xsZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwYWNlYTdkYi1lOWQ0LTExZWItYjI1Yi0wMjQyYWMxMTAwMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06Y2x1c3RlcnJvbGUtYWdncmVnYXRpb24tY29udHJvbGxlciJ9.SgSxj_9uNLbAU9zYK4sTUYL4STx9CSYflGe0GwqGz_f3K7Db5NhpCuCoCcLYVJ7I2xogXICDJV016ZaNEAKJHWNJNeO-Ye378zjgddYmLAxX0uNw9OfSSiHKq-ksHgkKmoxwixgzVw2Df6PZGeYrKZZMSGuW96hkzd5JJx9Jhg2c4k8Fpqcpg5t-N4F5ZC8Ix4THpOmBueB8BaKox6OS5kUxqBAmdkJxZur1Wl8BvJV3Pnl6qbzYa0oPqun6N-WckGjVLrOlll1E6moABT1i8udZeRSrsX7YjjWgnbVyuyP0vzY7T1OQz-jcS0Lw3shMSIeANpwfFFB5XNvacsIV9A


Name:         coredns-token-z5tnd
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: coredns
              kubernetes.io/service-account.uid: 0ce34e5b-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJjb3JlZG5zLXRva2VuLXo1dG5kIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImNvcmVkbnMiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwY2UzNGU1Yi1lOWQ0LTExZWItYjI1Yi0wMjQyYWMxMTAwMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06Y29yZWRucyJ9.vTVtFW7YZ-_dxjyiHjrkJnxyw3Q1c2oy43wEfs71tnD5GIDcXjioGXRn4OFvGJYzUzSM7fujZy6e6lXUDZBx-PkiRZzaqbxZTuY1skfTKdAm31h6xbOESSvuFcR97eASVMv5WNdFNSXZik4P3_pEoF7XUKnOnY-o3nk-MZNWRhmdcOvj-BhBmTqXY9CQDvaW-P2MSQhaGvP-yVYBFHqNvOH_XgaBVdW5c35AuLGz3pVAiBl9s_Ghrkc5h0KrhU_eko5YAMGcXHUQPask-EVy4MYes_3ube_PbQLTvTgVZ6XwMjPNIOkYvrGsZrRZbRCDXKqc0KBFi2YT_dQWaoiNgg
ca.crt:     1025 bytes


Name:         cronjob-controller-token-6hfqf
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: cronjob-controller
              kubernetes.io/service-account.uid: 0e4b090e-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJjcm9uam9iLWNvbnRyb2xsZXItdG9rZW4tNmhmcWYiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiY3JvbmpvYi1jb250cm9sbGVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMGU0YjA5MGUtZTlkNC0xMWViLWIyNWItMDI0MmFjMTEwMDA5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmNyb25qb2ItY29udHJvbGxlciJ9.SKICsFaSIGnrn9pfqWg_Vrw1jBvYz_bc95tJ53FL_M9b6YQKgHRQZZH1DQciGdSd27nYSWnoG9L-ftKEZhaU7ABAAga3lNzyDJhgPdYtCy8OlxzY2nY2RdDbCsFRjaK76aIJQf_u0z8K-JSz6CjELBj294WWbXoAreCtZiQGRrrupgtUIPjUloaqx07BO2Y29N2ZMkELj_Ye5rSeffnz_djSFdwTIt6B3Jtxjp55DZjMzaj_coqmVXfGq4K90wkKlWLIbbAGYlRqEfJCpIK2OeCy5WF2EahlAUECHEQi5NqEQ1DoiVUj1LtdwQOGKXlQYT6h6GSLY_E1Q1Xv3vWFjQ


Name:         daemon-set-controller-token-6ftrt
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: daemon-set-controller
              kubernetes.io/service-account.uid: 0e251630-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYWVtb24tc2V0LWNvbnRyb2xsZXItdG9rZW4tNmZ0cnQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFlbW9uLXNldC1jb250cm9sbGVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMGUyNTE2MzAtZTlkNC0xMWViLWIyNWItMDI0MmFjMTEwMDA5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhZW1vbi1zZXQtY29udHJvbGxlciJ9.OTdB75fb-wdKbbcZJ3mS08JuJsq3urdovVUgiADbpAZnL4pVZWdAmrLadYVKFNqfXOlCaSqQefLNgSWsGcJee-YSQT6snP87Wplfo7VH05h9Xwnoz21uWpkjhIlioWbcKL1asBuBsvpSczqg4Bt3cyByxdrIJjBmBvoJpFhwf04IqyJ6ugxDLPTH-8CC5r5Xn6etm8Ey-p1rwQJb1u2FEq8K97nihv87TByrHLQv-CMn1wxfGHDYOEAneis2s0pWotvxrHWkFg1WnriDAq4hryGPDA9GRLqveFdCL76XTuT9plYFYMxLRF-8_EuwOwTa0F-LYm98o3iDhbvwkTDhiA
ca.crt:     1025 bytes


Name:         default-token-jw2ql
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 0fdb30ea-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZWZhdWx0LXRva2VuLWp3MnFsIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwZmRiMzBlYS1lOWQ0LTExZWItYjI1Yi0wMjQyYWMxMTAwMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06ZGVmYXVsdCJ9.ijnMq7u2xRTNKXov89YF67O27Jn7uLqnRYzK4V95pWt22dN3Qayf7U-jlLm6uWec79FLKNM_08IjnobzVxnzrFhRqLqOT55mkyLfXdNdR9EE__CrPOs-PiSIsHNbAG64bUSDOwbGaFjKROFBz1dwHj1O_XQZfLNaH3_LhLzOHtxJ4_FyOOf-hhjEWgjOptptTMkU3N_z7TRU4zzX9BUWTDg5s6JBpFkDSbZ-_yu-uzl4JiEBUwf0MY3YC79caRp9uFgwuaMCoFJYV7gSqWA5IpqgQe225LBTBS2HHLPJE67WLlSrkiEe4gvngnGtO_1RkO2MBCXgVp5icSFw4zxEzw


Name:         deployment-controller-token-fctz8
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: deployment-controller
              kubernetes.io/service-account.uid: 0cd7cffc-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZXBsb3ltZW50LWNvbnRyb2xsZXItdG9rZW4tZmN0ejgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVwbG95bWVudC1jb250cm9sbGVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMGNkN2NmZmMtZTlkNC0xMWViLWIyNWItMDI0MmFjMTEwMDA5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRlcGxveW1lbnQtY29udHJvbGxlciJ9.uobekKRLj02hr-qsNZCPNZWER2mYiZj8NCygiXz5c_r9B7AjhHkq3MlcfPe1RoU89P3hofrNWQpTY1Kikl9nspKQ_Yve2ykZidGGGoOtuaqI2h-SfNMjeqHGXM0CmQAEigrmkQtDrkwt-lz3pXEacRbh5pmS8BOWvpgfc5e8qt39YebFTKygNvCYvMgYMy5MsufSDeifFcb7J12hhXczxvYLDcbgDewhEnpxA887KaQlCML8soNmM36bC5qEU3nrGnKg1a5kvjjxaxres7VRb4U9fs0xogLAzcAsuOd40Re7QY8wNdY7dcUg2qivFSm7P-3vbiZ9Twr1kftazuR6Gg


Name:         disruption-controller-token-x8n2f
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: disruption-controller
              kubernetes.io/service-account.uid: 0cf3a17c-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkaXNydXB0aW9uLWNvbnRyb2xsZXItdG9rZW4teDhuMmYiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGlzcnVwdGlvbi1jb250cm9sbGVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMGNmM2ExN2MtZTlkNC0xMWViLWIyNWItMDI0MmFjMTEwMDA5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRpc3J1cHRpb24tY29udHJvbGxlciJ9.mu_3XUNoOGZwHlTyhKygDwAFjIWY0Uf0foIiS4vGNsnBBe5AUz3bN7gD0f7EpRZoI7KeIL-OWYGLvjtsCgC-1lYCRm9DNLmNxYnoDiNcbpCFx-xHF4E4yl4v51oJtXG1Bc-Xva-S15US673Gzv-soVAfpVKOzYQVklM1cbim__Eb_vXEZ_i0r-KD1DRNERMZWjvJ159DuiKMjd6CCzkgXCSQV1K4jS2jd2taiARKUcGNw0rQBeHh1_IN460jnMGCLM0lzbcZZAA4YKhdMfsU2AjIlsvK4k6VS5G-w4lo3PA1JW4Ve-Z-Im94Lqd67A2MSqUhWLPw1cn4PGGyWTaiJA
ca.crt:     1025 bytes


Name:         endpoint-controller-token-qbz7h
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: endpoint-controller
              kubernetes.io/service-account.uid: 0ab80371-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJlbmRwb2ludC1jb250cm9sbGVyLXRva2VuLXFiejdoIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImVuZHBvaW50LWNvbnRyb2xsZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwYWI4MDM3MS1lOWQ0LTExZWItYjI1Yi0wMjQyYWMxMTAwMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06ZW5kcG9pbnQtY29udHJvbGxlciJ9.z4jN9B3eHKiw7RTSz-UM0RqGiPXX2-7bV9d6N-6hkfPDCfOj3BwjApxqc3FoDiPeazVSNT-cmORpgAP3b5UMYYnN5wNw2R4XfysUXfY46i1clQJqZ14o9olfX1KegC1T-yZOC8MoEHMFvngem5gEfQquW6cIM6hYQ-kgWHaYRZwH00OqPg9YchBpmmm7erx5PZXmlWmZ76lobAyi-nAzShqJA5mKl0amSwzfQfd7ErZJhsHJrWynqpJi_SSTRsDL2zYY79aiNEMaXe4nXa0rlyVNZmRRiaA3_ca665S840KTMjvLfub4fShxpRib8EEnRJOAJft6kyfCn6bf2Uf1Og


Name:         expand-controller-token-q7k8m
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: expand-controller
              kubernetes.io/service-account.uid: 0fc0ecea-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJleHBhbmQtY29udHJvbGxlci10b2tlbi1xN2s4bSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJleHBhbmQtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjBmYzBlY2VhLWU5ZDQtMTFlYi1iMjViLTAyNDJhYzExMDAwOSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpleHBhbmQtY29udHJvbGxlciJ9.w0CjgPX7fKK1huV-Lt0ZRGLo5JgMiAR9yn11R0CrsIyDyPGRlwOrUoDSRxNn0t__Bq6YSjblPDAjz0hh2PToc44g4abe1aSfF2cXtps1y87Jee78jQ3rF9vGzinqPCJGmiXCZNvAunEAT88KkiaND74X3rHgPU-E0XjlNVyjlwMRUQWpA4Z7rqMTSb448L9vjFETbT4n7jdFib3iijeP8MgcKDxiZYpkKhQcsvvO-r-r6ndHRw7_eO06Yyl-nW20NALbc3CB8gl-1JLuFkB0CsVE2J8D3s8vkQ1rd0fMPo67cgqnc5jCULRgKWYoKOaGtcCc_UdxOg_P_9gS3zfhjA
ca.crt:     1025 bytes


Name:         generic-garbage-collector-token-qkjc7
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: generic-garbage-collector
              kubernetes.io/service-account.uid: 0f46f27b-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJnZW5lcmljLWdhcmJhZ2UtY29sbGVjdG9yLXRva2VuLXFramM3Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImdlbmVyaWMtZ2FyYmFnZS1jb2xsZWN0b3IiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwZjQ2ZjI3Yi1lOWQ0LTExZWItYjI1Yi0wMjQyYWMxMTAwMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06Z2VuZXJpYy1nYXJiYWdlLWNvbGxlY3RvciJ9.cVdg68sCtTFnHtZenBGyzJx5RRxPekXORa8L4et-5F5WVoAyvVLIc4g1kPEWgdrGQwAbXsgjowYEGZiZZM7VjHLTKpiHaYuYyFjPVtK-bBc4EuB4bPhX-5h0w5A5e1npDTqRHohIel9Q4Tx9bCfMGtzoP9C33Dog5kIFbNA1h45YaT5DUSIe9lnD8MbDbf6JoLLitB-jJdv9B7oscm7b-azrZpoU6ffe6KifzkZmlfbjwuQJtXDzgL4fD0wzTCfNP1Iun69u5NdejbEhWA2lS0Gt4KNgMX3wSQgL_4_htOs2__PwlH3d1F-VmZFweVJD6CSBqpttsgBEO3y1dwkM8w
ca.crt:     1025 bytes
namespace:  11 bytes


Name:         horizontal-pod-autoscaler-token-v46ss
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: horizontal-pod-autoscaler
              kubernetes.io/service-account.uid: 0d30b7d4-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJob3Jpem9udGFsLXBvZC1hdXRvc2NhbGVyLXRva2VuLXY0NnNzIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6Imhvcml6b250YWwtcG9kLWF1dG9zY2FsZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwZDMwYjdkNC1lOWQ0LTExZWItYjI1Yi0wMjQyYWMxMTAwMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06aG9yaXpvbnRhbC1wb2QtYXV0b3NjYWxlciJ9.0nwfw8R2M_qtziNugziYCoGFn-aXOHUrCWea6akqBGXBaIXVW0OCeLip3oW2WO2Il4LMHAFL_xSeqCkrIXgCO_LYGGbnQ0yS_QfLsDTDUzPOZzx4_o9COOWF-rfz_7OK8T01k5xns5_Gjxhh7sWpXv5IrrGeFhIWAS10d-CPXAOx-pt2r0aG4Vg-Pn3bPVi7DZvgED5LURx-kFSTsRRN8HX32jNmLKCcQ_mn3MRrBeOohf2tvCISqVzkMZfSN-fHiemZLXqQQlD25fC1zkLLQzDHoQn_A1VKH53Ac12xso0Z1r1tK4F38L85QLhV7QE-471kYjgJMJsSWv3SbcmnRg


Name:         job-controller-token-cmwfj
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: job-controller
              kubernetes.io/service-account.uid: 0ac95588-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJqb2ItY29udHJvbGxlci10b2tlbi1jbXdmaiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJqb2ItY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjBhYzk1NTg4LWU5ZDQtMTFlYi1iMjViLTAyNDJhYzExMDAwOSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpqb2ItY29udHJvbGxlciJ9.JFVVRYktyZFtJTwkepgJ5xFLJZXJReBN11ebo242b7LmembqxN3dw6qaNqd9gKJpaekzcUSJaCQDlHBivbW637YSfBsR2zov3US1_lVfm1jRCJ9Li8787-V8YcQmUCyM-gxbqwER15Sx3fGYhQicgDDVLuQ1JDfz3-9RTScKKTHOoFdpU0cmfB4kfgi4OaSl5hRvW33mi4psxLem-TjXV6EmYgGvfduVm6TGoewT-3uU2K7b9_rMLDLz6B85uolAJ_V0wajv8ZZjerTon3NyLFo5Uta_zt-gauEzaxGTVk4GHYIMhQ6-PTPZ2KpQVN8MI3j7kF9KJ9-nZ5gsvyEI4A
ca.crt:     1025 bytes


Name:         kube-proxy-token-ssgzc
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kube-proxy
              kubernetes.io/service-account.uid: 0cea9f15-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlLXByb3h5LXRva2VuLXNzZ3pjIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6Imt1YmUtcHJveHkiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwY2VhOWYxNS1lOWQ0LTExZWItYjI1Yi0wMjQyYWMxMTAwMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06a3ViZS1wcm94eSJ9.IzyixUsktQs4oQAQ-r1PTbtRrBrVf7elWCypxBkUHDhBjNQOZIjt0VnIjRoTdJenIOCH6tAmeMddJMWs8nYRMte_Pi-_XylesQypzAeiuE4oT8bxGZAC2H6JJV1D2OIrYaABWJzVejqakkrtsk5RQBbMMyXlkFK1_R1YP-5XmhzyNyfCaKzi9cprbQNX3IV72I9R9DUo6YwO3j0r_5cnKMfgPAk95ACrwqglYMpLWCFFRDjS7vWSW_ue65Yohvs85xBvYiG_ybkN75eegQYt-1AS6T0S-TBY2V3lyfO4k52VomAm34d1WgPjUyWUEun5ywH9ucDU7T_tGH4rzf66Kw


Name:         kubernetes-dashboard-certs
Namespace:    kube-system
Labels:       k8s-app=kubernetes-dashboard
Annotations:  
Type:         Opaque

Data
====


Name:         kubernetes-dashboard-key-holder
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
priv:  1679 bytes
pub:   459 bytes


Name:         kubernetes-dashboard-token-njndv
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kubernetes-dashboard
              kubernetes.io/service-account.uid: 41afcd0e-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi1uam5kdiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjQxYWZjZDBlLWU5ZDQtMTFlYi1iMjViLTAyNDJhYzExMDAwOSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.LirJ7cHBUyF30S6KKjKMNI27WLL2MC2gPPhJc1GovzWgTwUWVPYImNcXYoUOVUZ-ZJwqT9ukOH_0vxF2kblKW3xSmB_rHQ3cvs6daVNA5-HPeBT8kq0EhiRlFALYezgKpCgkpiXvTE6MGNjMhHPJ0C8yi8jEY6OC9Y4DZQXmPQV3B2Xpk7ZoaMXoHFHfPTDSpNdeWSUJN9JErrmVBFpWeTuGI6m9A2wInQY1zjLiCm-iH9y0AbpwYg1QKGWNvKVSwYkS4ViDA6qKEzGWQxWSbCdsCIGhjGNz_nxMoVm1yPeU4W1yn-Psk6LLiIzSpizZTl-3VfIR0dl5mNHq7N3KPg


Name:         namespace-controller-token-smw7m
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: namespace-controller
              kubernetes.io/service-account.uid: 0f20a4d0-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlci10b2tlbi1zbXc3bSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjBmMjBhNGQwLWU5ZDQtMTFlYi1iMjViLTAyNDJhYzExMDAwOSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpuYW1lc3BhY2UtY29udHJvbGxlciJ9.pQGalvSM_lh41fNKsY2ddZghUe_lJuz9Y9rmbXY0V-RjmF0BagSobz9SxucHAAOttIraamgTKksbxLEoAV509Yfxf62ybE4cW9KPggozc8iluwGLkdWYdGZxVe4midlnYcIpYTv4Zueav8f-9W2cGWyzHustTZf3R9b2A5N02uT91aZ38QWllsy_WKMVAaDLCLel71koJ_HjjDqrZ_ObO-YAtBeA9zslY0nhH-mpj0WsjW76Af4pVTotFBGdqtO9eV3Oe_H6sy6y2BAjxItGDTU9xvq7f3-taa-SB9j5XbEPJ1I8ObxcOLhYtvyCcHf2ZbxR9vVJ1CLR4ymry1mD2w


Name:         node-controller-token-9ppdt
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: node-controller
              kubernetes.io/service-account.uid: 0d0b0ff7-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJub2RlLWNvbnRyb2xsZXItdG9rZW4tOXBwZHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoibm9kZS1jb250cm9sbGVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMGQwYjBmZjctZTlkNC0xMWViLWIyNWItMDI0MmFjMTEwMDA5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOm5vZGUtY29udHJvbGxlciJ9.Hg-v6PYzmoSmC-FVhE07OVOtLSGb7eIDPeY77k9l4tlCT8wDPNSo3zF8LKlsKCehDeKVfQL6bdcG596ECF_Vh0zHjswZrZMCsOWO0sBArDLzOu6Zo2wgvbkPZHYA0X7nt5jc10W_q-Nw1Ud3WLNtW7v4iWpbeGZGz7EgCrqW5XqYxA9P8CA4jdMBmdit-zwjaJUu5jjIYRbeYQhKF8hrC00vMXyYMRZp2dzGSMAmI4yF2OO-QdKc1Mieq-w4i8pOl7gO4nYRLQCAcdY2TjCO5VbcigZ6IWzzQb120WgC2Iqd5mhSzcmMFcrT1IbzOm91bFkgwGAs1oQw8Cy8iCV9SQ


Name:         persistent-volume-binder-token-9hjlk
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: persistent-volume-binder
              kubernetes.io/service-account.uid: 0fb0da85-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJwZXJzaXN0ZW50LXZvbHVtZS1iaW5kZXItdG9rZW4tOWhqbGsiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoicGVyc2lzdGVudC12b2x1bWUtYmluZGVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMGZiMGRhODUtZTlkNC0xMWViLWIyNWItMDI0MmFjMTEwMDA5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOnBlcnNpc3RlbnQtdm9sdW1lLWJpbmRlciJ9.MbBQ48J2l7rgogx8YWgbX5ys_oyNBLBOn0xsuhO5_8jIKLwV5hup5X2JdjwTCw0uGZ0lkYgYANXnnK2Q39mlsJLMYCudaVWGZm7IumjyLuu9CkvhUmEHhYmqY2if-PAAXxaO_hKSbEpsmBEQKWGnU69wUdsEoQdBwqlHg4jZ69dXvrMsBgJ96ic_511e9B8R_GPKS1IXocTJtFcbCp-rpbk9REBGYQB-Fjh2UBpPvmeFQcfZ39yvBRMmi1LWW7sTQtwEiTErgYcIPpr7klSguSApm9un75P4tCjLPNFbIjUbf3lCqonwtEwADD1sKg2f9TSxdverIhoXX90IBJBTUA


Name:         pod-garbage-collector-token-l42r5
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: pod-garbage-collector
              kubernetes.io/service-account.uid: 0dfebd7d-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJwb2QtZ2FyYmFnZS1jb2xsZWN0b3ItdG9rZW4tbDQycjUiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoicG9kLWdhcmJhZ2UtY29sbGVjdG9yIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMGRmZWJkN2QtZTlkNC0xMWViLWIyNWItMDI0MmFjMTEwMDA5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOnBvZC1nYXJiYWdlLWNvbGxlY3RvciJ9.VaMklkb7hlu7MRkn0rE2Tlu-lJoeXQI4aB7qVGGHisp3nGVOjSodrldgQUxFnO3oJ9invV6RcYIS-yCaBfL8c0tak7yrGSWI2gjIrM-nS16PgLDYPjbZqJk9wXKIdzqt63Nt8HvQfBdi0IKmyM8eszOXywpHX9wyJFBGH5PYRdBs3HbpzNg5_GdjPO2IGbQGwZyyonb0o1_xk4GH-bAfbe3fCsHN5N1zY39DNo-qJ6xWC89Xfufgw95-_JxR_Pd7IiuUyqFN9KxqrbvdqdtML2ZS53fvpNqZ62_a9nK6xefGIgfxBXov_eqsjlBKh7R0vXyFAHpn3yTLUKZP-zHtGg


Name:         pv-protection-controller-token-qp2xr
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: pv-protection-controller
              kubernetes.io/service-account.uid: 0ed4594d-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJwdi1wcm90ZWN0aW9uLWNvbnRyb2xsZXItdG9rZW4tcXAyeHIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoicHYtcHJvdGVjdGlvbi1jb250cm9sbGVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMGVkNDU5NGQtZTlkNC0xMWViLWIyNWItMDI0MmFjMTEwMDA5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOnB2LXByb3RlY3Rpb24tY29udHJvbGxlciJ9.e52Sqc6aGtwNO4ecQRqddvzWsqkVOvj0MtMT8DmLf21sxv3W0yDDJYYTz6rzljqJPOkP7JBoNVoZL80e0NCXArmA27B4vCPTzCnSCrSSquoljMvxFmalnnJke6TQZKhplDtMy16orAM00Dr4KAUcphDP0uwO2Lsu6uSyFxiMQ53nRxzRt8fIf2HJZecHnvxA3qYKDpJ9ceZK0EgtzSBCSY8qNW9hVvAlUUjCkoJHeSzBeIrJTrI9GzYkkeqWfIdLy9fBpYHiCSxScKg7IVQ7iPqyZjYoo9BFKeXiIS87fD9qCh_xOAIKtmjAWmgaabEqcUx-kefbVTNKLhWn1AUyGw


Name:         pvc-protection-controller-token-b6q9s
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: pvc-protection-controller
              kubernetes.io/service-account.uid: 0ad1b174-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJwdmMtcHJvdGVjdGlvbi1jb250cm9sbGVyLXRva2VuLWI2cTlzIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6InB2Yy1wcm90ZWN0aW9uLWNvbnRyb2xsZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwYWQxYjE3NC1lOWQ0LTExZWItYjI1Yi0wMjQyYWMxMTAwMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06cHZjLXByb3RlY3Rpb24tY29udHJvbGxlciJ9.TeJQ1uCU_Jx5c7L6eZ2Is20BT9SmwvadmZdK7fF4W0jGzlohLbS-ACQQJTMszWxti9BiAxke_bijrUoKZElDlbNpgXqiqnxaIAUdJIkVPm8IQpVN7d0GfYmEeHcUl5kUHO5Oc9AFqFcQeknxcTo8RI839SXp595DHTq0SPT3gaMfSqpV3dHIlSHixBke9oG9bTYo4OoAOljfq2OT30JQSk_Z-6-_ftvCxyVtgGZGs9jyC462ic5oxQ8U4BwJoPxmTCXn_aQPXygmlONMsAo4HIRzCmv2eey44oKvGBR5cInhGaoP-SUWuc0JG1PZFDLIllU2lod-DBSdxm7err6cTQ


Name:         replicaset-controller-token-9mx8t
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: replicaset-controller
              kubernetes.io/service-account.uid: 0cf1ad30-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJyZXBsaWNhc2V0LWNvbnRyb2xsZXItdG9rZW4tOW14OHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoicmVwbGljYXNldC1jb250cm9sbGVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMGNmMWFkMzAtZTlkNC0xMWViLWIyNWItMDI0MmFjMTEwMDA5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOnJlcGxpY2FzZXQtY29udHJvbGxlciJ9.c5PbPe1CGfhMX1LzxQcQKPm3bNIFjwf9HXtSHs-wzF3caLtwNqiqczQid6F2Nf2EWbgG-ZarSJ1ESMdoPEvNprF-prVNXxeoTAuX4C9re-0zcCuZTGpxRvLq_IPZRJUHj12d3Lr8JAXdgJFBRvwQZhgDe0Tyinh5JJnNwlWvZ07mMBOVbAgcq2Ut_oAwu9DCiNv-gZHqBI-BISH6a7uF6YIdBOKIJxkTs3D3aaPYtyw-a0qBp2bAlRa3Un0IgjDUe9eiCGAc1cYRYuWUV-Ro2zrOBxwvVCh6r7iIWIuHb7y8Lc9NkbTXnxOSLwOyaCCkuioiSa_LytFlhCJlC09ubA


Name:         replication-controller-token-99sjd
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: replication-controller
              kubernetes.io/service-account.uid: 0efa8a2d-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJyZXBsaWNhdGlvbi1jb250cm9sbGVyLXRva2VuLTk5c2pkIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6InJlcGxpY2F0aW9uLWNvbnRyb2xsZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwZWZhOGEyZC1lOWQ0LTExZWItYjI1Yi0wMjQyYWMxMTAwMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06cmVwbGljYXRpb24tY29udHJvbGxlciJ9.d157wtS1_qaJdrOFJsmZR37qe7XwvIypzUOJHU4UZpvynL2Na2Wim1xH9-n5AlS-fO_VdwrkBSG7Ef8XV_sziJ9jJmLU2vXo_ZH6CVohviJyxkAob465QCVjjuvKwCgxS7vYCpJ78Y5Vdr7PDFu28X_kd3tNAIuhPMlhj5aeL5NWUmsRSo5aXl4DhoossqF81GkcmKe-kuMSnm9BOFNxokTDA2zFWbOtT6oK7ZSz2oKVN8YoIu5534nmH7-ydokDrcNgUnqy9ByTA8BAZp7ueMlIc5xktJlvt6sQ8a3gNg90j_3H2RCA1wQZJcIaB8IVVXe2bt0ZaQI9wrYBBb7vEg


Name:         resourcequota-controller-token-hdch8
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: resourcequota-controller
              kubernetes.io/service-account.uid: 0e881d99-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJyZXNvdXJjZXF1b3RhLWNvbnRyb2xsZXItdG9rZW4taGRjaDgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoicmVzb3VyY2VxdW90YS1jb250cm9sbGVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMGU4ODFkOTktZTlkNC0xMWViLWIyNWItMDI0MmFjMTEwMDA5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOnJlc291cmNlcXVvdGEtY29udHJvbGxlciJ9.omLEn20jTg0sn6P8Gihen5l3IuxFNIxK79wv_q00aOsGsAK1BbFQlIOWKHqaO81yvS3T_7UXSyCr01HTVhXabsJPS82t1Gr7jzzN4AKM230nHW_VGRjgU2gN4GxgAYByOTXBootTINz37J9Kmz0HSdTkIuqylXR6pUVrLKYJU2YqPqWe1fcoOi8P66d5qmpXfj806hw2SJgvxrf36v9cGMeoK5JDa1rI7VFAVLR-kWy9LgLQpVzDfuNqHAznay1wBcT3_Kb6l4JOkd_QdNZRl0xsP1NZMVaPqrbsxxO5us3_idNtZvo9fSsQU9itxodtnqBeSSlnvZH1Kr4tVa-7eg


Name:         service-account-controller-token-n268k
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: service-account-controller
              kubernetes.io/service-account.uid: 0ad49e7f-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJzZXJ2aWNlLWFjY291bnQtY29udHJvbGxlci10b2tlbi1uMjY4ayIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJzZXJ2aWNlLWFjY291bnQtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjBhZDQ5ZTdmLWU5ZDQtMTFlYi1iMjViLTAyNDJhYzExMDAwOSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpzZXJ2aWNlLWFjY291bnQtY29udHJvbGxlciJ9.gVr-OJa0nVuUTFjsUwdAeiQQHEFVztiXUQfV7AgRmi850C1QiwpiBAejT8wOK6lwhOrhzT5EQ77ITqmTQZz1BC_PfZRgdNdZz6Ytw01TW4AeWe3A723yiqFDnasmFB7XhbKDQxFLd0dw5VjlDsdaEu0IIU0rdQKul8EUiccJolP2s-5V4FvvPfcPqnuFTlbbTiv509DE9G1ZTcZ4TxcBx4-0aiIe_vdUjuE2ECaOIPry0rN1yg3JNA5mJhOM6hAfmaz-DzH7DNh134RiWdHxQxQqeoRrlby2aS7IayT_JSiSZecoYGvL-CFteJdzMEWqWzIzMuDb9uzHJ1CBzOezbQ


Name:         service-controller-token-dgqch
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: service-controller
              kubernetes.io/service-account.uid: 0ce95734-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJzZXJ2aWNlLWNvbnRyb2xsZXItdG9rZW4tZGdxY2giLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoic2VydmljZS1jb250cm9sbGVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMGNlOTU3MzQtZTlkNC0xMWViLWIyNWItMDI0MmFjMTEwMDA5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOnNlcnZpY2UtY29udHJvbGxlciJ9.b7Y5beL-iAl64nYe_gTiD8ogwl5GTLpeE9GBxaEMKy3KbLiags3p-0ZqXLoDD_e91BaPxpmUyRfavWV5lZ85uzzBdH-q4aD6FnwHBmoTwbw3TK1znxCeri9xObQNmxDPH7CjVQLVHbksMFGum1L1xWCf3dw0p2ZrPbgzRzq93uUYjiW-w2H40Ub0q9TFTp9ic-T6wRq4DqT8XgBKa__nNblCHiM8hlZ6ufHFYeE4aAFcLLc1RaKvFxH7oO10AJ0fB45NwaMPa1iC0O5dTl57jQbh9mxusLtKAysAMMugObnCJ9yYSA65n9p9OsYTYZVp0OOCRGu7P4lO3XR3ZLr1-Q


Name:         statefulset-controller-token-j7858
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: statefulset-controller
              kubernetes.io/service-account.uid: 0ce28a94-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJzdGF0ZWZ1bHNldC1jb250cm9sbGVyLXRva2VuLWo3ODU4Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6InN0YXRlZnVsc2V0LWNvbnRyb2xsZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwY2UyOGE5NC1lOWQ0LTExZWItYjI1Yi0wMjQyYWMxMTAwMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06c3RhdGVmdWxzZXQtY29udHJvbGxlciJ9.ju-pOwheRoeE61w_NiQ8jLXFFhIb0SA6uaOFcKR5SmaN4bhQYIdlK0IywWQwu6RzCzIgCr4XrGrWsxblNpBMQC2-dNIuELWIMAFR5gjKQZh4v2Zmh4ysXGhbj7repAOLYvcseN-tGW3hax4IywN0GI5Pw257xoHPXZToA9lnqnIPiVdzlSOe8WBfhv6omdeGmtrgNGkgAjh2LiXViwPW7V3DVPjIY5d3k0v3pqzHSuxNYTRdjqcg53TMNTUFe2LKTVbzmgLngdKufOPmtabtnxU21r1ua887BCxl9RO7FUm01CTBlofKF5wkaGr5_w6zXRVgLE6Fo4YrhILf1z1BaQ


Name:         token-cleaner-token-9fvhw
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: token-cleaner
              kubernetes.io/service-account.uid: 0eb7ce45-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJ0b2tlbi1jbGVhbmVyLXRva2VuLTlmdmh3Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6InRva2VuLWNsZWFuZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwZWI3Y2U0NS1lOWQ0LTExZWItYjI1Yi0wMjQyYWMxMTAwMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06dG9rZW4tY2xlYW5lciJ9.QDNslXVqy5OPVb2mKpgsBDpWbTIaSgaZtjnvwwlsxyARYyLDuBHHpik2_IcmVI94riv3LF-WKI2m3fw9uAlkQRmebQCNCDYKWiJEttvEXIgb6vcwEIO3Bp2Q-nbwPjzxvGHmxluidzKZ4EIusyxqsxoMCPevMmsYHQWhegIvezE8tL_7oUPIx_rGW1lqB6ohZwjSPHTXvnmzqvP1zQDmwPMDN4Neqy3W-ahOAmBdlGx1cnPtEWY7z4cN7oMqY_l4CXwbZbq54Dh6M9ODZmiwd5TarLGv2fqaNpiG5YZwtlnRz9cAIC8GfGz_OAsj9bMLkt3yC_SHHdXeQ3UYQijYkA


Name:         ttl-controller-token-wpd5f
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: ttl-controller
              kubernetes.io/service-account.uid: 0d9b87be-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJ0dGwtY29udHJvbGxlci10b2tlbi13cGQ1ZiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJ0dGwtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjBkOWI4N2JlLWU5ZDQtMTFlYi1iMjViLTAyNDJhYzExMDAwOSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTp0dGwtY29udHJvbGxlciJ9.jSUdoCKEBQYkAL5GtVQWCzFmH4FQ6GaenbDm9Cu5psOGhAd7_fpPejjmTinedpCmv0KgYIfOpzWyjQ-b1lJciZ9OFBapVDh--51kYR-enbhXpa5EP8T5O0PcnzXRojRZPMGPgQBWbn4JgTrfplzLRlrjFInTJVXA08a6B5rYXXmdABMCD08eaba9M4XeFuCIYoStmNhYw-fBPbtBegTif2ToT47-EojkMJnL8qI-oOL1nAbFHwaHSPC-czpf7VgSRBpnfTzbGrtrRwUX3vbYrrSTGo2RK87P9Qqb7HPWLnM1LDKHlgwkNmUpp9Dt8Tmun8XYWOpfQwaAqn6zRpbsPA


Name:         weave-net-token-fn28h
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: weave-net
              kubernetes.io/service-account.uid: 0d455a77-e9d4-11eb-b25b-0242ac110009

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJ3ZWF2ZS1uZXQtdG9rZW4tZm4yOGgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoid2VhdmUtbmV0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMGQ0NTVhNzctZTlkNC0xMWViLWIyNWItMDI0MmFjMTEwMDA5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOndlYXZlLW5ldCJ9.PpqFn31z-GiFAdE-Nd9-R8xuW8ESiisX53Vn_niAvFQ2DPII8q4150iNJDedLZ6nZiXBgV4e7b3NcZzNGhwRVAUyRgLrIqjdgNk-04YJ40bdP4d_5hSUylEn_QH1dgON6tMV9cK6skC0weFZzpPptGhA0bhyFdNl3DvGKGGmMgfyQsf3tEjZjEyqOuPLPtW0TPF2h98UmSsBzWbn-tCM_GEZTVY-3uxzfn2AfkuSI-e9ri1W4X8_W1z_cFIXk4Pwg0q7RWcJi44BCBJE079ezRLIWbowjXOXpqrZwe_jw_F-fJI_7ddxWDx68-wBwhjVIdNqnnMvzB-RKfgaIcWEiw
```

部署仪表板时，它使用 externalIPs 将服务绑定到端口 8443。这使得仪表板可供集群外部使用，并可在 https://2886795309-8443-elsy05.environments.katacoda.com/ 中查看

使用 *admin-user* 令牌访问仪表板。

对于生产环境，建议使用 `kubectl proxy` 来访问仪表板，而不是 externalIP。在 https://github.com/kubernetes/dashboard 上查看更多详细信息。
