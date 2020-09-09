---
title: kubernates 必知必会
tags: kubernates
key: "kubernates:2020-09-06"
---

## 问题

1. kubernates 是什么？
2. kubernates 解决了什么问题？
3. kubernates 该如何使用？
5. 集群中资源是如何调度的？

## k8s 是什么, 解决了什么问题

kubernates 是 Google 公司开源的一个容器编排与调度管理框架, 当前由 CNCF 托管. 

解决了分布式系统情况下, 容器的编排与调度问题.

### 发展历史

1. 2003-2004 年, Google 发布 Borg 系统, 3-4 人开发.  

   主要用于管理长时间运行的生产服务和批处理服务. Brog 将上述两种服务所需要的的机器统一成一个池子, 以便提高资源利用率, 同时降低成本. 

   之所以能够实现跨机器的资源共享以及进程隔离, 是因为可以拿到 Linux 内核的容器支持.

2. ... 年, 越来越多的应用被开发并运行在 Borg上, Google 团队开发了一个工具系统. 

   工具系统提供配合和更新 job 的机制, 可以预测资源需求, 动态的推送配置文件, 服务发现, 负载均衡, 自动扩容, 机器的生命周期管理, 额度管理等等. 

   由于要适配不同团队的需求, Borg 发展成了 ad-hoc 系统(即席查询, 用户根据自己的需求，灵活的选择查询条件，系统能够根据用户的选择生成相应的统计报表)集合, Borg 使用者可以使用不同的配置语言和进程来配置与沟通.

3. 2006-2007 年, Paul Menage和Rohit Seth 提出了 cgroups, 并且被合并到2.6.24版的内核中

4. 2013 年左右, 处于提升 Borg 生态系统软件工程的愿景, Google 发布了 Omega 集群管理系统. 

   Omega 把很多 Borg 内已经被认证的成功的模式搬过来, 同时从头开始搭建一个更加一致性的构架. 添加了控制面板, 调度器等等, 以此来优化偶尔发生的资源冲突问题.  同年, docker 正式出道.

5. 2014 年左右, Google 发布了 kubernates. 同年, Microsoft, Red Hat, IBM, Docker 等加入 kubernates 社区

6. 2015 年, 成立了 CNCF 基金会, 托管 kubernates

7. 2016 年左右, kubernates 成为主流. 

   在 CloudNative 2016 大会上, 来自全世界的贡献值和开发者一起交流kubernates, Fluentd, Promethues等云原生技术(采用开源堆栈 K8S+Docker 进行容器化，基于微服务架构提高灵活性和可维护性，借助敏捷方法、DevOps支持持续迭代和运维自动化，利用云平台设施实现弹性伸缩、动态调度、优化资源利用率)

8. 2017 年左右, 各大互联网厂商开始纷纷支持 kubernates. 同年, istio 正式出道. Docker 容器大战正式结束, 成功上位, 作为 kubernates 容器运行时的标配.

9. 2018 年左右, 无人不知 kubernates. 国内不论大小厂商开始进行云原生落地.

### Borg/Omega/K8S关联

详情见: [https://queue.acm.org/detail.cfm?id=2898444](https://queue.acm.org/detail.cfm?id=2898444)

### k8s特点

1. 可移植性: 支持公有云, 私有云, 混合云, 多重云
2. 可扩展性: 模块化, 插件化, 可挂载, 可组合
3. 自动化: 自动部署, 自动重启, 自动复制, 自动伸缩/扩展

## k8s 该如何使用

### 环境准备篇

#### 本地开发环境
详情见: [https://github.com/chaimch/k8s-for-docker-desktop](https://github.com/chaimch/k8s-for-docker-desktop)

#### 云羊毛环境
GCP 羊毛, 详情见: [https://console.cloud.google.com/freetrial](https://console.cloud.google.com/freetrial)

katacoda羊毛, 详情见: [https://katacoda.com/learn](https://katacoda.com/learn)

### 入门

#### 自动补全提示
1. 安装 bash-completion `yum install bash-completion`
2. 设置 kubectl 的别称 `alias k=kubectl`
3. bash 环境下, 调整别称 k 的自动补全 `source <(kubectl completion bash | sed s/kubectl/k/g)`
4. zsh 环境下, 调整 k 的自动补全 `source <(kubectl completion zsh)`
5. 设置切换名称空间别称 `alias kcd='k config set-context $(k config current-context) --namespace'`

#### hello world

创建配置

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

创建应用

```
kubectl create -f 我的配置文件
```

查看应用

```
$ kubectl get pods -l app=nginx
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-67594d6bf6-9gdvr   1/1       Running   0          10m
nginx-deployment-67594d6bf6-v6j7w   1/1       Running   0          10m
```

#### Kubernates 官方

详情见: [https://kubernetes.io/docs/tutorials/kubernetes-basics/](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

### 为什么需要 Pod

容器的本质是进程, Kubernates 的本质是操作系统

```
pstree -g
systemd(1)-+-accounts-daemon(1984)-+-{gdbus}(1984)
           | `-{gmain}(1984)
           |-acpid(2044)
          ...      
           |-lxcfs(1936)-+-{lxcfs}(1936)
           | `-{lxcfs}(1936)
           |-mdadm(2135)
           |-ntpd(2358)
           |-polkitd(2128)-+-{gdbus}(2128)
           | `-{gmain}(2128)
           |-rsyslogd(1632)-+-{in:imklog}(1632)
           |  |-{in:imuxsock) S 1(1632)
           | `-{rs:main Q:Reg}(1632)
           |-snapd(1942)-+-{snapd}(1942)
           |  |-{snapd}(1942)
           |  |-{snapd}(1942)
           |  |-{snapd}(1942)
           |  |-{snapd}(1942)
```

在真正的操作系统中, 进程往往是以进程组的方式被组织在一起. 比如 rsyslogd 程序, 负责 linux 日志处理. 其中主程序 main, 和内核日志模块 imklog 等同属于 1632 进程组. 这些进程之间相互协作.

如果将 rsyslogd 应用容器化, 由于受限于容器的“单进程模型”(容器没有管理多个进程的能力)，这三个模块往往需要被分别制作成三个不同的容器. 一旦 main 容器所在的节点不足以承担三个容器同时运行时候, 就容易出现成组调度问题.

但是在 Kubernates 中就存在这种问题, Pod 是 Kubernetes 里的原子调度单位, 统一按照 Pod 而非容器的资源需求进行计算调度. 一般容器间会发生直接的文件交换, 使用 localhost 或 socket 文件进行本地通信, 非常频繁的远程调用, 共享 ns 等适合需要将多个容器放到同一个 pod 中.

#### pod 实现原理

Pod 里的所有容器, 共享的是同一个 Network Namespace, 并且可以声明共享同一个 Volume. 

Kubernates 真正处理的还是 linux 容器的 ns 和 cgroups, 不并存在 pod 的边界或者隔离环境. pod 里所有的容器, 共享同一个 network namespace, 并且可以声明共享同一 volume. 相当于 A 和 B 两个容器的 pod, 等同于容器 A 共享容器 B 的网络和 volume. 

通过以下命令就可以实现

```
docker run --net=B --volumes-from=B --name=A image-A
```

但是, 上述的方式存在一个问题. B 容器必须要在 A 容器之前启动, 依次类推多个容器之前就需要去维护拓扑关系. 因此在 Kubernates 中会直接首先创建一个 infra 的容器, 这样其他的容器就可以直接以 Join Network Namespace 方式加入进来, 其他的容器直接就只是对等关系, 彼此不会相互依赖.

##### 容器设计模式

详情见: [https://www.usenix.org/conference/hotcloud16/workshop-program/presentation/burns](https://www.usenix.org/conference/hotcloud16/workshop-program/presentation/burns)


### 架构图
![architecture](https://cdn.jsdelivr.net/gh/chaimch/FigureBed@master/uPic/architecture.png)

### 组件功能

#### kubectl
官方提供的 CLI, 可以以交互式对 Kubernates API Server 进行操作. kubectl 发送相应的 http 请求, 由 Kubernates API Server 处理后返回并展示出结果.

#### kube-apiserver

负责将 Kubernates 的资源组/资源版本/资源以 restful 风格的形式对外暴露并提供服务. Kubernates 集群中的所有组件都通过 kube-apiserver 组件操作资源对象. 

##### 特点

1. 所有资源对象都封装成 restful 风格的 api 接口进行管理
2. 集群状态管理和数据管理, 是唯一与 etcd 集群交互的组件
3. 拥有丰富的集群安全访问机制, 以及认证, 授权及准入控制器

#### kube-controller-manager

管理 Kubernates 集群中Node节点, Pod副本, 服务, Endpoint端点, Namespace 命名空间, Service Account 服务账户, ResourceQuota 等. eg: 某个节点宕机时候,  Controller Manager 会及时发现并执行自动化修复流程.

Controller Manager 具备高可用性, 基于 etcd 集群上的分布式锁实现领导者选举机制. 多个实例同时运行,  通过 kube-apiserver 提供的资源锁进行选举竞争.

|                 简化版本的 Kubernates 架构图                 |                       简易版本冰箱架构                       |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20200908125907635](https://cdn.jsdelivr.net/gh/chaimch/FigureBed@master/uPic/image-20200908125907635.png) | ![image-20200908130029579](https://cdn.jsdelivr.net/gh/chaimch/FigureBed@master/uPic/image-20200908130029579.png) |

类似冰箱一样, 对 Kubernates 的操作都需要统一的入口管理, 即通过 api-server . 

|                          冰箱 V1.0                           |                          冰箱 V2.0                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20200908131150235](https://cdn.jsdelivr.net/gh/chaimch/FigureBed@master/uPic/image-20200908131150235.png) | ![image-20200908131220428](https://cdn.jsdelivr.net/gh/chaimch/FigureBed@master/uPic/image-20200908131220428.png) |

控制器专门用于控制各个系统的状态. 打开门时候, 控制器监控到门的变化, 替用户打开灯. 用户按下温控器时候, 控制器观察到用户设置的温度, 则它替用户去管理制冷系统.

|                          冰箱 V3.0                           |
| :----------------------------------------------------------: |
| ![image-20200908131321132](https://cdn.jsdelivr.net/gh/chaimch/FigureBed@master/uPic/image-20200908131321132.png) |

#### kube-scheduler

Kubernates 集群的默认调度器, 主要负责在 Kubernates 集群中为一个 Pod 资源对象找到合适的节点, 并在该节点上运行. 该调度器为每个 Pod 寻找合适节点的过程称之为调度周期, 一个调度周期之内只调度一个 Pod 资源对象. 该组件监控整个集群的 Pod 资源对象和 Node 资源对象, 一旦有新的 Pod 资源对象时, 会通过调度算法为其选择最优的节点. 

#### kubelet

该组件运行在每个 Kubernates 集群中的节点上, 用于接收, 处理, 上报 kube-apiserver 组件下发的任务.

 Kubelet 进程启动时会向 Kube-apiserver 注册节点自身信息, 定时上报所在节点的资源使用状态, 用以帮助kube-schedule 调度器为 Pod 资源对象预选节点. 

同时负责所在节点上的 Pod 资源对象的管理(Pod 资源对象的创建, 修改, 监控, 删除, 驱逐及Pod 声明周期管理等), 对所在节点的镜像和容器做一些清理工作, 保证节点上的镜像不会占满磁盘空间, 删除容器释放相关资源.

kubelet 开放的接口, CRI 容器运行时接口, CNI 容器网络接口, CSI 容器存储接口.

#### kube-proxy

节点上的网络代理, 监控 kube-apiserver 服务于端点资源变化, 同时可以通过 iptables/ipvs 等配置负载均衡器, 为一组 pod 提供统一的 tcp/udp 的流量转发和负载均衡功能.

