---
title: Kubernetes初探（六）
description: 部署用户手册应用
date: 2021-07-20T13:36:00+0800
lastmod: '2021-07-23T11:39:00+0800'
image: kubernates.png
slug: k8s-basic-6
categories:
    - k8s-basic
    - katacoda
tags:
    - K8S
    - 容器
    - Katacoda
    - YAML
---

> Katacoda在线课：[Networking Introduction](https://www.katacoda.com/courses/kubernetes/networking-introduction)
> 
> 本系列教程希望能通过交互式学习网站与传统方式结合，更高效、容易的学习知识。
> 本系列教程将使用 [Katacoda在线学习平台](https://www.katacoda.com) 完成学习。

*Kubernetes* 具有先进的网络功能，允许 *Pod* 和 *Service* 在集群网络内部和外部进行通信。

在此场景中，您将学习以下类型的 *Kubernetes* *Service*。

- Cluster IP
- Target Ports
- NodePort
- External IPs
- Load Balancer

*Kubernetes* 服务是一个抽象，它定义了如何访问一组 *Pod* 的策略和方法。可以通过 *Service* 访问基于标签选择器的 *Pod* 集合。


## Cluster IP

*Cluster IP* 是创建 *Kubernetes* 服务时的默认方法。该服务被分配了一个内部 IP，其他组件可以使用它来访问 *Pod*。

*Service* 能够通过单个 *IP* 地址在多个 *Pod* 之间进行负载平衡。

通过 `kubectl apply -f clusterip.yaml` 命令部署服务。

在 *cat clusterip.yaml* 可以查看相关定义。

**clusterip.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp1-clusterip-svc
  labels:
    app: webapp1-clusterip
spec:
  ports:
  - port: 80
  selector:
    app: webapp1-clusterip
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-clusterip-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-clusterip
    spec:
      containers:
      - name: webapp1-clusterip-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
```

```bash
controlplane $ kubectl apply -f clusterip.yaml
service/webapp1-clusterip-svc created
deployment.extensions/webapp1-clusterip-deployment created
```

该文件定义了部署具有两个副本的 *Web* 应用程序，用来展示负载平衡和服务。
可以通过 `kubectl get pods` 命令查看 *Pod* 状态。

```bash
controlplane $ kubectl get pods
NAME                                            READY   STATUS    RESTARTS   AGE
webapp1-clusterip-deployment-669c7c65c4-r6k2l   1/1     Running   0          15s
webapp1-clusterip-deployment-669c7c65c4-xdsd4   1/1     Running   0          15s
```

同时也部署了一个 *Service*，可以通过 `kubectl get svc` 命令查看。

```bash
NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes              ClusterIP   10.96.0.1       <none>        443/TCP   8m12s
webapp1-clusterip-svc   ClusterIP   10.108.32.169   <none>        80/TCP    39s
```

可以通过 `kubectl describe svc/webapp1-clusterip-svc` 命令查看有关服务配置和活动端点 (*Pod*) 的更多详细信息

```bash
controlplane $ kubectl describe svc/webapp1-clusterip-svc
Name:              webapp1-clusterip-svc
Namespace:         default
Labels:            app=webapp1-clusterip
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"webapp1-clusterip"},"name":"webapp1-clusterip-svc","name...
Selector:          app=webapp1-clusterip
Type:              ClusterIP
IP:                10.108.32.169
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.32.0.5:80,10.32.0.6:80
Session Affinity:  None
Events:            <none>
```

部署完成后，可以通过分配的 *Cluster IP* 访问该服务。

```bash
controlplane $ export CLUSTER_IP=$(kubectl get services/webapp1-clusterip-svc -o go-template='{{(index .spec.clusterIP)}}')
controlplane $ echo CLUSTER_IP=$CLUSTER_IP
CLUSTER_IP=10.108.32.169
controlplane $ curl $CLUSTER_IP:80
<h1>This request was processed by host: webapp1-clusterip-deployment-669c7c65c4-r6k2l</h1>
```

将通过多个请求来展示基于公共标签选择器的跨多个* Pod* 的负载均衡器。

```bash
controlplane $ curl $CLUSTER_IP:80
<h1>This request was processed by host: webapp1-clusterip-deployment-669c7c65c4-xdsd4</h1>
controlplane $ curl $CLUSTER_IP:80
<h1>This request was processed by host: webapp1-clusterip-deployment-669c7c65c4-r6k2l</h1>
```

## Target Port

*TargetPort* 允许我们将服务可用的端口与应用程序正在侦听的端口分开。 *TargetPort* 是应用配置为侦听的端口。端口是从外部访问应用的方式。

与之前类似，*Service* 和额外的 *Pod* 是通过 `kubectl apply -f clusterip-target.yaml` 命令来部署的。

**clusterip-target.yaml **
```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp1-clusterip-targetport-svc
  labels:
    app: webapp1-clusterip-targetport
spec:
  ports:
  - port: 8080
    targetPort: 80
  selector:
    app: webapp1-clusterip-targetport
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-clusterip-targetport-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-clusterip-targetport
    spec:
      containers:
      - name: webapp1-clusterip-targetport-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
```

以下命令将创建服务。

```bash
controlplane $ kubectl apply -f clusterip-target.yaml
service/webapp1-clusterip-targetport-svc created
deployment.extensions/webapp1-clusterip-targetport-deployment created
```

```bash
controlplane $ kubectl get svc
NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes                         ClusterIP   10.96.0.1       <none>        443/TCP    13m
webapp1-clusterip-svc              ClusterIP   10.108.32.169   <none>        80/TCP     5m29s
webapp1-clusterip-targetport-svc   ClusterIP   10.102.59.206   <none>        8080/TCP   40s
```

```bash
controlplane $ kubectl describe svc/webapp1-clusterip-targetport-svc
Name:              webapp1-clusterip-targetport-svc
Namespace:         default
Labels:            app=webapp1-clusterip-targetport
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"webapp1-clusterip-targetport"},"name":"webapp1-clusterip...
Selector:          app=webapp1-clusterip-targetport
Type:              ClusterIP
IP:                10.102.59.206
Port:              <unset>  8080/TCP
TargetPort:        80/TCP
Endpoints:         10.32.0.7:80,10.32.0.8:80
Session Affinity:  None
Events:            <none>
```

*Service* 和 *Pod* 部署完成后，可以像以前一样通过 *Cluster IP* 访问，但这次是在定义的端口 8080 上。

```bash
controlplane $ export CLUSTER_IP=$(kubectl get services/webapp1-clusterip-targetport-svc -o go-template='{{(index .spec.clusterIP)}}')
controlplane $ echo CLUSTER_IP=$CLUSTER_IP
CLUSTER_IP=10.102.59.206
```

```bash
controlplane $ curl $CLUSTER_IP:8080
<h1>This request was processed by host: webapp1-clusterip-targetport-deployment-5599945ff4-96fqh</h1>
controlplane $ curl $CLUSTER_IP:8080
<h1>This request was processed by host: webapp1-clusterip-targetport-deployment-5599945ff4-zwr8f</h1>
```

应用程序本身仍然配置为侦听 *80* 端口。*Kubernetes* 服务管理着两者之间的转换。

## NodePort

虽然 *TargetPort* 和 *ClusterIP* 使其可用于集群内部，但 *NodePort* 通过定义的静态端口在每个节点的 *IP* 上公开服务。无论访问集群内的哪个节点，都可以通过定义的端口号直接访问该服务。

```bash
controlplane $ kubectl apply -f nodeport.yaml
service/webapp1-nodeport-svc created
deployment.extensions/webapp1-nodeport-deployment created
```

查看服务定义时，注意定义的附加类型和 *NodePort* 属性。

**nodeport.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp1-nodeport-svc
  labels:
    app: webapp1-nodeport
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
  selector:
    app: webapp1-nodeport
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-nodeport-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-nodeport
    spec:
      containers:
      - name: webapp1-nodeport-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
```

```bash
controlplane $ kubectl get svc
NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes                         ClusterIP   10.96.0.1       <none>        443/TCP        22m
webapp1-clusterip-svc              ClusterIP   10.108.32.169   <none>        80/TCP         14m
webapp1-clusterip-targetport-svc   ClusterIP   10.102.59.206   <none>        8080/TCP       9m44s
webapp1-nodeport-svc               NodePort    10.102.135.99   <none>        80:30080/TCP   5m52s
```

```bash
controlplane $ kubectl describe svc/webapp1-nodeport-svc
Name:                     webapp1-nodeport-svc
Namespace:                default
Labels:                   app=webapp1-nodeport
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"webapp1-nodeport"},"name":"webapp1-nodeport-svc","namesp...
Selector:                 app=webapp1-nodeport
Type:                     NodePort
IP:                       10.102.135.99
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30080/TCP
Endpoints:                10.32.0.10:80,10.32.0.9:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

可以通过 *NodePort* 定义的 *Node* 的 *IP* 地址访问到服务。

```bash
controlplane $ curl 172.17.0.31:30080
<h1>This request was processed by host: webapp1-nodeport-deployment-677bd89b96-j256b</h1>
controlplane $ curl 172.17.0.31:30080
<h1>This request was processed by host: webapp1-nodeport-deployment-677bd89b96-88qj5</h1>
```

## External IP

另一种方法是通过外部 *IP* 地址让服务在集群外可用。

将定义中的 *externalIPs* 更新为当前集群的 IP 地址

```bash
controlplane $ sed -i 's/HOSTIP/172.17.0.31/g' externalip.yaml
```
**externalip.yaml**
```
apiVersion: v1
kind: Service
metadata:
  name: webapp1-externalip-svc
  labels:
    app: webapp1-externalip
spec:
  ports:
  - port: 80
  externalIPs:
  - 172.17.0.30
  selector:
    app: webapp1-externalip
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-externalip-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-externalip
    spec:
      containers:
      - name: webapp1-externalip-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
controlplane $ cat externalip.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp1-externalip-svc
  labels:
    app: webapp1-externalip
spec:
  ports:
  - port: 80
  externalIPs:
  - 172.17.0.30
  selector:
    app: webapp1-externalip
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-externalip-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-externalip
    spec:
      containers:
      - name: webapp1-externalip-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
```

```bash
controlplane $ kubectl apply -f externalip.yaml
service/webapp1-externalip-svc created
deployment.extensions/webapp1-externalip-deployment created
```

```bash
controlplane $ kubectl get svc
NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes                         ClusterIP   10.96.0.1       <none>        443/TCP        28m
webapp1-clusterip-svc              ClusterIP   10.108.32.169   <none>        80/TCP         20m
webapp1-clusterip-targetport-svc   ClusterIP   10.102.59.206   <none>        8080/TCP       15m
webapp1-externalip-svc             ClusterIP   10.109.28.108   172.17.0.31   80/TCP         8s
webapp1-nodeport-svc               NodePort    10.102.135.99   <none>        80:30080/TCP   12m
```

```bash
controlplane $ kubectl describe svc/webapp1-externalip-svc
Name:              webapp1-externalip-svc
Namespace:         default
Labels:            app=webapp1-externalip
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"webapp1-externalip"},"name":"webapp1-externalip-svc","na...
Selector:          app=webapp1-externalip
Type:              ClusterIP
IP:                10.109.28.108
External IPs:      172.17.0.31
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.32.0.11:80,10.32.0.12:80
Session Affinity:  None
Events:            <none>
```

该服务现在绑定到主节点的 *IP* 地址和 *80*端口 。

```bash
controlplane $ curl 172.17.0.31
<h1>This request was processed by host: webapp1-externalip-deployment-6446b488f8-dxdb5</h1>
controlplane $ curl 172.17.0.31
<h1>This request was processed by host: webapp1-externalip-deployment-6446b488f8-2zw2p</h1>
```

## Load Balancer

在例如 *EC2* 或 *Azure* 的云服务中运行时，可以通过云服务商配置和分配发布的公网 *IP* 地址。这将通过负载均衡器（例如 *ELB*）发出，允许将额外的公网 *IP* 地址分配给 *Kubernetes* 集群，而无需直接与云提供商交互。

由于 *Katacoda* 不是云服务商，因此仍然可以为 *LoadBalancer* 类型的服务动态分配 *IP* 地址。这是通过使用 `kubectl apply -f cloudprovider.yaml` 部署 *Cloud Provider* 来完成的。并不是必需在云服务商提供的服务中运行的。

**cloudprovider.yaml**
```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: kube-keepalived-vip
  namespace: kube-system
spec:
  template:
    metadata:
      labels:
        name: kube-keepalived-vip
    spec:
      hostNetwork: true
      containers:
        - image: gcr.io/google_containers/kube-keepalived-vip:0.9
          name: kube-keepalived-vip
          imagePullPolicy: Always
          securityContext:
            privileged: true
          volumeMounts:
            - mountPath: /lib/modules
              name: modules
              readOnly: true
            - mountPath: /dev
              name: dev
          # use downward API
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          # to use unicast
          args:
          - --services-configmap=kube-system/vip-configmap
          # unicast uses the ip of the nodes instead of multicast
          # this is useful if running in cloud providers (like AWS)
          #- --use-unicast=true
      volumes:
        - name: modules
          hostPath:
            path: /lib/modules
        - name: dev
          hostPath:
            path: /dev
      nodeSelector:
        # type: worker # adjust this to match your worker nodes
---
## We also create an empty ConfigMap to hold our config
apiVersion: v1
kind: ConfigMap
metadata:
  name: vip-configmap
  namespace: kube-system
data:
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  labels:
    app: keepalived-cloud-provider
  name: keepalived-cloud-provider
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      app: keepalived-cloud-provider
  strategy:
    type: RollingUpdate
  template:
    metadata:
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ""
        scheduler.alpha.kubernetes.io/tolerations: '[{"key":"CriticalAddonsOnly", "operator":"Exists"}]'
      labels:
        app: keepalived-cloud-provider
    spec:
      containers:
      - name: keepalived-cloud-provider
        image: quay.io/munnerz/keepalived-cloud-provider:0.0.1
        imagePullPolicy: IfNotPresent
        env:
        - name: KEEPALIVED_NAMESPACE
          value: kube-system
        - name: KEEPALIVED_CONFIG_MAP
          value: vip-configmap
        - name: KEEPALIVED_SERVICE_CIDR
          value: 10.10.0.0/26 # pick a CIDR that is explicitly reserved for keepalived
        volumeMounts:
        - name: certs
          mountPath: /etc/ssl/certs
        resources:
          requests:
            cpu: 200m
        livenessProbe:
          httpGet:
            path: /healthz
            port: 10252
            host: 127.0.0.1
          initialDelaySeconds: 15
          timeoutSeconds: 15
          failureThreshold: 8
      volumes:
      - name: certs
        hostPath:
          path: /etc/ssl/certs
---
```

```bash
controlplane $ kubectl apply -f cloudprovider.yaml
daemonset.extensions/kube-keepalived-vip configured
configmap/vip-configmap configured
deployment.apps/keepalived-cloud-provider created
```

当服务请求负载均衡时，负载均衡的提供商将从配置中定义的 10.10.0.0/26 范围中分配一个 *IP*。

```bash
controlplane $ kubectl get pods -n kube-system
NAME                                        READY   STATUS    RESTARTS   AGE
coredns-fb8b8dccf-6rvjq                     1/1     Running   0          39m
coredns-fb8b8dccf-p5lxm                     1/1     Running   0          39m
etcd-controlplane                           1/1     Running   0          38m
katacoda-cloud-provider-66d7758d5d-bn748    1/1     Running   0          39m
keepalived-cloud-provider-78fc4468b-bk8wn   1/1     Running   0          6m16s
kube-apiserver-controlplane                 1/1     Running   0          39m
kube-controller-manager-controlplane        1/1     Running   2          39m
kube-keepalived-vip-42ql5                   1/1     Running   0          38m
kube-proxy-d5q4m                            1/1     Running   0          39m
kube-scheduler-controlplane                 1/1     Running   3          39m
weave-net-9rc2g                             2/2     Running   1          39m
```

该服务是通过负载均衡配置的，如 *cat loadbalancer.yaml* 中所定义所示。

**loadbalancer.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp1-loadbalancer-svc
  labels:
    app: webapp1-loadbalancer
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: webapp1-loadbalancer
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-loadbalancer-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-loadbalancer
    spec:
      containers:
      - name: webapp1-loadbalancer-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
```

```bash
controlplane $ kubectl apply -f loadbalancer.yaml
service/webapp1-loadbalancer-svc created
deployment.extensions/webapp1-loadbalancer-deployment created
```
在定义 *IP* 地址时，服务将显示 *Pending*。分配后，它将出现在服务列表中。

```bash
controlplane $ kubectl get svc
NAME                               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes                         ClusterIP      10.96.0.1       <none>        443/TCP        38m
webapp1-clusterip-svc              ClusterIP      10.108.32.169   <none>        80/TCP         31m
webapp1-clusterip-targetport-svc   ClusterIP      10.102.59.206   <none>        8080/TCP       26m
webapp1-externalip-svc             ClusterIP      10.109.28.108   172.17.0.31   80/TCP         10m
webapp1-loadbalancer-svc           LoadBalancer   10.105.60.108   172.17.0.31   80:30626/TCP   7s
webapp1-nodeport-svc               NodePort       10.102.135.99   <none>        80:30080/TCP   22m
```

```bash
controlplane $ kubectl describe svc/webapp1-loadbalancer-svc
Name:                     webapp1-loadbalancer-svc
Namespace:                default
Labels:                   app=webapp1-loadbalancer
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"webapp1-loadbalancer"},"name":"webapp1-loadbalancer-svc"...
Selector:                 app=webapp1-loadbalancer
Type:                     LoadBalancer
IP:                       10.105.60.108
LoadBalancer Ingress:     172.17.0.31
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30626/TCP
Endpoints:                10.32.0.14:80,10.32.0.15:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                Age   From                Message
  ----    ------                ----  ----                -------
  Normal  CreatingLoadBalancer  95s   service-controller  Creating load balancer
  Normal  CreatedLoadBalancer   95s   service-controller  Created load balancer
```

现在可以通过分配的 *IP* 地址访问该服务，在本例中的范围是 *10.10.0.0/26* 。

```bash
controlplane $ export LoadBalancerIP=$(kubectl get services/webapp1-loadbalancer-svc -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
controlplane $ echo LoadBalancerIP=$LoadBalancerIP
LoadBalancerIP=172.17.0.31
controlplane $ curl $LoadBalancerIP
<h1>This request was processed by host: webapp1-externalip-deployment-6446b488f8-2zw2p</h1>
controlplane $ curl $LoadBalancerIP
<h1>This request was processed by host: webapp1-externalip-deployment-6446b488f8-dxdb5</h1>
```