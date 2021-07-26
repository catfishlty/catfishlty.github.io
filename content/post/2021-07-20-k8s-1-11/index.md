---
title: Kubernetes初探（十一）
description: Helm - K8S包管理工具 
date: 2021-07-23T14:54:00+0800
lastmod: '2021-07-23T15:24:00+0800'
image: kubernates.png
slug: k8s-basic-11
categories:
    - k8s-basic
    - katacoda
    - helm
tags:
    - K8S
    - 容器
    - Katacoda
    - YAML
    - Helm
---

> Katacoda在线课：[Helm Package Manager](https://www.katacoda.com/courses/kubernetes/helm-package-manager)
> 
> 本系列教程希望能通过交互式学习网站与传统方式结合，更高效、容易的学习知识。
> 本系列教程将使用 [Katacoda在线学习平台](https://www.katacoda.com) 完成学习。


这个场景教你如何使用 *Kubernetes* 的包管理器 *Helm* 来部署 *Redis*。 *Helm* 简化了服务发现和部署到 *Kubernetes* 集群的步骤。。

> **"Helm is the best way to find, share, and use software built for Kubernetes."**
> 
> ***Helm* 是查找、共享和使用为 *Kubernetes* 构建的软件的最佳方式**

更多细节可以前往官网：[http://www.helm.sh/](http://www.helm.sh/)

## 安装 Helm

*Helm* 是一个单独的二进制文件，用于管理将 *Charts* 部署到 *Kubernetes*。 *Chart* 是 *kubernetes* 应用的一个打包单元。*Helm* 可以从 https://github.com/kubernetes/helm/releases 下载。

```bash
controlplane $ curl -LO https://storage.googleapis.com/kubernetes-helm/helm-v2.8.2-linux-amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 14.2M  100 14.2M    0     0  19.4M      0 --:--:-- --:--:-- --:--:-- 19.4M
controlplane $ tar -xvf helm-v2.8.2-linux-amd64.tar.gz
linux-amd64/
linux-amd64/helm
linux-amd64/README.md
linux-amd64/LICENSE
controlplane $ mv linux-amd64/helm /usr/local/bin/
```

安装后，初始化更新本地缓存将最新的包与本地安装环境同步。

```bash
controlplane $ helm init --stable-repo-url https://charts.helm.sh/stable
Creating /root/.helm 
Creating /root/.helm/repository 
Creating /root/.helm/repository/cache 
Creating /root/.helm/repository/local 
Creating /root/.helm/plugins 
Creating /root/.helm/starters 
Creating /root/.helm/cache/archive 
Creating /root/.helm/repository/repositories.yaml 
Adding stable repo with URL: https://charts.helm.sh/stable 
Adding local repo with URL: http://127.0.0.1:8879/charts 
$HELM_HOME has been configured at /root/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
controlplane $ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

## 搜索 Chart

您现在可以开始部署软件。可以使用搜索命令查找可用图表 *Chart* 。

例如，要部署 Redis，我们需要找到一个 *Redis* 的 *Chart* 。

```bash
controlplane $ helm search redis
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION                                       
stable/prometheus-redis-exporter        3.5.1           1.3.4           DEPRECATED Prometheus exporter for Redis metrics  
stable/redis                            10.5.7          5.0.7           DEPRECATED Open source, advanced key-value stor...
stable/redis-ha                         4.4.6           5.0.6           DEPRECATED - Highly available Kubernetes implem...
stable/sensu                            0.2.5           0.28            DEPRECATED Sensu monitoring framework backed by..
```

通过 *inspect* 命令，我们可以查看更多关于 *stable/redis* 的信息，信息量很大。

```bash
controlplane $ helm inspect stable/redis
apiVersion: v1
appVersion: 5.0.7
deprecated: true
description: DEPRECATED Open source, advanced key-value store. It is often referred
  to as a data structure server since keys can contain strings, hashes, lists, sets
  and sorted sets.
engine: gotpl
home: http://redis.io/
icon: https://bitnami.com/assets/stacks/redis/img/redis-stack-220x234.png
keywords:
- redis
- keyvalue
- database
name: redis
sources:
- https://github.com/bitnami/bitnami-docker-redis
version: 10.5.7

---
## Global Docker image parameters
## Please, note that this will override the image parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry and imagePullSecrets
##
global:
#   imageRegistry: myRegistryName
#   imagePullSecrets:
#     - myRegistryKeySecretName
#   storageClass: myStorageClass
  redis: {}

## Bitnami Redis image version
## ref: https://hub.docker.com/r/bitnami/redis/tags/
##
image:
  registry: docker.io
  repository: bitnami/redis
  ## Bitnami Redis image tag
  ## ref: https://github.com/bitnami/bitnami-docker-redis#supported-tags-and-respective-dockerfile-links
  ##
  tag: 5.0.7-debian-10-r32
  ## Specify a imagePullPolicy
  ## Defaults to 'Always' if image tag is 'latest', else set to 'IfNotPresent'
  ## ref: http://kubernetes.io/docs/user-guide/images/#pre-pulling-images
  ##
  pullPolicy: IfNotPresent
  ## Optionally specify an array of imagePullSecrets.
  ## Secrets must be manually created in the namespace.
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  ##
  # pullSecrets:
  #   - myRegistryKeySecretName

## String to partially override redis.fullname template (will maintain the release name)
##
# nameOverride:

## String to fully override redis.fullname template
##
# fullnameOverride:

## Cluster settings
cluster:
  enabled: true
  slaveCount: 2

## Use redis sentinel in the redis pod. This will disable the master and slave services and
## create one redis service with ports to the sentinel and the redis instances
sentinel:
  enabled: false
  ## Require password authentication on the sentinel itself
  ## ref: https://redis.io/topics/sentinel
  usePassword: true
  ## Bitnami Redis Sentintel image version
  ## ref: https://hub.docker.com/r/bitnami/redis-sentinel/tags/
  ##
  image:
    registry: docker.io
    repository: bitnami/redis-sentinel
    ## Bitnami Redis image tag
    ## ref: https://github.com/bitnami/bitnami-docker-redis-sentinel#supported-tags-and-respective-dockerfile-links
    ##
    tag: 5.0.7-debian-10-r27
    ## Specify a imagePullPolicy
    ## Defaults to 'Always' if image tag is 'latest', else set to 'IfNotPresent'
    ## ref: http://kubernetes.io/docs/user-guide/images/#pre-pulling-images
    ##
    pullPolicy: IfNotPresent
    ## Optionally specify an array of imagePullSecrets.
    ## Secrets must be manually created in the namespace.
    ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
    ##
    # pullSecrets:
    #   - myRegistryKeySecretName
  masterSet: mymaster
  initialCheckTimeout: 5
  quorum: 2
  downAfterMilliseconds: 60000
  failoverTimeout: 18000
  parallelSyncs: 1
  port: 26379
  ## Additional Redis configuration for the sentinel nodes
  ## ref: https://redis.io/topics/config
  ##
  configmap:
  ## Enable or disable static sentinel IDs for each replicas
  ## If disabled each sentinel will generate a random id at startup
  ## If enabled, each replicas will have a constant ID on each start-up
  ##
  staticID: false
  ## Configure extra options for Redis Sentinel liveness and readiness probes
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes)
  ##
  livenessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 5
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 5
  readinessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 5
    timeoutSeconds: 1
    successThreshold: 1
    failureThreshold: 5
  ## Redis Sentinel resource requests and limits
  ## ref: http://kubernetes.io/docs/user-guide/compute-resources/
  # resources:
  #   requests:
  #     memory: 256Mi
  #     cpu: 100m
  ## Redis Sentinel Service properties
  service:
    ##  Redis Sentinel Service type
    type: ClusterIP
    sentinelPort: 26379
    redisPort: 6379

    ## Specify the nodePort value for the LoadBalancer and NodePort service types.
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
    ##
    # sentinelNodePort:
    # redisNodePort:

    ## Provide any additional annotations which may be required. This can be used to
    ## set the LoadBalancer service type to internal only.
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer
    ##
    annotations: {}
    labels: {}
    loadBalancerIP:

## Specifies the Kubernetes Cluster's Domain Name.
##
clusterDomain: cluster.local

networkPolicy:
  ## Specifies whether a NetworkPolicy should be created
  ##
  enabled: false

  ## The Policy model to apply. When set to false, only pods with the correct
  ## client label will have network access to the port Redis is listening
  ## on. When true, Redis will accept connections from any source
  ## (with the correct destination port).
  ##
  # allowExternal: true

  ## Allow connections from other namespacess. Just set label for namespace and set label for pods (optional).
  ##
  ingressNSMatchLabels: {}
  ingressNSPodMatchLabels: {}

serviceAccount:
  ## Specifies whether a ServiceAccount should be created
  ##
  create: false
  ## The name of the ServiceAccount to use.
  ## If not set and create is true, a name is generated using the fullname template
  name:

rbac:
  ## Specifies whether RBAC resources should be created
  ##
  create: false

  role:
    ## Rules to create. It follows the role specification
    # rules:
    #  - apiGroups:
    #    - extensions
    #    resources:
    #      - podsecuritypolicies
    #    verbs:
    #      - use
    #    resourceNames:
    #      - gce.unprivileged
    rules: []

## Redis pod Security Context
securityContext:
  enabled: true
  fsGroup: 1001
  runAsUser: 1001
  ## sysctl settings for master and slave pods
  ##
  ## Uncomment the setting below to increase the net.core.somaxconn value
  ##
  # sysctls:
  # - name: net.core.somaxconn
  #   value: "10000"

## Use password authentication
usePassword: true
## Redis password (both master and slave)
## Defaults to a random 10-character alphanumeric string if not set and usePassword is true
## ref: https://github.com/bitnami/bitnami-docker-redis#setting-the-server-password-on-first-run
##
password: ""
## Use existing secret (ignores previous password)
# existingSecret:
## Password key to be retrieved from Redis secret
##
# existingSecretPasswordKey:

## Mount secrets as files instead of environment variables
usePasswordFile: false

## Persist data to a persistent volume (Redis Master)
persistence: {}
  ## A manually managed Persistent Volume and Claim
  ## Requires persistence.enabled: true
  ## If defined, PVC must be created manually before volume will be bound
  # existingClaim:

# Redis port
redisPort: 6379

##
## Redis Master parameters
##
master:
  ## Redis command arguments
  ##
  ## Can be used to specify command line arguments, for example:
  ##
  command: "/run.sh"
  ## Additional Redis configuration for the master nodes
  ## ref: https://redis.io/topics/config
  ##
  configmap:
  ## Redis additional command line flags
  ##
  ## Can be used to specify command line flags, for example:
  ##
  ## extraFlags:
  ##  - "--maxmemory-policy volatile-ttl"
  ##  - "--repl-backlog-size 1024mb"
  extraFlags: []
  ## Comma-separated list of Redis commands to disable
  ##
  ## Can be used to disable Redis commands for security reasons.
  ## Commands will be completely disabled by renaming each to an empty string.
  ## ref: https://redis.io/topics/security#disabling-of-specific-commands
  ##
  disableCommands:
  - FLUSHDB
  - FLUSHALL

  ## Redis Master additional pod labels and annotations
  ## ref: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
  podLabels: {}
  podAnnotations: {}

  ## Redis Master resource requests and limits
  ## ref: http://kubernetes.io/docs/user-guide/compute-resources/
  # resources:
  #   requests:
  #     memory: 256Mi
  #     cpu: 100m
  ## Use an alternate scheduler, e.g. "stork".
  ## ref: https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
  ##
  # schedulerName:

  ## Configure extra options for Redis Master liveness and readiness probes
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes)
  ##
  livenessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 5
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 5
  readinessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 5
    timeoutSeconds: 1
    successThreshold: 1
    failureThreshold: 5

  ## Redis Master Node selectors and tolerations for pod assignment
  ## ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector
  ## ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#taints-and-tolerations-beta-feature
  ##
  # nodeSelector: {"beta.kubernetes.io/arch": "amd64"}
  # tolerations: []
  ## Redis Master pod/node affinity/anti-affinity
  ##
  affinity: {}

  ## Redis Master Service properties
  service:
    ##  Redis Master Service type
    type: ClusterIP
    port: 6379

    ## Specify the nodePort value for the LoadBalancer and NodePort service types.
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
    ##
    # nodePort:

    ## Provide any additional annotations which may be required. This can be used to
    ## set the LoadBalancer service type to internal only.
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer
    ##
    annotations: {}
    labels: {}
    loadBalancerIP:
    # loadBalancerSourceRanges: ["10.0.0.0/8"]

  ## Enable persistence using Persistent Volume Claims
  ## ref: http://kubernetes.io/docs/user-guide/persistent-volumes/
  ##
  persistence:
    enabled: true
    ## The path the volume will be mounted at, useful when using different
    ## Redis images.
    path: /data
    ## The subdirectory of the volume to mount to, useful in dev environments
    ## and one PV for multiple services.
    subPath: ""
    ## redis data Persistent Volume Storage Class
    ## If defined, storageClassName: <storageClass>
    ## If set to "-", storageClassName: "", which disables dynamic provisioning
    ## If undefined (the default) or set to null, no storageClassName spec is
    ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
    ##   GKE, AWS & OpenStack)
    ##
    # storageClass: "-"
    accessModes:
    - ReadWriteOnce
    size: 8Gi
    ## Persistent Volume selectors
    ## https://kubernetes.io/docs/concepts/storage/persistent-volumes/#selector
    matchLabels: {}
    matchExpressions: {}

  ## Update strategy, can be set to RollingUpdate or onDelete by default.
  ## https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/#updating-statefulsets
  statefulset:
    updateStrategy: RollingUpdate
    ## Partition update strategy
    ## https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#partitions
    # rollingUpdatePartition:

  ## Redis Master pod priorityClassName
  # priorityClassName: {}

##
## Redis Slave properties
## Note: service.type is a mandatory parameter
## The rest of the parameters are either optional or, if undefined, will inherit those declared in Redis Master
##
slave:
  ## Slave Service properties
  service:
    ## Redis Slave Service type
    type: ClusterIP
    ## Redis port
    port: 6379
    ## Specify the nodePort value for the LoadBalancer and NodePort service types.
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
    ##
    # nodePort:

    ## Provide any additional annotations which may be required. This can be used to
    ## set the LoadBalancer service type to internal only.
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer
    ##
    annotations: {}
    labels: {}
    loadBalancerIP:
    # loadBalancerSourceRanges: ["10.0.0.0/8"]

  ## Redis slave port
  port: 6379
  ## Can be used to specify command line arguments, for example:
  ##
  command: "/run.sh"
  ## Additional Redis configuration for the slave nodes
  ## ref: https://redis.io/topics/config
  ##
  configmap:
  ## Redis extra flags
  extraFlags: []
  ## List of Redis commands to disable
  disableCommands:
  - FLUSHDB
  - FLUSHALL

  ## Redis Slave pod/node affinity/anti-affinity
  ##
  affinity: {}

  ## Configure extra options for Redis Slave liveness and readiness probes
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes)
  ##
  livenessProbe:
    enabled: true
    initialDelaySeconds: 30
    periodSeconds: 10
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 5
  readinessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 10
    timeoutSeconds: 10
    successThreshold: 1
    failureThreshold: 5

  ## Redis slave Resource
  # resources:
  #   requests:
  #     memory: 256Mi
  #     cpu: 100m

  ## Redis slave selectors and tolerations for pod assignment
  # nodeSelector: {"beta.kubernetes.io/arch": "amd64"}
  # tolerations: []

  ## Use an alternate scheduler, e.g. "stork".
  ## ref: https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
  ##
  # schedulerName:

  ## Redis slave pod Annotation and Labels
  podLabels: {}
  podAnnotations: {}

  ## Redis slave pod priorityClassName
  # priorityClassName: {}

  ## Enable persistence using Persistent Volume Claims
  ## ref: http://kubernetes.io/docs/user-guide/persistent-volumes/
  ##
  persistence:
    enabled: true
    ## The path the volume will be mounted at, useful when using different
    ## Redis images.
    path: /data
    ## The subdirectory of the volume to mount to, useful in dev environments
    ## and one PV for multiple services.
    subPath: ""
    ## redis data Persistent Volume Storage Class
    ## If defined, storageClassName: <storageClass>
    ## If set to "-", storageClassName: "", which disables dynamic provisioning
    ## If undefined (the default) or set to null, no storageClassName spec is
    ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
    ##   GKE, AWS & OpenStack)
    ##
    # storageClass: "-"
    accessModes:
    - ReadWriteOnce
    size: 8Gi
    ## Persistent Volume selectors
    ## https://kubernetes.io/docs/concepts/storage/persistent-volumes/#selector
    matchLabels: {}
    matchExpressions: {}

  ## Update strategy, can be set to RollingUpdate or onDelete by default.
  ## https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/#updating-statefulsets
  statefulset:
    updateStrategy: RollingUpdate
    ## Partition update strategy
    ## https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#partitions
    # rollingUpdatePartition:

## Prometheus Exporter / Metrics
##
metrics:
  enabled: false

  image:
    registry: docker.io
    repository: bitnami/redis-exporter
    tag: 1.4.0-debian-10-r3
    pullPolicy: IfNotPresent
    ## Optionally specify an array of imagePullSecrets.
    ## Secrets must be manually created in the namespace.
    ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
    ##
    # pullSecrets:
    #   - myRegistryKeySecretName

  ## Metrics exporter resource requests and limits
  ## ref: http://kubernetes.io/docs/user-guide/compute-resources/
  ##
  # resources: {}

  ## Extra arguments for Metrics exporter, for example:
  ## extraArgs:
  ##   check-keys: myKey,myOtherKey
  # extraArgs: {}

  ## Metrics exporter pod Annotation and Labels
  podAnnotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9121"
  # podLabels: {}

  # Enable this if you're using https://github.com/coreos/prometheus-operator
  serviceMonitor:
    enabled: false
    ## Specify a namespace if needed
    # namespace: monitoring
    # fallback to the prometheus default unless specified
    # interval: 10s
    ## Defaults to what's used if you follow CoreOS [Prometheus Install Instructions](https://github.com/helm/charts/tree/master/stable/prometheus-operator#tldr)
    ## [Prometheus Selector Label](https://github.com/helm/charts/tree/master/stable/prometheus-operator#prometheus-operator-1)
    ## [Kube Prometheus Selector Label](https://github.com/helm/charts/tree/master/stable/prometheus-operator#exporters)
    selector:
      prometheus: kube-prometheus

  ## Custom PrometheusRule to be defined
  ## The value is evaluated as a template, so, for example, the value can depend on .Release or .Chart
  ## ref: https://github.com/coreos/prometheus-operator#customresourcedefinitions
  prometheusRule:
    enabled: false
    additionalLabels: {}
    namespace: ""
    rules: []
      ## These are just examples rules, please adapt them to your needs.
      ## Make sure to constraint the rules to the current postgresql service.
      #  - alert: RedisDown
      #    expr: redis_up{service="{{ template "redis.fullname" . }}-metrics"} == 0
      #    for: 2m
      #    labels:
      #      severity: error
      #    annotations:
      #      summary: Redis instance {{ "{{ $instance }}" }} down
      #      description: Redis instance {{ "{{ $instance }}" }} is down.
      #  - alert: RedisMemoryHigh
      #    expr: >
      #       redis_memory_used_bytes{service="{{ template "redis.fullname" . }}-metrics"} * 100
      #       /
      #       redis_memory_max_bytes{service="{{ template "redis.fullname" . }}-metrics"}
      #       > 90 =< 100
      #    for: 2m
      #    labels:
      #      severity: error
      #    annotations:
      #      summary: Redis instance {{ "{{ $instance }}" }} is using too much memory
      #      description: Redis instance {{ "{{ $instance }}" }} is using {{ "{{ $value }}" }}% of its available memory.
      #  - alert: RedisKeyEviction
      #    expr: increase(redis_evicted_keys_total{service="{{ template "redis.fullname" . }}-metrics"}[5m]) > 0
      #    for: 1s
      #    labels:
      #      severity: error
      #    annotations:
      #      summary: Redis instance {{ "{{ $instance }}" }} has evicted keys
      #      description: Redis instance {{ "{{ $instance }}" }} has evicted {{ "{{ $value }}" }} keys in the last 5 minutes.


  ## Metrics exporter pod priorityClassName
  # priorityClassName: {}
  service:
    type: ClusterIP
    ## Use serviceLoadBalancerIP to request a specific static IP,
    ## otherwise leave blank
    # loadBalancerIP:
    annotations: {}
    labels: {}

##
## Init containers parameters:
## volumePermissions: Change the owner of the persist volume mountpoint to RunAsUser:fsGroup
##
volumePermissions:
  enabled: false
  image:
    registry: docker.io
    repository: bitnami/minideb
    tag: buster
    pullPolicy: Always
    ## Optionally specify an array of imagePullSecrets.
    ## Secrets must be manually created in the namespace.
    ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
    ##
    # pullSecrets:
    #   - myRegistryKeySecretName
  resources: {}
  # resources:
  #   requests:
  #     memory: 128Mi
  #     cpu: 100m

## Redis config file
## ref: https://redis.io/topics/config
##
configmap: |-
  # Enable AOF https://redis.io/topics/persistence#append-only-file
  appendonly yes
  # Disable RDB persistence, AOF persistence already enabled.
  save ""

## Sysctl InitContainer
## used to perform sysctl operation to modify Kernel settings (needed sometimes to avoid warnings)
sysctlImage:
  enabled: false
  command: []
  registry: docker.io
  repository: bitnami/minideb
  tag: buster
  pullPolicy: Always
  ## Optionally specify an array of imagePullSecrets.
  ## Secrets must be manually created in the namespace.
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  ##
  # pullSecrets:
  #   - myRegistryKeySecretName
  mountHostSys: false
  resources: {}
  # resources:
  #   requests:
  #     memory: 128Mi
  #     cpu: 100m

## PodSecurityPolicy configuration
## ref: https://kubernetes.io/docs/concepts/policy/pod-security-policy/
##
podSecurityPolicy:
  ## Specifies whether a PodSecurityPolicy should be created
  ##
  create: false
```

## 部署 Redis

使用 *install* 命令部署 *Chart* 至集群中。

```bash
controlplane $ helm install stable/redis
NAME:   iced-ibis
LAST DEPLOYED: Mon Jul 26 05:44:02 2021
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME                      TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)   AGE
iced-ibis-redis-headless  ClusterIP  None            <none>       6379/TCP  1s
iced-ibis-redis-master    ClusterIP  10.105.234.211  <none>       6379/TCP  0s
iced-ibis-redis-slave     ClusterIP  10.107.214.148  <none>       6379/TCP  0s

==> v1/StatefulSet
NAME                    DESIRED  CURRENT  AGE
iced-ibis-redis-master  1        1        0s
iced-ibis-redis-slave   2        1        0s

==> v1/Pod(related)
NAME                      READY  STATUS   RESTARTS  AGE
iced-ibis-redis-master-0  0/1    Pending  0         0s
iced-ibis-redis-slave-0   0/1    Pending  0         0s

==> v1/Secret
NAME             TYPE    DATA  AGE
iced-ibis-redis  Opaque  1     1s

==> v1/ConfigMap
NAME                    DATA  AGE
iced-ibis-redis         3     1s
iced-ibis-redis-health  6     1s


NOTES:
This Helm chart is deprecated

Given the `stable` deprecation timeline (https://github.com/helm/charts#deprecation-timeline), the Bitnami maintained Redis Helm chart is now located at bitnami/charts (https://github.com/bitnami/charts/).

The Bitnami repository is already included in the Hubs and we will continue providing the same cadence of updates, support, etc that we've been keeping here these years. Installation instructions are very similar, just adding the _bitnami_ repo and using it during the installation (`bitnami/<chart>` instead of `stable/<chart>`)

\\`\\`\\`bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm install my-release bitnami/<chart>           # Helm 3
$ helm install --name my-release bitnami/<chart>    # Helm 2
\\`\\`\\`

To update an exisiting _stable_ deployment with a chart hosted in the bitnami repository you can execute
\\`\\`\\`bash                                                                                                                                               $ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm upgrade my-release bitnami/<chart>
\\`\\`\\`

  Issues and PRs related to the chart itself will be redirected to `bitnami/charts` GitHub repository. In the same way, we'll be happy to answer questions related to this migration process in this issue (https://github.com/helm/charts/issues/20969) created as a common place for discussion.

** Please be patient while the chart is being deployed **
Redis can be accessed via port 6379 on the following DNS names from within your cluster:

iced-ibis-redis-master.default.svc.cluster.local for read/write operations
iced-ibis-redis-slave.default.svc.cluster.local for read-only operations


To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace default iced-ibis-redis -o jsonpath="{.data.redis-password}" | base64 --decode)

To connect to your Redis server:

1. Run a Redis pod that you can use as a client:

   kubectl run --namespace default iced-ibis-redis-client --rm --tty -i --restart='Never' \
    --env REDIS_PASSWORD=$REDIS_PASSWORD \
   --image docker.io/bitnami/redis:5.0.7-debian-10-r32 -- bash

2. Connect using the Redis CLI:
   redis-cli -h iced-ibis-redis-master -a $REDIS_PASSWORD
   redis-cli -h iced-ibis-redis-slave -a $REDIS_PASSWORD

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/iced-ibis-redis-master 6379:6379 &
    redis-cli -h 127.0.0.1 -p 6379 -a $REDIS_PASSWORD
```

*Helm* 现在将启动所需的 *Pod*。您可以使用 `helm ls` 查看所有包。

```bash
controlplane $ helm ls
NAME            REVISION        UPDATED                         STATUS          CHART           NAMESPACE
iced-ibis       1               Mon Jul 26 05:44:02 2021        DEPLOYED        redis-10.5.7    default  
```

如果您收到 *Helm* *无法找到准备好的 Pod* 的错误消息，则表示 *helm* 仍在部署中。稍等片刻，直到 *Docker Image* 完成下载。

在下一步中，我们将验证部署状态。


## 查看结果

*Helm* 部署了所有 *Pod*、*Replication Controller* 和 *Controller* 。使用 *kubectl* 找出部署的内容。

```bash
controlplane $ kubectl get all
NAME                           READY   STATUS    RESTARTS   AGE
pod/iced-ibis-redis-master-0   0/1     Pending   0          12m
pod/iced-ibis-redis-slave-0    0/1     Pending   0          12m

NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/iced-ibis-redis-headless   ClusterIP   None             <none>        6379/TCP   12m
service/iced-ibis-redis-master     ClusterIP   10.105.234.211   <none>        6379/TCP   12m
service/iced-ibis-redis-slave      ClusterIP   10.107.214.148   <none>        6379/TCP   12m
service/kubernetes                 ClusterIP   10.96.0.1        <none>        443/TCP    53m

NAME                                      READY   AGE
statefulset.apps/iced-ibis-redis-master   0/1     12m
statefulset.apps/iced-ibis-redis-slave    0/2     12m
```

在下载 *Docker* 镜像时，*Pod* 将处于 *pending* 状态，直到 *Persistent Volume* 可用。

```bash
controlplane $ kubectl apply -f pv.yaml
persistentvolume/pv-volume1 created
persistentvolume/pv-volume2 created
persistentvolume/pv-volume3 created
```

**pv.yaml**

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume1
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data1"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume2
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data2"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume3
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data3"
```

*Redis* 需要写入权限。
```bash
controlplane $ chmod 777 -R /mnt/data*
```

```bash
controlplane $ kubectl get all
NAME                            READY   STATUS             RESTARTS   AGE
pod/iced-ibis-redis-master-0    1/1     Running            3          18m
pod/iced-ibis-redis-slave-0     0/1     CrashLoopBackOff   3          18m

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/iced-ibis-redis-headless    ClusterIP   None             <none>        6379/TCP   18m
service/iced-ibis-redis-master      ClusterIP   10.105.234.211   <none>        6379/TCP   18m
service/iced-ibis-redis-slave       ClusterIP   10.107.214.148   <none>        6379/TCP   18m
service/kubernetes                  ClusterIP   10.96.0.1        <none>        443/TCP    58m

NAME                                       READY   AGE
statefulset.apps/iced-ibis-redis-master    1/1     18m
statefulset.apps/my-release-redis-master   1/1     36s
```
一旦完成，它将进入*running* 状态。您现在将拥有一个在 *Kubernetes* 之上运行的 *Redis* 集群。

*Helm* 能够为部署的组件提供自定义的名称，例如：

```bash
helm install --name my-release stable/redis
```
