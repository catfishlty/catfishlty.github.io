---
title: Kubernetes初探（七）
description: 创建入口路由
date: 2021-07-20T15:30:00+0800
lastmod: '2021-07-23T13:41:00+0800'
image: kubernates.png
slug: k8s-basic-7
categories:
    - k8s-basic
    - katacoda
tags:
    - K8S
    - 容器
    - Katacoda
    - YAML
---

> Katacoda在线课：[Create Ingress Routing](https://www.katacoda.com/courses/kubernetes/create-kubernetes-ingress-routes)
> 
> 本系列教程希望能通过交互式学习网站与传统方式结合，更高效、容易的学习知识。
> 本系列教程将使用 [Katacoda在线学习平台](https://www.katacoda.com) 完成学习。

*Kubernetes* 具有先进的网络功能，允许 *Pod* 和 *Service* 在集群网络内部进行通信。 *Ingress* 开启了到集群的入站连接，允许外部流量到达正确的 *Pod*。

*Ingress* 能够提供外部可访问的 *URL*、负载平衡流量、终止 SSL、为 *Kubernetes* 集群提供基于名称的虚拟主机。

在此场景中，您将学习如何部署和配置 *Ingress* 规则来管理传入的 *HTTP* 请求。

## 创建 Deployment

首先，部署一个示例 *HTTP* 服务器，它将成为我们请求的目标。部署中包含三个部署，分别为 *webapp1*， *webapp2*， *webapp3*，每个部署都有一个服务。

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
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp2
  template:
    metadata:
      labels:
        app: webapp2
    spec:
      containers:
      - name: webapp2
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp3
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp3
  template:
    metadata:
      labels:
        app: webapp3
    spec:
      containers:
      - name: webapp3
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: webapp1-svc
  labels:
    app: webapp1
spec:
  ports:
  - port: 80
  selector:
    app: webapp1
---
apiVersion: v1
kind: Service
metadata:
  name: webapp2-svc
  labels:
    app: webapp2
spec:
  ports:
  - port: 80
  selector:
    app: webapp2
---
apiVersion: v1
kind: Service
metadata:
  name: webapp3-svc
  labels:
    app: webapp3
spec:
  ports:
  - port: 80
  selector:
    app: webapp3
```

### 任务

使用 `kubectl apply -f deployment.yaml` 命令部署 *YAML* 定义。

```bash
controlplane $ kubectl apply -f deployment.yaml
deployment.apps/webapp1 created
deployment.apps/webapp2 created
deployment.apps/webapp3 created
service/webapp1-svc created
service/webapp2-svc created
service/webapp3-svc created
```

可以用 `kubectl get deployment` 查看状态。

```bash
controlplane $ kubectl get deployment
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
webapp1   1/1     1            1           15s
webapp2   1/1     1            1           15s
webapp3   1/1     1            1           15s
```

```bash
controlplane $ kubectl get svc
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP   7m41s
webapp1-svc   ClusterIP   10.98.236.192    <none>        80/TCP    41s
webapp2-svc   ClusterIP   10.105.201.139   <none>        80/TCP    41s
webapp3-svc   ClusterIP   10.103.159.185   <none>        80/TCP    41s
```

## 部署 Ingress

*ingress.yaml* 文件中定义了一个基于 *Nginx* 的 *Ingress* 控制器以及一个 *Service* ，开放 *80* 端口用于使用 *External IPs* 的外部连接。如果 *Kubernetes* 集群在云服务商上运行，那么它将为 *LoadBalancer* 服务类型。

*ServiceAccount* 定义了带有权限的、连接集群以操作定义的入口规则的一组的帐户。默认服务器密钥是其他 *Nginx* 示例 *SSL* 连接的自签名证书，并且是[Nginx Default Example](https://github.com/nginxinc/kubernetes-ingress/tree/master/deployments) 中所必须的。

**ingress.yaml**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nginx-ingress
---
apiVersion: v1
kind: Secret
metadata:
  name: default-server-secret
  namespace: nginx-ingress
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN2akNDQWFZQ0NRREFPRjl0THNhWFhEQU5CZ2txaGtpRzl3MEJBUXNGQURBaE1SOHdIUVlEVlFRRERCWk8KUjBsT1dFbHVaM0psYzNORGIyNTBjbTlzYkdWeU1CNFhEVEU0TURreE1qRTRNRE16TlZvWERUSXpNRGt4TVRFNApNRE16TlZvd0lURWZNQjBHQTFVRUF3d1dUa2RKVGxoSmJtZHlaWE56UTI5dWRISnZiR3hsY2pDQ0FTSXdEUVlKCktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ0VCQUwvN2hIUEtFWGRMdjNyaUM3QlBrMTNpWkt5eTlyQ08KR2xZUXYyK2EzUDF0azIrS3YwVGF5aGRCbDRrcnNUcTZzZm8vWUk1Y2Vhbkw4WGM3U1pyQkVRYm9EN2REbWs1Qgo4eDZLS2xHWU5IWlg0Rm5UZ0VPaStlM2ptTFFxRlBSY1kzVnNPazFFeUZBL0JnWlJVbkNHZUtGeERSN0tQdGhyCmtqSXVuektURXUyaDU4Tlp0S21ScUJHdDEwcTNRYzhZT3ExM2FnbmovUWRjc0ZYYTJnMjB1K1lYZDdoZ3krZksKWk4vVUkxQUQ0YzZyM1lma1ZWUmVHd1lxQVp1WXN2V0RKbW1GNWRwdEMzN011cDBPRUxVTExSakZJOTZXNXIwSAo1TmdPc25NWFJNV1hYVlpiNWRxT3R0SmRtS3FhZ25TZ1JQQVpQN2MwQjFQU2FqYzZjNGZRVXpNQ0F3RUFBVEFOCkJna3Foa2lHOXcwQkFRc0ZBQU9DQVFFQWpLb2tRdGRPcEsrTzhibWVPc3lySmdJSXJycVFVY2ZOUitjb0hZVUoKdGhrYnhITFMzR3VBTWI5dm15VExPY2xxeC9aYzJPblEwMEJCLzlTb0swcitFZ1U2UlVrRWtWcitTTFA3NTdUWgozZWI4dmdPdEduMS9ienM3bzNBaS9kclkrcUI5Q2k1S3lPc3FHTG1US2xFaUtOYkcyR1ZyTWxjS0ZYQU80YTY3Cklnc1hzYktNbTQwV1U3cG9mcGltU1ZmaXFSdkV5YmN3N0NYODF6cFErUyt1eHRYK2VBZ3V0NHh3VlI5d2IyVXYKelhuZk9HbWhWNThDd1dIQnNKa0kxNXhaa2VUWXdSN0diaEFMSkZUUkk3dkhvQXprTWIzbjAxQjQyWjNrN3RXNQpJUDFmTlpIOFUvOWxiUHNoT21FRFZkdjF5ZytVRVJxbStGSis2R0oxeFJGcGZnPT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
  tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBdi91RWM4b1JkMHUvZXVJTHNFK1RYZUprckxMMnNJNGFWaEMvYjVyYy9XMlRiNHEvClJOcktGMEdYaVN1eE9ycXgrajlnamx4NXFjdnhkenRKbXNFUkJ1Z1B0ME9hVGtIekhvb3FVWmcwZGxmZ1dkT0EKUTZMNTdlT1l0Q29VOUZ4amRXdzZUVVRJVUQ4R0JsRlNjSVo0b1hFTkhzbysyR3VTTWk2Zk1wTVM3YUhudzFtMApxWkdvRWEzWFNyZEJ6eGc2clhkcUNlUDlCMXl3VmRyYURiUzc1aGQzdUdETDU4cGszOVFqVUFQaHpxdmRoK1JWClZGNGJCaW9CbTVpeTlZTW1hWVhsMm0wTGZzeTZuUTRRdFFzdEdNVWozcGJtdlFmazJBNnljeGRFeFpkZFZsdmwKMm82MjBsMllxcHFDZEtCRThCay90elFIVTlKcU56cHpoOUJUTXdJREFRQUJBb0lCQVFDZklHbXowOHhRVmorNwpLZnZJUXQwQ0YzR2MxNld6eDhVNml4MHg4Mm15d1kxUUNlL3BzWE9LZlRxT1h1SENyUlp5TnUvZ2IvUUQ4bUFOCmxOMjRZTWl0TWRJODg5TEZoTkp3QU5OODJDeTczckM5bzVvUDlkazAvYzRIbjAzSkVYNzZ5QjgzQm9rR1FvYksKMjhMNk0rdHUzUmFqNjd6Vmc2d2szaEhrU0pXSzBwV1YrSjdrUkRWYmhDYUZhNk5nMUZNRWxhTlozVDhhUUtyQgpDUDNDeEFTdjYxWTk5TEI4KzNXWVFIK3NYaTVGM01pYVNBZ1BkQUk3WEh1dXFET1lvMU5PL0JoSGt1aVg2QnRtCnorNTZud2pZMy8yUytSRmNBc3JMTnIwMDJZZi9oY0IraVlDNzVWYmcydVd6WTY3TWdOTGQ5VW9RU3BDRkYrVm4KM0cyUnhybnhBb0dCQU40U3M0ZVlPU2huMVpQQjdhTUZsY0k2RHR2S2ErTGZTTXFyY2pOZjJlSEpZNnhubmxKdgpGenpGL2RiVWVTbWxSekR0WkdlcXZXaHFISy9iTjIyeWJhOU1WMDlRQ0JFTk5jNmtWajJTVHpUWkJVbEx4QzYrCk93Z0wyZHhKendWelU0VC84ajdHalRUN05BZVpFS2FvRHFyRG5BYWkyaW5oZU1JVWZHRXFGKzJyQW9HQkFOMVAKK0tZL0lsS3RWRzRKSklQNzBjUis3RmpyeXJpY05iWCtQVzUvOXFHaWxnY2grZ3l4b25BWlBpd2NpeDN3QVpGdwpaZC96ZFB2aTBkWEppc1BSZjRMazg5b2pCUmpiRmRmc2l5UmJYbyt3TFU4NUhRU2NGMnN5aUFPaTVBRHdVU0FkCm45YWFweUNweEFkREtERHdObit3ZFhtaTZ0OHRpSFRkK3RoVDhkaVpBb0dCQUt6Wis1bG9OOTBtYlF4VVh5YUwKMjFSUm9tMGJjcndsTmVCaWNFSmlzaEhYa2xpSVVxZ3hSZklNM2hhUVRUcklKZENFaHFsV01aV0xPb2I2NTNyZgo3aFlMSXM1ZUtka3o0aFRVdnpldm9TMHVXcm9CV2xOVHlGanIrSWhKZnZUc0hpOGdsU3FkbXgySkJhZUFVWUNXCndNdlQ4NmNLclNyNkQrZG8wS05FZzFsL0FvR0FlMkFVdHVFbFNqLzBmRzgrV3hHc1RFV1JqclRNUzRSUjhRWXQKeXdjdFA4aDZxTGxKUTRCWGxQU05rMXZLTmtOUkxIb2pZT2pCQTViYjhibXNVU1BlV09NNENoaFJ4QnlHbmR2eAphYkJDRkFwY0IvbEg4d1R0alVZYlN5T294ZGt5OEp0ek90ajJhS0FiZHd6NlArWDZDODhjZmxYVFo5MWpYL3RMCjF3TmRKS2tDZ1lCbyt0UzB5TzJ2SWFmK2UwSkN5TGhzVDQ5cTN3Zis2QWVqWGx2WDJ1VnRYejN5QTZnbXo5aCsKcDNlK2JMRUxwb3B0WFhNdUFRR0xhUkcrYlNNcjR5dERYbE5ZSndUeThXczNKY3dlSTdqZVp2b0ZpbmNvVlVIMwphdmxoTUVCRGYxSjltSDB5cDBwWUNaS2ROdHNvZEZtQktzVEtQMjJhTmtsVVhCS3gyZzR6cFE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress 
  namespace: nginx-ingress
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-config
  namespace: nginx-ingress
data:
---
# Described at: https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/
# Source from: https://github.com/nginxinc/kubernetes-ingress/blob/master/deployments/common/ingress-class.yaml
apiVersion: networking.k8s.io/v1beta1
kind: IngressClass
metadata:
  name: nginx
  # annotations:
  #   ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: nginx.org/ingress-controller
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress
  namespace: nginx-ingress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-ingress
  template:
    metadata:
      labels:
        app: nginx-ingress
    spec:
      serviceAccountName: nginx-ingress
      containers:
      - image: nginx/nginx-ingress:edge
        imagePullPolicy: Always
        name: nginx-ingress
        ports:
        - name: http
          containerPort: 80
        - name: https
          containerPort: 443
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        args:
          - -nginx-configmaps=$(POD_NAMESPACE)/nginx-config
          - -default-server-tls-secret=$(POD_NAMESPACE)/default-server-secret
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
  namespace: nginx-ingress
spec:
  type: NodePort 
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    app: nginx-ingress
  externalIPs:
    - 172.17.0.46
```

### 任务

*Ingress* 控制器以同样的方式部署到 *Kubernetes* 集群，使用 `kubectl create -f ingress.yaml`命令操作。

```bash
controlplane $ kubectl create -f ingress.yaml
namespace/nginx-ingress created
secret/default-server-secret created
serviceaccount/nginx-ingress created
configmap/nginx-config created
ingressclass.networking.k8s.io/nginx created
deployment.apps/nginx-ingress created
service/nginx-ingress created
```

可以使用 `kubectl get deployment -n nginx-ingress` 查看状态。

```bash
controlplane $ kubectl get deployment -n nginx-ingress
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
nginx-ingress   1/1     1            1           35s
```

## 部署 Ingress Rules

*Ingress Rules* 是 Kubernetes 的对象类型。规则可以基于请求主机（域），或请求的路径，或两者的组合。

`cat ingress-rules.yaml` 中定义了一组示例规则。

**ingress-rules.yaml**
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: webapp-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: my.kubernetes.example
    http:
      paths:
      - path: /webapp1
        backend:
          serviceName: webapp1-svc
          servicePort: 80
      - path: /webapp2
        backend:
          serviceName: webapp2-svc
          servicePort: 80
      - backend:
          serviceName: webapp3-svc
          servicePort: 80
```

规则的重要部分定义如下。

这些规则适用于对主机 *my.kubernetes.example* 的请求。基于路径请求定义了两个规则，并使用一个 *catch all* 定义。对路径 */webapp1* 的请求被转发到服务 *webapp1-svc*。同样，对 */webapp2* 的请求被转发到 *webapp2-svc*。如果没有规则适用，将使用 *webapp3-svc*。

这演示了应用的 *URL* 结构如何独立于应用程序的部署方式。

### 任务

与所有 *Kubernetes 对象一样*，它们可以通过 `kubectl create -f ingress-rules.yaml` 进行部署。

```bash
controlplane $ kubectl create -f ingress-rules.yaml
ingress.extensions/webapp-ingress created
```

部署后，可以通过 `kubectl get ing` 查看所有 *Ingress* 规则的状态

```bash
controlplane $ kubectl get ing
NAME             CLASS   HOSTS                   ADDRESS   PORTS   AGE
webapp-ingress   nginx   my.kubernetes.example             80      41s
```

## 测试

应用入口规则后，流量将被路由到定义的位置。

第一个请求将由 *webapp1* 处理。

```bash
controlplane $ curl -H "Host: my.kubernetes.example" 172.17.0.68/webapp1
<h1>This request was processed by host: webapp1-7c456784b7-cg6sk</h1>
```

第二个请求将由 *webapp2* 处理。

```bash
controlplane $ curl -H "Host: my.kubernetes.example" 172.17.0.68/webapp2
<h1>This request was processed by host: webapp2-79f9947d45-25j6l</h1>
```

最后，所有的其他请求将由 *webapp3* 处理。

```bash
controlplane $ curl -H "Host: my.kubernetes.example" 172.17.0.68
<h1>This request was processed by host: webapp3-777f4dd675-n2pqs</h1>
```