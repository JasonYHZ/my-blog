---
title: 面向开发者的 Kubernetes 入门系列 - 基础篇
date: 2022-05-23 10:00:00
comment: true
cover: https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202205241738862.jpeg
toc: true
tags:
    - Kubernetes 入门
categories:
    - 教程
    - 运维
description: 本文针对于开发者或者是想去了解入门 Kubernetes 的读者，以快速上手为目标，将 Kubernetes 的核心的一些概念尽量简短快速的做一个介绍.
---

## 前言
本文针对于开发者或者是想去了解入门 Kubernetes 的读者，以快速上手为目标，将 Kubernetes 的核心的一些概念尽量简短快速的做一个介绍，因为 Kubernetes 的内容非常的庞大且繁琐（对的就是繁琐并不是复杂，因为它的概念太多了对新手来说不知道从哪里入手），所以这也是我写这篇文章的原因，其实 Kubernetes 学习起来并不复杂，只要先了解了最基础的一些核心概念，然后直接上手实践就是最好的学习路径，因此才有了这个入门系列的文章，最终目标是可以让想要入门学习 Kubernetes 的开发者、运维人员等 IT 从业者用最短的时间将 Kubernetes 这个好玩的玩具玩起来。
要学习它还是有些前提基础要掌握的不过比较简单，需要读者们掌握 Linux、Docker 即可。
当然学习它之前还是需要将一些概念掌握清楚才可以，这也是必不可少的，当然也是比较枯燥的，而且也是困住新手的第一大关，过不了这一关就没办法上手去玩它就永远没办法学会它。不过放心我并不是想做一个大头书文档，而是用一些通俗易懂的解释将一些比较关键的概念解释清楚，目的就是玩它！
此外还要声明一下如果有了基础的同学，之后再学习 Kubernetes 最好的办法、最好的资料就是官方文档，没有之一。


## Kubernetes 的起源与作用
### 起源
在学习一门技术之前我们不妨先来了解下它的出身历史，以便更好的去了解它学习它。
近几年 Kubernetes 这个词被炒的热火朝天，有稍微了解过的同学可能知道它是 Google 公司于2014年6月份将内部的一套容器编排调度引擎 Borg 系统改造后开源来的，因此它的核心就是针对容器的编排调度，所以给它起了一个特别形象的名字 “Kubernetes” 希腊语里意为 “舵手”、“飞行员”或“州长”之类的以及控制类的词的根源，然后由于这个单词 “K” 和 “s” 之间有8个字母所以简称就叫了 “K8s”。下面来看下一个形象的图加深下对它的理解。

![](./k8s.jpeg)

### 容器
由于前文提到 Kubernetes 的核心就是针对容器的编排调度，所以我们这里也大概介绍下什么是容器。
其实容器技术很早就已经出现了，最早是在1979年就在 Unix v7 系统中得到了支持，一致发展到2013年随着 Docker 的诞生才把容器化推向了风口。
容器指的是系统虚拟化的一种形式，有点类似虚拟机，有自己的文件系统、CPU、内存、进程空间等。
但是它比虚拟机更轻量化，因为各容器之间共享一个操作系统内核，容器使用操作系统的虚拟化形式，利用操作系统的功能（例如Linux内核中的名称空间和cgroups）来隔离进程，并控制这些进程可以使用的 CPU、内存和磁盘的数量，因此容器它与虚拟机不同，容器不需要在每次启动时都包含一个完整的操作系统，它只需要利用主机操作系统的功能和资源即可。
目前我们使用最多的应该就是 Docker 这个容器，其实并不是只有 Docker，因为在 2015年6月22日的时候 Docker 公司就将他们的 [libcontainer](https://github.com/docker-archive/libcontainer) 技术开源，并联合 CoreOS、Google、RedHat 等公司一起将此技术改名为成立了 RunC 项目，并交由中立的开源基金会管理，并以 RunC 为基础，共同制定了一套容器和镜像的标准和规范，称为 OCI：Open Container Initiative。
所以目前除了Docker之外还有一些基于 RunC 的容器技术例如 Podman、CRI-O、Containerd等。
聊完容器之后还要在了解一个概念叫容器化，容器只是一个环境，能够为程序提供一个类似于虚拟机但比虚拟机更高效的运行环境，因此容器化其实就是将原先的应用程序所需要的所有内容，包括编译后的二进制文件、依赖关系、库和配置文件等等，全部都封装进容器之中，然后可以针对起容器做一些基础资源的访问限制，例如可用的cpu、内存、存储等资源，类似于一个轻量化的虚拟机，然后容器化过的应用可以无视任何操作系统架构和环境，都能直接运行容器化应用而不需要重复的编译构建，解决运行环境依赖等问题。

## Kubernetes 架构

在继续深入了解 Kubernetes 其他的概念之前，先来介绍下 Kubernetes 的总体架构，先来认识个全貌再来介绍具体内容。

### 集群架构

我们先来看一张完整的 Kubernetes 的集群图。

![](./framework.png)

Kubernetes 是一个生产级别的开源平台，可以协调高可用的计算机集群，它可以协调在集群内或者夸集群的应用容器的部署调度和执行。
每个计算机作为独立的单元相互连接到集群中并由 Kubernetes 进行控制管理， Kubernetes 中的抽象允许你将容器化的应用部署到集群，而无需将它们绑定到某个特定的独立计算机。
为了使用这种新的部署模型，应用需要以将应用与单个主机分离的方式打包：因此它们需要被容器化。与过去的那种应用直接以包的方式深度与主机集成的部署模型相比，容器化应用更灵活、更可用。
如图中所示，一个 Kubernetes 集群中主要包含两种类型的资源：

- **Master** 负责管理整个集群

 Master 协调集群中的所有活动，例如调度应用、维护应用的所需状态、应用扩容以及推出新的更新。

- **Node** 负责运行应用，它可以是一个虚拟机或者物理机

**它在 Kubernetes 集群中充当工作机器的角色** 每个Node都有 Kubelet , 它管理 Node 而且是 Node 与 Master 通信的代理。
Node 还应该具有用于处理容器操作的工具，例如 Docker。
在 Kubernetes 上部署应用时，你告诉 Master 启动应用容器。 Master 就编排容器在集群的 Node 上运行。** Node 使用 Master 暴露的 Kubernetes API 与 Master 通信。**终端用户也可以使用 Kubernetes API 与集群交互。

### Node 节点

接着我们来了解一下工作节点 ** Node **中都有什么，为此我们来看一下这张图

![](./module_03_nodes.svg)

可以看到 Node 节点上只有一种资源类型，那就是 Pod，它是 Kubernetes 中创建和管理的、最小的可部署的计算单元。

### Pod

Pod 是 Kubernetes 抽象出来的，表示一组一个或多个应用程序容器（如 Docker），以及这些容器的一些共享资源。这些资源包括:

- 共享存储，当作卷
- 网络，作为唯一的集群 IP 地址
- 有关每个容器如何运行的信息，例如容器映像版本或要使用的特定端口。

除了应用容器，Pod 还可以包含在 Pod 启动期间运行的 [Init 容器](https://kubernetes.io/zh/docs/concepts/workloads/pods/init-containers/)。 你也可以在集群中支持[临时性容器](https://kubernetes.io/zh/docs/concepts/workloads/pods/ephemeral-containers/) 的情况下，为调试的目的注入临时性容器。
一个 pod 总是运行在 **工作节点**。工作节点是 Kubernetes 中的参与计算的机器，可以是虚拟机或物理计算机，具体取决于集群。每个工作节点由主节点管理。工作节点可以有多个 pod 。
Kubernetes 主节点会自动处理在集群中的工作节点上调度 pod 。 主节点的自动调度考量了每个工作节点上的可用资源。
每个 Kubernetes 工作节点至少运行:

- Kubelet，负责 Kubernetes 主节点和工作节点之间通信的过程; 它管理 Pod 和机器上运行的容器。
- 容器运行时（如 Docker）负责从仓库中提取容器镜像，解压缩容器以及运行应用程序。

### Kubernetes 的组件

当你部署完 Kubernetes，即拥有了一个完整的集群。
一个 Kubernetes 集群由一组被称作节点的机器组成。这些节点上运行 Kubernetes 所管理的容器化应用。集群具有至少一个工作节点。
工作节点托管作为应用负载的组件的 Pod 。控制平面管理集群中的工作节点和 Pod 。 为集群提供故障转移和高可用性，这些控制平面一般跨多主机运行，集群跨多个节点运行。
现在我们来了解下交付正常运行的 Kubernetes 集群所需的各种组件。
** 控制平面组件（Control Plane Components） **
控制平面的组件对集群做出全局决策（比如调度），以及检测和响应集群事件（例如，当不满足部署的 replicas 字段时，启动新的 [pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)）。
控制平面组件可以在集群中的任何节点上运行。 然而，为了简单起见，设置脚本通常会在同一个计算机上启动所有控制平面组件， 并且不会在此计算机上运行用户容器。 请参阅[使用 kubeadm 构建高可用性集群](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/high-availability/) 中关于跨多机器控制平面设置的示例。
**kube-apiserver**
API 服务器是 Kubernetes [控制面](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-control-plane)的组件， 该组件公开了 Kubernetes API。 API 服务器是 Kubernetes 控制面的前端。
Kubernetes API 服务器的主要实现是 [kube-apiserver](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-apiserver/)。 kube-apiserver 设计上考虑了水平伸缩，也就是说，它可通过部署多个实例进行伸缩。 你可以运行 kube-apiserver 的多个实例，并在这些实例之间平衡流量。
**etcd**
etcd 是兼具一致性和高可用性的键值数据库，可以作为保存 Kubernetes 所有集群数据的后台数据库。
你的 Kubernetes 集群的 etcd 数据库通常需要有个备份计划。
**kube-scheduler**
控制平面组件，负责监视新创建的、未指定运行[节点（node）](https://kubernetes.io/zh/docs/concepts/architecture/nodes/)的 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)，选择节点让 Pod 在上面运行。
调度决策考虑的因素包括单个 Pod 和 Pod 集合的资源需求、硬件/软件/策略约束、亲和性和反亲和性规范、数据位置、工作负载间的干扰和最后时限。
**kube-controller-manager**
运行[控制器](https://kubernetes.io/zh/docs/concepts/architecture/controller/)进程的控制平面组件。
从逻辑上讲，每个[控制器](https://kubernetes.io/zh/docs/concepts/architecture/controller/)都是一个单独的进程， 但是为了降低复杂性，它们都被编译到同一个可执行文件，并在一个进程中运行。
这些控制器包括：

- 节点控制器（Node Controller）：负责在节点出现故障时进行通知和响应
- 任务控制器（Job controller）：监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
- 端点控制器（Endpoints Controller）：填充端点（Endpoints）对象（即加入 Service 与 Pod）
- 服务帐户和令牌控制器（Service Account & Token Controllers）：为新的命名空间创建默认帐户和 API 访问令牌

** cloud-controller-manager **
云控制器管理器是指嵌入特定云的控制逻辑的 [控制平面](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-control-plane)组件。 一般是云厂商来使用。

## 有趣的工作负载
工作负载是在 Kubernetes 上运行的应用程序。
无论你的负载是单一组件还是由多个一同工作的组件构成，在 Kubernetes 中你 可以在一组 [Pods](https://kubernetes.io/zh/docs/concepts/workloads/pods) 中运行它。 在 Kubernetes 中，Pod 代表的是集群上处于运行状态的一组 [容器](https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/#why-containers)。
Kubernetes Pods 有[确定的生命周期](https://kubernetes.io/zh/docs/concepts/workloads/pods/pod-lifecycle/)。 例如，当某 Pod 在你的集群中运行时，Pod 运行所在的 [节点](https://kubernetes.io/zh/docs/concepts/architecture/nodes/) 出现致命错误时， 所有该节点上的 Pods 都会失败。Kubernetes 将这类失败视为最终状态： 即使该节点后来恢复正常运行，你也需要创建新的 Pod 来恢复应用。
不过，为了让用户的日子略微好过一些，你并不需要直接管理每个 Pod。 相反，你可以使用 _负载资源_ 来替你管理一组 Pods。 这些资源配置 [控制器](https://kubernetes.io/zh/docs/concepts/architecture/controller/) 来确保合适类型的、处于运行状态的 Pod 个数是正确的，与你所指定的状态相一致。
Pod 通常不是直接创建的，而是使用工作负载资源创建的。 有关如何将 Pod 用于工作负载资源的更多信息，请参阅 [使用 Pod](https://kubernetes.io/zh/docs/concepts/workloads/pods/#working-with-pods)。
下面我们来讲 Kubernetes 内置的工作负载。
### Deployment
Deployment 非常适合管理集群上的无状态应用，Deployment 控制的所有 Pod 都是互相等价的，并可以在任何时候去动态的创建和销毁。
所谓的无状应用就是指针对每次请求的处理，不依赖其他的相关服务，并且自身也不保存任何相关的信息，而且任何时间启动的服务针对同一个请求的响应都是一致的，例如 Nginx 应用。
Deployment 为 Pod 提供了可以声明式更新的能力，你只需要负责向 Deployment 描述目标的状态，然后 Deployment 控制器就以受控速率去更改实际的状态，以使其达到描述中的期望状态。
### StatefulSet
既然有无状态应用，那么就会存在有状态的应用，例如 Mysql、Redis 等应用，那么这类的应用就应该由 StatefulSet 来管理。
StatefulSet 用来管理某 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 集合的部署和扩缩， 并为这些 Pod 提供持久存储和持久标识符。
和 [Deployment](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/) 类似， StatefulSet 管理基于相同容器规约的一组 Pod。但和 Deployment 不同的是， StatefulSet 为它们的每个 Pod 维护了一个有粘性的 ID。这些 Pod 是基于相同的规约来创建的， 但是不能相互替换：无论怎么调度，每个 Pod 都有一个永久不变的 ID。
StatefulSets 对于需要满足以下一个或多个需求的应用程序很有价值：

- 稳定的、唯一的网络标识符。
- 稳定的、持久的存储。
- 有序的、优雅的部署和缩放。
- 有序的、自动的滚动更新。

在上面描述中，“稳定的”意味着 Pod 调度或重调度的整个过程是有持久性的。 如果应用程序不需要任何稳定的标识符或有序的部署、删除或伸缩，则应该使用 由一组无状态的副本控制器提供的工作负载来部署应用程序，例如 Deployment。
### DaemonSet
由于 Deployment 是可能在同一个节点上部署同一个 Pod 多次的，但是有时候我们只希望我们的某个应用在每个节点上都只有一个 Pod 副本在执行，那就可以使用 DaemonSet 来创建 Pod。
_DaemonSet_ 确保全部（或者某些）节点上运行一个 Pod 的副本。 当有节点加入集群时， 也会为他们新增一个 Pod 。 当有节点从集群移除时，这些 Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod。
DaemonSet 的一些典型用法：

- 在每个节点上运行集群守护进程
- 在每个节点上运行日志收集守护进程
- 在每个节点上运行监控守护进程

一种简单的用法是为每种类型的守护进程在所有的节点上都启动一个 DaemonSet。 一个稍微复杂的用法是为同一种守护进程部署多个 DaemonSet；每个具有不同的标志， 并且对不同硬件类型具有不同的内存、CPU 要求。
### Job&Cron Job
还有一种情况就是比如我们需要执行一些一次性的任务，或者以某种指定的频率周期来定期的执行某些任务，那么就可以使用 Job 和 Cron Job 来管理我们的 Pod。
**Job** 会创建一个或者多个 Pods，并将继续重试 Pods 的执行，直到指定数量的 Pods 成功终止。随着 Pods 成功结束，Job 跟踪记录成功完成的 Pods 个数。 当数量达到指定的成功个数阈值时，任务（即 Job）结束。 删除 Job 的操作会清除所创建的全部 Pods。
一种简单的使用场景下，你会创建一个 Job 对象以便以一种可靠的方式运行某 Pod 直到完成。 当第一个 Pod 失败或者被删除（比如因为节点硬件失效或者重启）时，Job 对象会启动一个新的 Pod。
你也可以使用 Job 以并行的方式运行多个 Pod。
**Cron Job** 就是可以周期性重复的 ** Job， **可以参考下 Linux 中的 Crontab。 
{% note warning %}
**Caution:**
所有 **CronJob** 的 schedule: 时间都是基于 [kube-controller-manager](https://kubernetes.io/docs/reference/generated/kube-controller-manager/). 的时区。
如果你的控制平面在 Pod 或是裸容器中运行了 kube-controller-manager， 那么为该容器所设置的时区将会决定 Cron Job 的控制器所使用的时区。
{% endnote %}

## 重要的Service
我们部署的应用最终都是运行在集群 Node 中的 Pod 里，如果我们要访问我们自己的应用就不可避免的要考虑如何访问 Pod。
在 Kubernetes 中每一个 [Pod](https://kubernetes.io/zh/docs/concepts/workloads/pods/) 都有它自己的IP地址， 这就意味着你不需要显式地在 Pod 之间创建链接， 你几乎不需要处理容器端口到主机端口之间的映射。 这将形成一个干净的、向后兼容的模型；在这个模型里，从端口分配、命名、服务发现、 [负载均衡](https://kubernetes.io/zh/docs/concepts/services-networking/ingress/#load-balancing)、应用配置和迁移的角度来看， Pod 可以被视作虚拟机或者物理主机。

- 节点上的所有 Pod 之间就像局域网一样可以无障碍的进行通信
- 节点上的代理可以和节点上的所有 Pod 通信

所以简单的来说你可以通过通过 Node 本身进行请求端口转发，将请求转发到你的 Pod上就可以访问到我们的应用了，但是这时一个很不推荐的做法，因为 Pod 本身不知道 Node 的端口是什么，另外 Pod 有可能会销毁和重建，每一次销毁重建，Pod 的网络 IP 都会发生变化，因此如果通过这种方式来访问应用，就又丧失了 Kubernetes 的意义了。
### 概念
将运行在一组 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 上的应用程序公开为网络服务的抽象方法。
使用 Kubernetes，你无需修改应用程序即可使用不熟悉的服务发现机制。 Kubernetes 为 Pods 提供自己的 IP 地址，并为一组 Pod 提供相同的 DNS 名， 并且可以在它们之间进行负载均衡。
### 作用
因为 Kubernetes 会动态的来创建和销毁 Pod 以匹配集群的状态，因此 Pod 不是固定不变的，这导致了一个问题： 如果一组 Pod（称为“后端”）为集群内的其他 Pod（称为“前端”）提供功能， 那么前端如何找出并跟踪要连接的 IP 地址，以便前端可以使用提供工作负载的后端部分？
因此 Kubernetes 抽象了一种资源叫 Service 资源：逻辑上的一组 Pod，一种可以访问它们的策略 —— 通常称为微服务。Service 所针对的 Pods 集合通常是通过[选择算符](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/labels/)来确定的
举个例子，考虑一个图片处理后端，它运行了 3 个副本。这些副本是可互换的 —— 前端不需要关心它们调用了哪个后端副本。 然而组成这一组后端程序的 Pod 实际上可能会发生变化， 前端客户端不应该也没必要知道，而且也不需要跟踪这一组后端的状态。
Service 定义的抽象能够解耦这种关联。

![](./module_04_labels.svg)

Service 匹配一组 Pod 是使用 [标签(Label)和选择器(Selector)](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/labels), 它们是允许对 Kubernetes 中的对象进行逻辑操作的一种分组原语。标签(Label)是附加在对象上的键/值对，可以以多种方式使用:

- 指定用于开发，测试和生产的对象
- 嵌入版本标签
- 使用 Label 将对象进行分类

标签(Label)可以在创建时或之后附加到对象上。他们可以随时被修改。现在使用 Service 发布我们的应用程序并添加一些 Label 。

![](./module_04_labels1.svg)

## 数据的安身之所
Kubernetes 集群本身并不为你处理数据的存储，在容器中的文件在磁盘上都是临时存放的，这些数据会随着容器的销毁而一起消失。另外在同一个 Pod 中运行的多个容器也会出现多个容器需要共享文件。
为此 Kubernetes 提供了 Volume 卷这一概念来解决这两个问题。
在 Docker 中也有 Volume 卷的概念，但对它只有少量且松散的管理。 Docker 卷是磁盘上或者另外一个容器内的一个目录。
Kubernetes 支持很多类型的卷。 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 可以同时使用任意数目的卷类型。 临时卷类型的生命周期与 Pod 相同，但持久卷可以比 Pod 的存活期长。 当 Pod 不再存在时，Kubernetes 也会销毁临时卷；不过 Kubernetes 不会销毁持久卷。 对于给定 Pod 中任何类型的卷，在容器重启期间数据都不会丢失。
卷的核心是一个目录，其中可能存有数据，Pod 中的容器可以访问该目录中的数据。 所采用的特定的卷类型将决定该目录如何形成的、使用何种介质保存数据以及目录中存放的内容。
### Volume 的类型
Kuberneters 的 Volume 类型非常的丰富，在日常使用中用到最多的是下面几个类型：

- emptyDir

当 Pod 分派到某个 Node 上时，emptyDir 卷会被创建，并且在 Pod 在该节点上运行期间，卷一直存在。当 Pod 因为某些原因被从节点上删除时，emptyDir 卷中的数据也会被永久删除。
此卷一般是用来当作临时存储使用

- ConfigMap、Secret

特殊的存储类型，这是向 Pod 提供配置文件和加密文件的主要方式，[configMap](https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-pod-configmap/) 卷提供了向 Pod 注入配置数据的方法。 ConfigMap 对象中存储的数据可以被 configMap 类型的卷引用，然后被 Pod 中运行的容器化应用使用
secret 卷用来给 Pod 传递敏感信息，例如密码。你可以将 Secret 存储在 Kubernetes API 服务器上，然后以文件的形式挂在到 Pod 中，无需直接与 Kubernetes 耦合。

- persistentVolumeClaim

Kubernetes 的持久化存储卷，用来将持久卷（PersistentVolume 简称 PV）挂载到 Pod 中。
对于持久卷的申请（PersitentVolumeClaim 简称 PVC）是在用户无感知的环境下由云厂商来处理实现的。
### PV、PVC 和 StorageClass
目前关于持久化类型里在集群中使用最多的其实是动态的卷供应，由集群管理员来制定存储设备的规格，然后由云厂商来动态的提供存储设备。
动态卷供应允许按需创建存储卷。 如果没有动态供应，集群管理员必须手动地联系他们的云或存储提供商来创建新的存储卷， 然后在 Kubernetes 集群创建 [PersistentVolume对象](https://kubernetes.io/zh/docs/concepts/storage/persistent-volumes/)来表示这些卷。 动态供应功能消除了集群管理员预先配置存储的需要。 相反，它在用户请求时自动供应存储。
这个章节的知识点比较复杂，这里推荐阅读以下资料
[https://support.huaweicloud.com/basics-cce/kubernetes_0030.html](https://support.huaweicloud.com/basics-cce/kubernetes_0030.html)
[https://kubernetes.io/zh/docs/concepts/storage/dynamic-provisioning/](https://kubernetes.io/zh/docs/concepts/storage/dynamic-provisioning/)
## ConfigMap&Secret
### ConfigMap
ConfigMap 是一种 API 对象，用来将非机密性的数据保存到键值对中。使用时， [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 可以将其用作环境变量、命令行参数或者存储卷中的配置文件。
ConfigMap 将你的环境配置信息和 [容器镜像](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-image) 解耦，便于应用配置的修改。
比如，假设你正在开发一个应用，它可以在你自己的电脑上（用于开发）和在云上 （用于实际流量）运行。 你的代码里有一段是用于查看环境变量 DATABASE_HOST，在本地运行时， 你将这个变量设置为 localhost，在云上，你将其设置为引用 Kubernetes 集群中的 公开数据库组件的 [服务](https://kubernetes.io/zh/docs/concepts/services-networking/service/)。
这让你可以获取在云中运行的容器镜像，并且如果有需要的话，在本地调试完全相同的代码。
ConfigMap 在设计上不是用来保存大量数据的。在 ConfigMap 中保存的数据不可超过 1 MiB。如果你需要保存超出此尺寸限制的数据，你可能希望考虑挂载存储卷 或者使用独立的数据库或者文件服务。
### Secret
Secret 是一种包含少量敏感信息例如密码、令牌或密钥的对象。 这样的信息可能会被放在 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 规约中或者镜像中。 使用 Secret 意味着你不需要在应用程序代码中包含机密数据。
由于创建 Secret 可以独立于使用它们的 Pod， 因此在创建、查看和编辑 Pod 的工作流程中暴露 Secret（及其数据）的风险较小。 Kubernetes 和在集群中运行的应用程序也可以对 Secret 采取额外的预防措施， 例如避免将机密数据写入非易失性存储。
Secret 类似于 [ConfigMap](https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-pod-configmap/) 但专门用于保存机密数据。
Pod 可以用三种方式之一来使用 Secret：

- 作为挂载到一个或多个容器上的[卷](https://kubernetes.io/zh/docs/concepts/storage/volumes/) 中的[文件](https://kubernetes.io/zh/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod)。
- 作为[容器的环境变量](https://kubernetes.io/zh/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)。
- 由 [kubelet 在为 Pod 拉取镜像时使用](https://kubernetes.io/zh/docs/concepts/configuration/secret/#using-imagepullsecrets)。

Kubernetes 控制面也使用 Secret； 例如，[引导令牌 Secret](https://kubernetes.io/zh/docs/concepts/configuration/secret/#bootstrap-token-secrets) 是一种帮助自动化节点注册的机制。

本篇全都是概念性的讲解，可能会有些枯燥，但这也是必经之路没有办法绕过，后续我会更新入门操作篇，后续再会。