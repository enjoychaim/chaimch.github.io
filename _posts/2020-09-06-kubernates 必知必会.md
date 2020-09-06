---
title: kubernates 必知必会
tags: kubernates
key: "kubernates:2020-09-06"
---

## 问题

1. kubernates 是什么？
2. kubernates 解决了什么问题？
3. kubernates 该如何使用？
4. 定义的资源是如何被编排的?
5. 集群中资源是如何调度的？

## kubernates 是什么, 解决了什么问题

kubernates 是 Google 公司开源的一个容器编排与调度管理框架, 当前由 CNCF 托管. 

解决了分布式系统情况下, 容器的编排与调度问题.

### 发展历史

1. 2003-2004 年, Google 发布 Borg 系统, 3-4 人开发.  主要用于管理长时间运行的生产服务和批处理服务. Brog 将上述两种服务所需要的的机器统一成一个池子, 以便提高资源利用率, 同时降低成本. 之所以能够实现跨机器的资源共享以及进程隔离, 是因为可以拿到 Linux 内核的容器支持.``
2. ... 年, 越来越多的应用被开发并运行在 Borg上, Google 团队开发了一个工具系统. 工具系统提供配合和更新 job 的机制, 可以预测资源需求, 动态的推送配置文件, 服务发现, 负载均衡, 自动扩容, 机器的生命周期管理, 额度管理等等. 由于要适配不同团队的需求, Borg 发展成了 ad-hoc 系统(即席查询, 用户根据自己的需求，灵活的选择查询条件，系统能够根据用户的选择生成相应的统计报表)集合, Borg 使用者可以使用不同的配置语言和进程来配置与沟通.
3. 2006-2007 年, Paul Menage和Rohit Seth 提出了 cgroups, 并且被合并到2.6.24版的内核中
4. 2013 年左右, 处于提升 Borg 生态系统软件工程的愿景, Google 发布了 Omega 集群管理系统. Omega 把很多 Borg 内已经被认证的成功的模式搬过来, 同时从头开始搭建一个更加一致性的构架. 添加了控制面板, 调度器等等, 以此来优化偶尔发生的资源冲突问题.  同年, docker 正式出道.
5. 2014 年左右, Google 发布了 kubernates. 同年, Microsoft, Red Hat, IBM, Docker 等加入 kubernates 社区
6. 2015 年, 成立了 CNCF 基金会, 托管 kubernates
7. 2016 年左右, kubernates 成为主流. 在 CloudNative 2016 大会上, 来自全世界的贡献值和开发者一起交流kubernates, Fluentd, Promethues等云原生技术(采用开源堆栈 K8S+Docker 进行容器化，基于微服务架构提高灵活性和可维护性，借助敏捷方法、DevOps支持持续迭代和运维自动化，利用云平台设施实现弹性伸缩、动态调度、优化资源利用率)
8. 2017 年左右, 各大互联网厂商开始纷纷支持 kubernates. 同年, istio 正式出道. Docker 容器大战正式结束, 成功上位, 作为 kubernates 容器运行时的标配.
9. 2018 年左右, 无人不知 kubernates. 国内不论大小厂商开始进行云原生落地.

### 特点

1. 可移植性: 支持公有云, 私有云, 混合云, 多重云
2. 可扩展性: 模块化, 插件化, 可挂载, 可组合
3. 自动化: 自动部署, 自动重启, 自动复制, 自动伸缩/扩展



## kubernates 该如何使用

## 本地开发环境

参见: https://github.com/chaimch/k8s-for-docker-desktop

## 云上薅羊毛环境

参见一, gcp 300$ 随便薅: https://console.cloud.google.com/freetrial

参见二, katacoda 屌丝便捷必备: https://katacoda.com/learn

### 架构图

![architecture](https://cdn.jsdelivr.net/gh/chaimch/FigureBed@master/uPic/architecture.png)

