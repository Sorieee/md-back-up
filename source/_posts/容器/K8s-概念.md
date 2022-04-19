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

- **服务发现和负载均衡**

  Kubernetes 可以使用 DNS 名称或自己的 IP 地址公开容器，如果进入容器的流量很大， Kubernetes 可以负载均衡并分配网络流量，从而使部署稳定。

- **存储编排**

  Kubernetes 允许你自动挂载你选择的存储系统，例如本地存储、公共云提供商等。

- **自动部署和回滚**

  你可以使用 Kubernetes 描述已部署容器的所需状态，它可以以受控的速率将实际状态 更改为期望状态。例如，你可以自动化 Kubernetes 来为你的部署创建新容器， 删除现有容器并将它们的所有资源用于新容器。

- **自动完成装箱计算**

  Kubernetes 允许你指定每个容器所需 CPU 和内存（RAM）。 当容器指定了资源请求时，Kubernetes 可以做出更好的决策来管理容器的资源。

- **自我修复**

  Kubernetes 重新启动失败的容器、替换容器、杀死不响应用户定义的 运行状况检查的容器，并且在准备好服务之前不将其通告给客户端。

- **密钥与配置管理**

  Kubernetes 允许你存储和管理敏感信息，例如密码、OAuth 令牌和 ssh 密钥。 你可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。

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

## 持久化

​	Kubernetes 通过将对象的序列化状态写入 etcd 来存储它们。 

## API和版本

​	为了更容易消除字段或重组资源表示，Kubernetes 支持多个 API 版本，每个版本位于不同的 API 路径，例如 `/api/v1` 或 ``/apis/rbac.authorization.k8s.io/v1alpha1`。

​	版本控制是在 API 级别而不是在资源或字段级别完成的，以确保 API 呈现清晰、一致的系统资源和行为视图，并能够控制对报废和/或实验 API 的访问。

​	为了更容易发展和扩展其 API，Kubernetes 实现了可以启用或禁用的 API 组。

​	To make it easier to evolve and to extend its API, Kubernetes implements [API groups](https://kubernetes.io/docs/reference/using-api/#api-groups) that can be [enabled or disabled](https://kubernetes.io/docs/reference/using-api/#enabling-or-disabling).

​	API 资源通过其 API 组、资源类型、命名空间（用于命名空间资源）和名称来区分。 API 服务器透明地处理 API 版本之间的转换：所有不同的版本实际上是相同持久数据的表示。 API 服务器可以通过多个 API 版本提供相同的底层数据。

​	例如，假设同一个资源有两个 API 版本，v1 和 v1beta1。 如果您最初使用其 API 的 v1beta1 版本创建了一个对象，您以后可以使用 v1beta1 或 v1 API 版本读取、更新或删除该对象。

## API变更

​	任何成功的系统都需要随着新用例的出现或现有用例的变化而发展和改变。 因此，Kubernetes 将 Kubernetes API 设计为不断变化和成长。 Kubernetes 项目旨在不破坏与现有客户端的兼容性，并在一段时间内保持这种兼容性，以便其他项目有机会适应。

​	一般来说，新的 API 资源和新的资源字段可以经常和频繁地添加。 消除资源或字段需要遵循 API 弃用政策。

​	一旦官方 Kubernetes API 达到普遍可用性 (GA)，Kubernetes 做出了坚定的承诺，通常在 API 版本 v1 上保持它们的兼容性。 此外，即使在可行的情况下，Kubernetes 也保持对 beta API 版本的兼容性：如果您采用 beta API，您可以继续使用该 API 与您的集群进行交互，即使在功能稳定之后也是如此。

> 注意：虽然 Kubernetes 还旨在维护 alpha API 版本的兼容性，但在某些情况下这是不可能的。 如果您使用任何 alpha API 版本，请在升级集群时查看 Kubernetes 的发行说明，以防 API 发生更改。

​	Refer to [API versions reference](https://kubernetes.io/docs/reference/using-api/#api-versioning) for more details on the API version level definitions

## API扩展

The Kubernetes API can be extended in one of two ways:

1. [Custom resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) let you declaratively define how the API server should provide your chosen resource API.
2. You can also extend the Kubernetes API by implementing an [aggregation layer](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/).

# Working with Kubernetes Objects

​		本页介绍了 Kubernetes 对象如何在 Kubernetes API 中表示，以及如何以 .yaml 格式表示它们。

## Understanding Kubernetes Objects

​	Kubernetes 对象是 Kubernetes 系统中的持久实体。 Kubernetes 使用这些实体来表示集群的状态。 具体来说，他们可以描述：

* 正在运行哪些容器化应用程序（以及在哪些节点上）。
* 这些应用程序可用的资源。
* 有关这些应用程序行为方式的策略，例如重启策略、升级和容错。

### Object Spec and Status

​	几乎每个 Kubernetes 对象都包含两个管理对象配置的嵌套对象字段：对象`spec`和对象`status`。 对于具有`spec`的对象，您必须在创建对象时进行设置，提供您希望资源具有的特征的描述：其所需的状态。

​	`status`描述对象的当前状态，由 Kubernetes 系统及其组件提供和更新。 Kubernetes 控制平面持续主动地管理每个对象的实际状态，以匹配您提供的所需状态。

​	例如：在 Kubernetes 中，Deployment 是一个对象，可以代表在您的集群上运行的应用程序。 创建部署时，您可以设置部署规范以指定要运行应用程序的三个副本。 Kubernetes 系统读取部署规范并启动所需应用程序的三个实例——更新状态以匹配您的规范。 如果这些实例中的任何一个失败（状态更改），Kubernetes 系统会通过进行更正来响应规范和状态之间的差异——在这种情况下，启动一个替换实例。

​	For more information on the object spec, status, and metadata, see the [Kubernetes API Conventions](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md).

### Describing a Kubernetes object

​	在 Kubernetes 中创建对象时，必须提供描述其所需状态的对象规范，以及有关对象的一些基本信息（例如名称）。 当您使用 Kubernetes API 创建对象（直接或通过 kubectl）时，该 API 请求必须将该信息作为 JSON 包含在请求正文中。 大多数情况下，您在 .yaml 文件中向 kubectl 提供信息。 kubectl 在发出 API 请求时将信息转换为 JSON。

​	Here's an example `.yaml` file that shows the required fields and object spec for a Kubernetes Deployment:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

​	使用上述 .yaml 文件创建部署的一种方法是在 kubectl 命令行界面中使用 kubectl apply 命令，将 .yaml 文件作为参数传递。 这是一个例子：

```shell
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
```

The output is similar to this:

```
deployment.apps/nginx-deployment created
```

### Required Fields

在您要创建的 Kubernetes 对象的 .yaml 文件中，您需要为以下字段设置值：

* `apiVersion` - Which version of the Kubernetes API you're using to create this object
* `kind` - What kind of object you want to create
* `metadata` - Data that helps uniquely identify the object, including a `name` string, `UID`, and optional `namespace`
* `spec` - What state you desire for the object



​	每个 Kubernetes 对象的对象规范的精确格式都不同，并且包含特定于该对象的嵌套字段。 Kubernetes API  [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)参考可以帮助您找到可以使用 Kubernetes 创建的所有对象的规范格式。

​	例如，Pod 的参考详细说明了 API 中 Pod 的 spec 字段，而 Deployment 的参考详细说明了 Deployment 的 spec 字段。 在这些 API 参考页面中，您会看到对 PodSpec 和 DeploymentSpec 的提及。 这些名称是 Kubernetes 用来实现其 API 的 Golang 代码的实现细节。

## Kubernetes Object Management

​	kubectl 命令行工具支持多种不同的方式来创建和管理 Kubernetes 对象。 本文档概述了不同的方法。 阅读 Kubectl 书籍，了解 Kubectl 管理对象的详细信息。

### Management techniques

> 警告：应该只使用一种技术来管理 Kubernetes 对象。 同一对象的混合和匹配技术会导致未定义的行为。

| Management technique             | 操作对象             | 推荐环境             | Supported writers | 学习曲线 |
| -------------------------------- | -------------------- | -------------------- | ----------------- | -------- |
| Imperative commands              | Live objects         | Development projects | 1+                | Lowest   |
| Imperative object configuration  | Individual files     | Production projects  | 1                 | Moderate |
| Declarative object configuration | Directories of files | Production projects  | 1+                | Highest  |

### Imperative commands

​	使用imperative commands时，用户直接对集群中的活动对象进行操作。 用户将操作作为参数或标志提供给 kubectl 命令。

​	这是开始或在集群中运行一次性任务的推荐方式。 因为这种技术直接对活动对象进行操作，所以它不提供以前配置的历史记录。

#### Examples

Run an instance of the nginx container by creating a Deployment object:

```sh
kubectl create deployment nginx --image nginx
```

#### 对比

与对象配置相比的优势：

* 命令表示为单个动作词。
* 命令只需要一个步骤即可对集群进行更改。

与对象配置相比的劣势：

* 命令不与变更审查流程集成。
* 命令不提供与更改关联的审计跟踪。
* 命令不提供记录的来源，除了实时的。
* 命令不提供用于创建新对象的模板。

### Imperative object configuration

​	在命令式对象配置中，kubectl 命令指定操作（创建、替换等）、可选标志和至少一个文件名。 指定的文件必须包含 YAML 或 JSON 格式的对象的完整定义。

​	See the [API reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/) for more details on object definitions.

> 警告：命令`replace`命令用新提供的规范替换现有规范，删除配置文件中缺少的对象的所有更改。 此方法不应用于规范独立于配置文件更新的资源类型。 例如，LoadBalancer 类型的服务的 externalIPs 字段的更新独立于集群的配置。

#### Examples

Create the objects defined in a configuration file:

```sh
kubectl create -f nginx.yaml
```

Delete the objects defined in two configuration files:

```sh
kubectl delete -f nginx.yaml -f redis.yaml
```

Update the objects defined in a configuration file by overwriting the live configuration:

```sh
kubectl replace -f nginx.yaml
```

#### 对比

与命令式命令相比的优点：

* 对象配置可以存储在源代码控制系统中，例如 Git。
* 对象配置可以与流程集成，例如在推送和审计跟踪之前审查更改。
* 对象配置提供了创建新对象的模板。

与命令式命令相比的缺点：

* 对象配置需要对对象模式有基本的了解:

* 对象配置需要额外的编写 YAML 文件的步骤。

与声明性对象配置相比的优势：

* 命令式对象配置行为更简单、更容易理解。
* 从 Kubernetes 1.5 版本开始，命令式对象配置更加成熟。

与声明性对象配置相比的缺点：

* 命令式对象配置最适用于文件，而不是目录。
* 对活动对象的更新必须反映在配置文件中，否则它们将在下次替换时丢失。

### Declarative object configuration

​	当使用声明性对象配置时，用户对存储在本地的对象配置文件进行操作，但是用户没有定义要对文件进行的操作。 kubectl 会自动检测每个对象的创建、更新和删除操作。 这允许在目录上工作，不同的对象可能需要不同的操作。

> 注意：声明性对象配置保留其他编写者所做的更改，即使这些更改没有合并回对象配置文件。 这可以通过使用补丁 API 操作来仅写入观察到的差异，而不是使用替换 API 操作来替换整个对象配置。

### Examples

​	Process all object configuration files in the `configs` directory, and create or patch the live objects. You can first `diff` to see what changes are going to be made, and then apply:

```sh
kubectl diff -f configs/
kubectl apply -f configs/
```

​	Recursively process directories:

```sh
kubectl diff -R -f configs/
kubectl apply -R -f configs/
```

#### 对比

与命令式对象配置相比的优势:

* 直接对活动对象所做的更改会被保留，即使它们没有合并回配置文件。
* 声明式对象配置更好地支持对目录进行操作并自动检测每个对象的操作类型（创建、修补、删除）

与命令式对象配置相比的缺点：

* 声明性对象配置在意外时更难调试和理解结果。
* 使用差异的部分更新会创建复杂的合并和修补操作。

## Object Names and IDs

​	集群中的每个对象都有一个对该类型资源唯一的名称。 每个 Kubernetes 对象还有一个在整个集群中唯一的 UID。

​	例如，在同一个命名空间中只能有一个名为 myapp-1234 的 Pod，但可以有一个 Pod 和一个 Deployment，每个都命名为 myapp-1234。

​	For non-unique user-provided attributes, Kubernetes provides [labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) and [annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/).

### Names

​	略。

### DNS Subdomain Names

大多数资源类型需要一个可用作 RFC 1123 中定义的 DNS 子域名的名称。这意味着该名称必须：

* 包含不超过 253 个字符
* 仅包含小写字母数字字符、“-”或“.”
* 以字母数字字符开头
* 以字母数字字符结尾

### RFC 1123 Label Names

某些资源类型要求其名称遵循 RFC 1123 中定义的 DNS 标签标准。这意味着名称必须：

* 最多包含 63 个字符
* 仅包含小写字母数字字符或“-”
* 以字母数字字符开头
* 以字母数字字符结尾

### RFC 1035 Label Names

某些资源类型要求其名称遵循 RFC 1035 中定义的 DNS 标签标准。这意味着名称必须：

* 最多包含 63 个字符
* 仅包含小写字母数字字符或“-”
* 以字母字符开头
* 以字母数字字符结尾

### Path Segment Names

​	某些资源类型要求它们的名称能够安全地编码为path segment。 换句话说，名称可能不是“.” 或“..”，并且名称不能包含“/”或“%”。

​	这是一个名为 nginx-demo 的 Pod 的示例清单。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

> **Note:** Some resource types have additional restrictions on their names.

### UIDs

​	A Kubernetes systems-generated string to uniquely identify objects.

​	Every object created over the whole lifetime of a Kubernetes cluster has a distinct UID. It is intended to distinguish between historical occurrences of similar entities.

​	Kubernetes UIDs are universally unique identifiers (also known as UUIDs). UUIDs are standardized as ISO/IEC 9834-8 and as ITU-T X.667

## Namespaces

​	在 Kubernetes 中，命名空间提供了一种在单个集群中隔离资源组的机制。 资源名称在命名空间内必须是唯一的，但跨命名空间不需要。 基于命名空间的作用域仅适用于命名空间对象（例如部署、服务等），不适用于集群范围的对象（例如 StorageClass、Nodes、PersistentVolumes 等）。

### When to Use Multiple Namespaces

​	略。

### Working with Namespaces

​	Creation and deletion of namespaces are described in the [Admin Guide documentation for namespaces](https://kubernetes.io/docs/tasks/administer-cluster/namespaces).

> **Note:** Avoid creating namespaces with the prefix `kube-`, since it is reserved for Kubernetes system namespaces.

#### Viewing namespaces

You can list the current namespaces in a cluster using:

```shell
kubectl get namespace
NAME              STATUS   AGE
default           Active   1d
kube-node-lease   Active   1d
kube-public       Active   1d
kube-system       Active   1d
```

Kubernetes starts with four initial namespaces:

* `default` The default namespace for objects with no other namespace
* `kube-system` The namespace for objects created by the Kubernetes system
* `kube-public` This namespace is created automatically and is readable by all users (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.
* `kube-node-lease` This namespace holds [Lease](https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/lease-v1/) objects associated with each node. Node leases allow the kubelet to send [heartbeats](https://kubernetes.io/docs/concepts/architecture/nodes/#heartbeats) so that the control plane can detect node failure.

### Setting the namespace for a request

To set the namespace for a current request, use the `--namespace` flag.

For example:

```shell
kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>
kubectl get pods --namespace=<insert-namespace-name-here>
```

### Setting the namespace preference

You can permanently save the namespace for all subsequent kubectl commands in that context.

```shell
kubectl config set-context --current --namespace=<insert-namespace-name-here>
# Validate it
kubectl config view --minify | grep namespace:
```

### Namespaces and DNS

​	When you create a [Service](https://kubernetes.io/docs/concepts/services-networking/service/), it creates a corresponding [DNS entry](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/). This entry is of the form `<service-name>.<namespace-name>.svc.cluster.local`, which means that if a container only uses `<service-name>`, it will resolve to the service which is local to a namespace. This is useful for using the same configuration across multiple namespaces such as Development, Staging and Production. If you want to reach across namespaces, you need to use the fully qualified domain name (FQDN).

​	As a result, all namespace names must be valid [RFC 1123 DNS labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names).

> **警告**： 通过创建与公共顶级域同名的命名空间，这些命名空间中的服务可以具有与公共 DNS 记录重叠的短 DNS 名称。 来自任何名称空间的工作负载执行不带尾随点的 DNS 查找，将被重定向到这些服务，优先于公共 DNS。 为了缓解这种情况，请将创建命名空间的权限限制为受信任的用户。 如果需要，您可以额外配置第三方安全控制，例如准入 webhook，以阻止使用公共 TLD 名称创建任何命名空间。 

### Not All Objects are in a Namespace

​	Most Kubernetes resources (e.g. pods, services, replication controllers, and others) are in some namespaces. However namespace resources are not themselves in a namespace. And low-level resources, such as [nodes](https://kubernetes.io/docs/concepts/architecture/nodes/) and persistentVolumes, are not in any namespace.

To see which Kubernetes resources are and aren't in a namespace:

```shell
# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false
```

### Automatic labelling

**FEATURE STATE:** `Kubernetes 1.21 [beta]`

​	The Kubernetes control plane sets an immutable [label](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels) `kubernetes.io/metadata.name` on all namespaces, provided that the `NamespaceDefaultLabelName` [feature gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) is enabled. The value of the label is the namespace name.

## Labels and Selectors

​	标签是附加到对象（例如 pod）的键/值对。 标签旨在用于指定对用户有意义且相关的对象的标识属性，但不直接暗示核心系统的语义。 标签可用于组织和选择对象的子集。 标签可以在创建时附加到对象上，随后可以随时添加和修改。 每个对象都可以定义一组键/值标签。 对于给定的对象，每个 Key 必须是唯一的。

```json
"metadata": {
  "labels": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```

​	Labels allow for efficient queries and watches and are ideal for use in UIs and CLIs. Non-identifying information should be recorded using [annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)

### Motivation

标签使用户能够以松散耦合的方式将他们自己的组织结构映射到系统对象上，而无需客户端存储这些映射。

服务部署和批处理管道通常是多维实体（例如，多个分区或部署、多个发布轨道、多个层、每层多个微服务）。管理通常需要横切操作，这打破了严格层次表示的封装，特别是由基础设施而不是用户确定的刚性层次。

示例标签：

* “发布”：“稳定”，“发布”：“金丝雀”
* “环境”：“开发”，“环境”：“qa”，“环境”：“生产”
* “层”：“前端”，“层”：“后端”，“层”：“缓存”
* “分区”：“客户A”，“分区”：“客户B”
* “轨道”：“每日”，“轨道”：“每周”



​	这些是常用标签的示例；您可以自由开发自己的约定。请记住，标签 Key 对于给定对象必须是唯一的。

### Syntax and character set

​	标签是键值对。 有效的标签键有两个部分：可选的前缀和名称，由斜杠 (`/`) 分隔。 名称段是必需的，并且必须为 63 个字符或更少，以字母数字字符 (`[a-z0-9A-Z]`) 开头和结尾，并带有破折号 (`-`)、下划线 (`_`)、圆点 (`.`) 和字母数字. 前缀是可选的。 如果指定，前缀必须是 DNS 子域：由点 (`.`) 分隔的一系列 DNS 标签，总共不超过 253 个字符，后跟斜杠 (`/`)。

​	如果省略了前缀，则假定标签密钥对用户是私有的。 向最终用户对象添加标签的自动化系统组件（例如 kube-scheduler、kube-controller-manager、kube-apiserver、kubectl 或其他第三方自动化）必须指定前缀。

​	The `kubernetes.io/` and `k8s.io/` prefixes are [reserved](https://kubernetes.io/docs/reference/labels-annotations-taints/) for Kubernetes core components.

Valid label value:

* must be 63 characters or less (can be empty),
* unless empty, must begin and end with an alphanumeric character (`[a-z0-9A-Z]`),
* could contain dashes (`-`), underscores (`_`), dots (`.`), and alphanumerics between.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

### Label selectors

​	Unlike [names and UIDs](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/), labels do not provide uniqueness. In general, we expect many objects to carry the same label(s).

​	Via a *label selector*, the client/user can identify a set of objects. The label selector is the core grouping primitive in Kubernetes.

​	API 目前支持两种类型的选择器：基于相等和基于集合。 标签选择器可以由逗号分隔的多个要求组成。 在多个要求的情况下，必须满足所有要求，因此逗号分隔符充当逻辑 AND (&&) 运算符。

#### *Equality-based* requirement

​	Three kinds of operators are admitted `=`,`==`,`!=`. The first two represent *equality* (and are synonyms), while the latter represents *inequality*. For example:

```
environment = production
tier != frontend
```

One usage scenario for equality-based label requirement is for Pods to specify node selection criteria. For example, the sample Pod below selects nodes with the label "`accelerator=nvidia-tesla-p100`".

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

### *Set-based* requirement

​	*Set-based* label requirements allow filtering keys according to a set of values. Three kinds of operators are supported: `in`,`notin` and `exists` (only the key identifier). For example:

```
environment in (production, qa)
tier notin (frontend, backend)
partition
!partition
```

​	Similarly the comma separator acts as an *AND* operator. So filtering resources with a `partition` key (no matter the value) and with `environment` different than `qa` can be achieved using `partition,environment notin (qa)`. The *set-based* label selector is a general form of equality since `environment=production` is equivalent to `environment in (production)`; similarly for `!=` and `notin`.

​	*Set-based* requirements can be mixed with *equality-based* requirements. For example: `partition in (customerA, customerB),environment!=qa`.

### API

#### LIST and WATCH filtering

​	LIST 和 WATCH 操作可以指定标签选择器来过滤使用查询参数返回的对象集。 这两个要求都是允许的（此处显示它们将出现在 URL 查询字符串中）：

* *equality-based* requirements: `?labelSelector=environment%3Dproduction,tier%3Dfrontend`
* *set-based* requirements: `?labelSelector=environment+in+%28production%2Cqa%29%2Ctier+in+%28frontend%29`



​	两种标签选择器样式都可用于通过 REST 客户端列出或查看资源。 例如，使用 kubectl 定位 apiserver 并使用基于等式的可能会这样写：

