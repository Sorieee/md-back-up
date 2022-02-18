https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/

# 概览

## 何为Kubernetes

​	Kubernetes 是一个可移植、可扩展的开源平台，用于管理容器化的工作负载和服务，可促进声明式配置和自动化。

​	缩写为K8S。

## 时光倒流

![Deployment evolution](https://d33wubrfki0l68.cloudfront.net/26a177ede4d7b032362289c6fccd448fc4a91174/eb693/images/docs/container_evolution.svg)

容器之所以流行，是因为它们提供了额外的好处，例如：

* 敏捷的应用程序创建和部署：与使用 VM 映像相比，容器映像创建的简便性和效率更高。
* 持续开发、集成和部署：提供可靠且频繁的容器映像构建和部署以及快速高效的回滚（由于映像不变性）。
* Dev 和 Ops 的关注点分离：在构建/发布时而不是部署时创建应用程序容器映像，从而将应用程序与基础架构解耦。
* 可观察性：不仅可以显示操作系统级别的信息和指标，还可以显示应用程序运行状况和其他信号。
* 开发、测试和生产之间的环境一致性：在笔记本电脑上运行与在云中运行相同。
* 云和操作系统分发可移植性：在 Ubuntu、RHEL、CoreOS、本地、主要公共云和其他任何地方运行。
* 以应用程序为中心的管理：将抽象级别从在虚拟硬件上运行操作系统提高到使用逻辑资源在操作系统上运行应用程序。
* 松散耦合、分布式、弹性、自由的微服务：应用程序被分解成更小的、独立的部分，并且可以动态部署和管理——而不是在一台大型单一用途机器上运行的单一堆栈。
* 资源隔离：可预测的应用程序性能。
* 资源利用：高效率、高密度。

## 为啥需要K8s以及他可以做什么

K8s提供:

* **服务发现和负载平衡** Kubernetes 可以使用 DNS 名称或使用自己的 IP 地址公开容器。 如果容器的流量很高，Kubernetes 能够负载均衡和分配网络流量，从而使部署稳定。
* **存储编排** K8s允许自动挂在你说选择的存储系统，比如本地存储，开放的云存储等等。
* **自动推出和回滚** 您可以使用 Kubernetes 描述已部署容器的所需状态，并且它可以以受控的速率将实际状态更改为所需状态。 例如，您可以自动化 Kubernetes 为您的部署创建新容器、删除现有容器并将其所有资源用于新容器。
* **自定打包二进制** 您为 Kubernetes 提供了一个节点集群，可用于运行容器化任务。 你告诉 Kubernetes 每个容器需要多少 CPU 和内存 (RAM)。 Kubernetes 可以将容器安装到您的节点上，以充分利用您的资源。
* **自我修复**  Kubernetes 会重启失败的容器、替换容器、杀死不响应用户定义的健康检查的容器，并且在它们准备好服务之前不会将它们通告给客户端。
* **隐私和配置管理** Kubernetes 允许您存储和管理敏感信息，例如密码、OAuth 令牌和 SSH 密钥。 您可以部署和更新机密和应用程序配置，而无需重新构建容器映像，也无需在堆栈配置中公开机密。

## What Kubernetes is not

* 不限制支持的应用程序类型。 Kubernetes 旨在支持极其多样化的工作负载，包括无状态、有状态和数据处理工作负载。 如果应用程序可以在容器中运行，它应该在 Kubernetes 上运行良好。
* 不部署源代码，也不构建您的应用程序。 持续集成、交付和部署 (CI/CD) 工作流程由组织文化和偏好以及技术要求决定。
* 不提供应用级服务，例如中间件（例如，消息总线）、数据处理框架（例如，Spark）、数据库（例如，MySQL）、缓存，也不提供集群存储系统（例如，Ceph） 作为内置服务。 此类组件可以在 Kubernetes 上运行，和/或可以由运行在 Kubernetes 上的应用程序通过可移植机制（例如 Open Service Broker）访问。
* 不规定日志记录、监控或警报解决方案。 它提供了一些集成作为概念证明，以及收集和导出指标的机制。
* 不提供也不强制要求配置语言/系统（例如 Jsonnet）。 它提供了一个声明性 API，可以针对任意形式的声明性规范。
* 不提供也不采用任何全面的机器配置、维护、管理或自我修复系统。
* 此外，Kubernetes 不仅仅是一个编排系统。 事实上，它消除了编排的需要。 编排的技术定义是执行已定义的工作流：首先执行 A，然后执行 B，然后执行 C。相比之下，Kubernetes 包含一组独立的、可组合的控制过程，它们不断地将当前状态驱动到提供的所需状态。 从 A 到 C 的方式无关紧要。也不需要集中控制。 这导致了一个更易于使用、更强大、更健壮、弹性和可扩展的系统。

## K8s组件

​	K8s集群由一组工作机器构成，叫做 [nodes](https://kubernetes.io/docs/concepts/architecture/nodes/), 用来运行容器化应用。每个集群至少有一个工作节点。

​	工作节点托管作为应用程序工作负载组件的 Pod。 控制平面管理集群中的工作节点和 Pod。 在生产环境中，控制平面通常跨多台计算机运行，集群通常运行多个节点，提供容错和高可用性。

![](https://pic.imgdb.cn/item/620f60892ab3f51d91d8d86a.jpg)

### 控制面板组件

​	控制平面的组件对集群做出全局决策（例如，调度），以及检测和响应集群事件（例如，当部署的副本字段不满足时启动新的 Pod）。

​	控制平面组件可以在集群中的任何机器上运行。 但是，为简单起见，设置脚本通常在同一台机器上启动所有控制平面组件，并且不在这台机器上运行用户容器。 有关跨多个 VM 运行的示例控制平面设置，请参阅使用 kubeadm 创建高可用性集群  [Creating Highly Available clusters with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)。

#### kube-apiserver

​	API 服务器是 Kubernetes 控制面板的一个组件，它公开了 Kubernetes API。 API 服务器是 Kubernetes 控制面板的前端。

​	Kubernetes API 服务器的主要实现是 kube-apiserver。 kube-apiserver 旨在水平扩展——也就是说，它通过部署更多实例来扩展。 您可以运行多个 kube-apiserver 实例并平衡这些实例之间的流量。

#### etcd

​	一致且高度可用的键值存储，用作 Kubernetes 的所有集群数据的后备存储。

​	如果您的 Kubernetes 集群使用 etcd 作为其后备存储，请确保您为这些数据制定了备份计划。

​	您可以在官方文档中找到有关 etcd 的深入信息。

#### kube-scheduler

​	控制平面组件，用于监视没有分配节点的新创建的 Pod，并选择一个节点供它们运行。

​	调度决策考虑的因素包括：个人和集体资源需求、硬件/软件/策略约束、亲和性和反亲和性规范、数据局部性、工作负载间干扰和截止日期。

#### kube-controller-manager

​	运行控制器进程的控制面板组件。

​	从逻辑上讲，每个控制器都是一个单独的进程，但为了降低复杂性，它们都被编译成一个二进制文件并在一个进程中运行。

​	这些控制器的一部分类型是：

* 节点控制器：负责在节点宕机时进行通知和响应。
* 作业控制器：监视代表一次性任务的作业对象，然后创建 Pod 以运行这些任务以完成。
* Endpoints 控制器：填充 Endpoints 对象（即加入 Services & Pods）。
* 服务帐户和令牌控制器：为新命名空间创建默认帐户和 API 访问令牌

### cloud-controller-manager

​	一个 Kubernetes 控制面板组件，它嵌入了特定于云的控制逻辑。云控制器管理器允许您将集群链接到云提供商的 API，并将与该云平台交互的组件与仅与您的集群交互的组件分开。

​	 cloud-controller-manager 仅运行特定于您的云提供商的控制器。 如果您在自己的场所运行 Kubernetes，或者在您自己 PC 内的学习环境中运行 Kubernetes，则集群没有云控制器管理器。

​	 与 kube-controller-manager 一样，cloud-controller-manager 将几个逻辑上独立的控制循环组合成一个二进制文件，您可以将其作为单个进程运行。 您可以水平扩展（运行多个副本）以提高性能或帮助容忍故障。

以下控制器可以具有云提供商依赖项：

* 节点控制器：用于检查云提供商以确定节点停止响应后是否已在云中删除。
* 路由控制器：用于在底层云基础设施中设置路由
* 服务控制器：用于创建、更新和删除云提供商负载均衡器

### Node组件

​	节点组件在每个节点上运行，维护运行的 pod 并提供 Kubernetes 运行时环境。

#### kubelet

​	在集群中的每个节点上运行的代理。 它确保容器在 Pod 中运行。

​	kubelet 采用一组通过各种机制提供的 PodSpec，并确保这些 PodSpec 中描述的容器运行且健康。 kubelet 不管理不是由 Kubernetes 创建的容器。

#### kube-proxy

​	kube-proxy 是一个网络代理，运行在集群中的每个节点上，实现了 Kubernetes 服务概念的一部分。

​	kube-proxy 在节点上维护网络规则。 这些网络规则允许从集群内部或外部的网络会话与 Pod 进行网络通信。

​	kube-proxy 使用操作系统包过滤层（如果有并且可用）。 否则，kube-proxy 会自行转发流量。

#### Container runtime

​	容器运行时是负责运行容器的软件。

​	Kubernetes 支持容器运行时，例如 containerd、CRI-O 以及 Kubernetes CRI（容器运行时接口）的任何其他实现。

### 插件

​	插件使用 Kubernetes 资源（DaemonSet、Deployment 等）来实现集群功能。 因为这些提供集群级别的功能，所以插件的命名空间资源属于 kube-system 命名空间。

​	选定的插件如下所述； 有关可用插件的扩展列表，请参阅插件[Addons](https://kubernetes.io/docs/concepts/cluster-administration/addons/).

### DNS

​	虽然其他插件不是严格要求的，但所有 Kubernetes 集群都应该有集群 DNS，因为许多示例都依赖它。

​	除了您环境中的其他 DNS 服务器之外，集群 DNS 是一个 DNS 服务器，它为 Kubernetes 服务提供 DNS 记录。

​	由 Kubernetes 启动的容器会在其 DNS 搜索中自动包含此 DNS 服务器。

#### Web UI (Dashboard)

​	Dashboard 是用于 Kubernetes 集群的通用、基于 Web 的 UI。 它允许用户对集群中运行的应用程序以及集群本身进行管理和故障排除。

#### Container Resource Monitoring

​	Container Resource Monitoring 在中央数据库中记录有关容器的通用时间序列指标，并提供用于浏览该数据的 UI。

#### Cluster-level Logging

​	集群级别的日志机制负责将容器日志保存到具有搜索/浏览界面的中央日志存储中。

# The Kubernetes API

