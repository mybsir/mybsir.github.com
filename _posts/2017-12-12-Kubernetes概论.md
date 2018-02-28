---
layout: post
title:  Kubernetes概论
categories: Kubernetes
description: Kubernetes概论
keywords: Kubernetes,Kubernetes概论
---
# Kubernetes 难度不是一点点

## 1. Kubernetes Master

Kubernetes Master主要由kube-apiserver、kube-controller-manager和kube-scheduler三个进程组成，他们运行在集群中一个单独的节点上

### 1.1 APIServer


APIServer负责对外提供RESTful的Kubernetes API服务，它是系统管理指令的统一入口，任何对资源进行增删改查的操作都要交给APIServer处理后再交给etcd。kubectl（Kubernetes提供的客户端工具）是直接和APIServer交互的


### 1.2 Controller Manager


如果APIServer类比于前端、Controller Manager就是负责后端的。
每个资源一般都对应有一个控制器、而Controller Manager就是负责管理这些控制器的，包括：

- Node Controller
- Replication Controller
- Endpoints Controller
- Service Account
- Token Controller


### 1.3 Scheduler


负责Kubernetes集群内部的资源调度，主要负责监视Kubernetes集群内部已创建但没有被调度到某个Node上的Pod，然后将该Pod调度到一个选定的Node上面运行


### 1.4 ectd（附加）


etcd是一个高可用的键值存储系统，Kubernetes使用它来存储各个资源的状态，从而实现Restful的API



## 2. Kubernetes Node

每个Node节点主要由三个模块组成：kubelet、kube-proxy、runtime

### 2.1 runtime


runtime指的是容器运行环境，目前最流行的就是docker容器


### 2.2 kube-proxy


在Kubernetes集群中，每个Node上都有一个网络代理进程，它主要负责为Pod对象提供代理：
定期从etcd存储中获取所有的Service对象，并根据Service信息创建代理。
当某个Client要访问一个Pod时，请求会经过该Node上的代理进程进行转发

另一种解释：

该模块实现了Kubernetes中的服务发现和反向代理功能：
反向代理方面：kube-proxy支持TCP和UDP连接转发，默认基于Round Robin算法将客户端流量转发到与service对应的一组后端pod。
服务发现方面：kube-proxy使用etcd的watch机制，监控集群中service和endpoint对象数据的动态变化，并且维护一个service到endpoint的映射关系，从而保证了后端pod的IP变化不会对访问者造成影响。
另外kube-proxy还支持session affinity



### 2.3 kubelet


它是每个Node节点上最重要的模块，负责监视指派到它所在Node上的Pod，但是如果容器不是通过Kubernetes创建的，它并不会管理。
本质上，它负责使Pod的运行状态与期望的状态一致。
kubelet还负责处理如下工作：
- 为Pod挂载Volume
- 下载Pod拥有的Secret
- 运行属于Pod的容器
- 周期性的检查Pod中的容器是否存活
- 向Master报告当前Node上Pod的状态信息
- 向Master报告当前Node的状态信息


## 3. NameSpace


在一个Kubernetes集群中，可以使用Namespace创建多个“虚拟集群”，这些Namespace之间可以完全隔离，也可以通过某种方式，使一个Namespace的中的Service可以访问到其他Namespace中的Service

查看当前Kubernetes集群中的Namespace列表，执行以下命令

```
kubectl get namespaces
```

默认情况下，会有两个系统自动创建好的Namespace

- default: Kubernetes集群中没有Namespace的对象都会放到该默认Namespace中
- kube-system: Kubernetes集群自动创建的Namespace


## 4. Pod

Pod是Kubernetes的基本操作单元，也是应用运行的载体。整个Kubernetes系统都是围绕着Pod展开的，比如如何部署运行Pod、如何保证Pod的数量、如何访问Pod等。Pod可以认为是容器的一种延伸扩展，一个Pod也是一个隔离体，而Pod内部包含的一组容器又是共享的（包括PID、Network、IPC、UTS）。


在Kubernetes集群中，Pod有两种使用方式：

- 一个Pod运行一个容器

    这种模式是最常见的用法，可以把Pod想想成是单个容器的封装，Kubernetes直接管理的是Pod，而不是Pod内部的容器

- 一个Pod中同时运行多个容器

    一个Pod中也可以同时运行多个容器，这些容器之间需要紧密协作，并共享资源。这些在同一个Pod中的容器可以互相协作，逻辑上代表一个Service对象。每个Pod都是应用的一个实例，如果我们想要运行多个实例，就应该运行多个Pod。
    同一个Pod的容器，会自动分配到同一个Node上。每个Pod都会被分配一个唯一的IP地址，该Pod中的所有容器共享网络空间，包括IP地址和端口。Pod内部的容器可以使用localhost互相通信。Pod中的容器与外界通信时，必须分配共享网络资源（例如：使用宿主机的端口映射）
    我们可以为一个Pod指定多个共享的Volume，它内部的所有容器都可以访问共享的Volume。Volume也可以用来持久化Pod中的存错资源，以防容器重启后文件丢失

### 4.1 Pod基本操作


| 创建 | kubectl create -f xxx.yaml |
|-----|--------------------------- |
| 查询 | kubectl get pod yourPodName <br> kubectl describe pod yourPodName |
| 删除 | kubectl delete pod yourPodName |
| 更新 | kubectl replace /path/to/yourNewYaml.yaml |

### 4.2 镜像

在Kubernetes集群中，镜像的下载策略为：

- **Always**: 每次都下载最新的镜像
- **Never**: 只使用本地镜像，从不下载
- **IfNotPresent**: 只有当本地没有的时候才下载镜像

Pod被分配到Node之后会根据镜像下载策略进行镜像下载，可以根据自身集群的特点来决定采用何种下载策略。

### 4.3 其他设置

通过yaml文件，可以再Pod中设置：

- **启动命令**： 如 spec --> containers --> command;
- **环境变量**： 如 spec --> containers --> env --> name/value;
- **端口桥接**： 如 spec --> containers --> ports --> containerPort/protocol/hostIP/hostPort
- **Host网络**： 如 spec --> hostNetwork=true;在一些特殊场景下，容器必须要以host方式进行网络设置（如接受物理机网络才能接收到的组播流）
- **数据持久化**: 如 spec --> containers --> volumeMounts --> mountPaht
- **重启策略**： 当Pod中的容器终止退出后，重启容器的策略。这里的所谓Pod的重启，实际上的做法是容器的重建，之前容器中的数据会丢失，如果需要持久化数据，那么需要使用数据卷进行持久化设置。Pod支持三种重启策略：Always(默认策略，当容器终止退出后，总是重启容器)、OnFailure（当容器终止且异常退出时，重启）、Never（从不重启）


### 4.4 Pod生命周期

Pod被分配到一个Node上之后，就不会离开这个Node，直到被删除。当某个Pod失败。首先会被Kubernetes清理掉，之后ReplicationController将会在其他机器上（或本机）重建Pod，重建之后Pod的ID发生了变化，那将会是一个新的Pod。所以，Kubernetes中Pod的迁移，实际指的是在新Node上重建Pod。下图是Pod生命周期图

![](./images/pod_alive.png)


### 4.5生命周期回调函数
PostStart(容器创建成功后调研该回调函数)、PreStop（在容器被终止前调用该回调函数），以下实例中，定义了一个Pod，包含一个Java的Web应用容器，其中设置了PostStart和PreStop回调函数。即在容器创建成功后，复制/sample.war到/app文件夹中。而在容器终止之前，发送Http请求到http://monitor.com:8080/waring，即向监控系统发送警告。具体示例如下：

```yaml
………..
containers:
- image: sample:v2  
     name: war
     lifecycle：
      posrStart:
       exec:
         command:
          - “cp”
          - “/sample.war”
          - “/app”
      prestop:
       httpGet:
        host: monitor.com
        psth: /waring
        port: 8080
        scheme: HTTP
```

## 5. Replication Controller

Replication Controller (RC) 是Kubernetes中的另一个核心概念，应用托管在Kubernetes之后，Kubernetes需要保证应用能够持续运行，这是RC的工作内容，它会确保任何时间Kubernetes中都有指定数量的Pod在运行。如果Pod副本的数量过多，则RC会kill掉部分Pod使其数量与预期保持一致，如果Pod数量过少，则会自动创建新的Pod副本以与预期数量相同。在此基础上，RC还提供了一些更高级的特性，比如滚动升级、升级回滚等

示例：

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
运行:
```shell
kubectl create -f ./replication.yaml
replicationcontroller "nginx" created
```
检查RC运行状态
```shell
$ kubectl describe replicationcontrollers/nginx

Name:         nginx
Namespace:    default
Selector:     app=nginx
Labels:       app=nginx
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  1 Running / 2 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx
    Port:         80/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                    Message
  ----    ------            ----  ----                    -------
  Normal  SuccessfulCreate  7s    replication-controller  Created pod: nginx-b2ktx
  Normal  SuccessfulCreate  7s    replication-controller  Created pod: nginx-npn6j
  Normal  SuccessfulCreate  7s    replication-controller  Created pod: nginx-fqqmj

```

列出RC某一个selector中的Pod

```
# kubectl get pods --selector=app=nginx
NAME          READY     STATUS    RESTARTS   AGE
nginx-b2ktx   1/1       Running   0          1m
nginx-fqqmj   1/1       Running   0          1m
nginx-npn6j   1/1       Running   0          1m
```

删除RC

```
# kubectl delete replicationcontrollers/nginx
replicationcontroller "nginx" deleted
```
ReplicaSet是Replication Controller的下一代实现，它们之间没有本质的区别，除了ReplicaSet支持在selector中通过集合的方式进行配置。在新版本的Kubernetes中支持并建议使用ReplicaSet，而且我们应该尽量使用Deployment来管理一个ReplicaSet。

### 5.1 RC与Pod的关联--Label

RC与Pod的关联是通过Label来实现的，Label机制是Kubernetes中一个重要设计，通过Label进行对象的弱关联，可以灵活的进行分类和选择。对于Pod，需要设置其自身的Label来进行标识，Label是一系列的Key/Value对，在Pod-->metadata-->Labels中进行设置


我们可以通过给指定的资源对象捆绑一个多个不同的Label来实现多维度的资源分组管理功能，以便于灵活、方便的进行资源分配、调度、配置、部署等管理工作。例如：部署不同版本的应用到不同的环境中；或者监控和分析应用（日志记录、监控、告警）等。以下为常用Label示例

```
      版本标签  "release" : "stable", "release" : "canary"

　　　　环境标签  "environment" : "dev", "environment" : "qa", "environment" : "production"

　　　　架构标签  "tier" : "frontend", "tier" : "backend", "tier" : "cache"

　　　　分区标签  "partition" : "customerA", "partition" : "customerB"

　　　　质量管控标签  "track" : "daily", "track" : "weekly"
```
例如：当你在RC的yaml文件中定义了该RC的selector中的label为app:my-web，那么这个RC就会去关注Pod-->metadata-->labels中label为app:my-web的Pod。修改了对应Pod的Label，就会使Pod脱离RC的控制。同样，在RC运行正常的时候，若试图继续创建相同Label的Pod，是创建不出来的。因为RC认为副本数已经正常了，再多起的话会被RC删除的

### 5.2 弹性收缩
弹性收缩时指适应负载变化，以弹性可伸缩的方式提供资源。反应到Kubernetes中，指的是可根据负载的高低动态调整Pod的副本数量。调整Pod的副本数是通过修改RC中Pod的副本来实现的。示例如下:
扩容Pod的副本数量到10
```
[root@iZuf6ib63rx9bjpdgylm5aZ ~]# kubectl get pods --selector=app=nginx
NAME          READY     STATUS    RESTARTS   AGE
nginx-dt58s   1/1       Running   0          14s
nginx-gjr2z   1/1       Running   0          14s
nginx-h4kcz   1/1       Running   0          14s
[root@iZuf6ib63rx9bjpdgylm5aZ ~]# kubectl scale  --replicas=10 replicationcontrollers/nginx
replicationcontroller "nginx" scaled
[root@iZuf6ib63rx9bjpdgylm5aZ ~]# kubectl get pods --selector=app=nginx
NAME          READY     STATUS              RESTARTS   AGE
nginx-7c7jh   0/1       ContainerCreating   0          3s
nginx-85tlm   0/1       ContainerCreating   0          3s
nginx-8h6cz   0/1       ContainerCreating   0          3s
nginx-bfzzl   0/1       ContainerCreating   0          3s
nginx-dt58s   1/1       Running             0          23s
nginx-gjr2z   1/1       Running             0          23s
nginx-h4kcz   1/1       Running             0          23s
nginx-p4fkp   0/1       ContainerCreating   0          3s
nginx-xvv7l   0/1       ContainerCreating   0          3s
nginx-zfchx   0/1       ContainerCreating   0          3s
[root@iZuf6ib63rx9bjpdgylm5aZ ~]#
```
缩容Pod的副本数量到1
```
[root@iZuf6ib63rx9bjpdgylm5aZ ~]# kubectl scale  --replicas=1 replicationcontrollers/nginx
replicationcontroller "nginx" scaled
[root@iZuf6ib63rx9bjpdgylm5aZ ~]# kubectl get pods --selector=app=nginx
NAME          READY     STATUS        RESTARTS   AGE
nginx-7c7jh   1/1       Terminating   0          1m
nginx-85tlm   0/1       Terminating   0          1m
nginx-8h6cz   0/1       Terminating   0          1m
nginx-bfzzl   1/1       Terminating   0          1m
nginx-dt58s   1/1       Running       0          2m
nginx-gjr2z   1/1       Terminating   0          2m
nginx-h4kcz   0/1       Terminating   0          2m
nginx-p4fkp   0/1       Terminating   0          1m
nginx-xvv7l   1/1       Terminating   0          1m
nginx-zfchx   0/1       Terminating   0          1m
[root@iZuf6ib63rx9bjpdgylm5aZ ~]# kubectl get pods --selector=app=nginx
NAME          READY     STATUS        RESTARTS   AGE
nginx-7c7jh   0/1       Terminating   0          1m
nginx-85tlm   0/1       Terminating   0          1m
nginx-8h6cz   0/1       Terminating   0          1m
nginx-dt58s   1/1       Running       0          2m
[root@iZuf6ib63rx9bjpdgylm5aZ ~]# kubectl get pods --selector=app=nginx
NAME          READY     STATUS    RESTARTS   AGE
nginx-dt58s   1/1       Running   0          2m
```

### 5.3 滚动升级

滚动升级是一种平滑过渡的升级方式，通过逐步替换策略，保证整体系统的稳定，在初始升级的时候就可以及时发现、调整问题，以保证问题影响度不会扩大，滚动升级的命令如下：

```
kubectl rolling-update my-rcName-v1 -f my-rcName-v2-rc.yaml --update-period=10s
```
升级开始后，首先依据提供的定义文件创建V2版本的RC，然后每隔10s(--update-period=10s)逐步增加V2版本的Pod副本数，逐步减少V1版本Pod的副本数。升级完成之后，删除V1版本的RC，保留V2版本的RC，及实现滚动升级
升级过程中，发生了错误中途退出时，可以选择继续升级。Kubernetes能够智能的判断升级中断之前的状态，然后紧接着继续执行升级。当然，也可以进行回退，命令如下：
```
kubectl rolling-update my-rcName-v1 -f my-rcName-v2-rc.yaml --update-period=10s --rollback
```
回退的方式实际就是升级的逆操作，逐步增加V1.0版本Pod的副本数，逐步减少V2版本Pod的副本数。
