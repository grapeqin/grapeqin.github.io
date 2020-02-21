## 概览

Docker是一个开发、运输和运行应用程序的开放平台。Docker允许从基础设施中分离你的应用程序以便你能快速交付应用。通过Docker，你可以像管理应用一样管理基础设施。利用Docker快速运输、测试和部署代码的方法论，你能显著减少从写代码到让它运行在线上的延迟时间。

### Docker平台

Docker提供了打包并运行应用程序的能力，这种能力是通过容器这种松散隔离的环境来实现的。它的隔离性和安全性允许你在一台宿主机上同时运行许多容器。容器直接运行在宿主机操作系统的内核之上，不需要承担hypervisor的额外负载，所以说它是轻量级的。这意味着在同样的硬件条件下Docker相比虚拟机能运行更多的容器。甚至可以直接在虚拟机上运行Docker容器。

Docker提供了一系列工具和平台来帮你管理容器的生命周期：

- 使用容器开发应用程序和它的支持组件。
- 容器是分发和测试应用程序的最小单元
- 将容器或编排好的服务部署到生产环境。不论你的生产环境是私有云，还是公有云，还是混合云，应用程序的表现都不会改变。

### Docker引擎

Docker引擎是一个C/S架构的应用，它包含这些主要的组件：

- 守护进程：一个长期运行的程序(`dockerd`命令)
- REST API：让程序和守护进程对话的入口，同时告诉守护进程应该执行什么动作
- CLI：命令行接口客户端

![Docker Engine Components Flow](https://docs.docker.com/engine/images/engine-components-flow.png)

CLI借助脚本或命令使用Docker REST API来控制并与守护进程交互。许多其他的Docker应用都是使用底层的API和CLI。

守护进程创建并管理*Docker对象，比如镜像、容器、网络和数据卷*。

要获取更多信息，请参考下面的[Docker架构]()

### 使用Docker能做什么

#### 应用的快速、一致交付

Docker允许开发者通过容器提供的应用和服务在标准的环境中工作来将开发流程化。在持续集成/持续交付领域容器具有天然的优势。

考虑下面这个示例场景：

- 开发人员本地编码并通过容器与他的同事协作
- 他们利用Docker将应用推送到测试环境、自动启动并完成测试
- 开发人员发现bug，在开发环境中迅速修复并重新部署到测试环境以便继续测试和验证
- 测试通过后，为客户修复问题会跟把修改后的镜像推送到生产环境一样简单

#### 响应式部署和可扩展性

Docker作为容器化的平台可移植性高。Docker容器可以运行在本地实验台上，数据中心的物理机或虚拟机上，公有云或混合云上。

Docker的可移植性和轻量级使得很容易动态管理它的工作负载，这就使得近实时的根据工作负载扩容和缩容应用和服务成为可能。

#### 同样的硬件上运行更多的工作负载

Docker轻量级并且运行速度很快。相比基于hypervisor的虚拟机它的消耗更少，因此你可以使用更多的运算能力来达成业务目标。Docker尤其适合于高密度的中小应用，可以用较少的资源运行更多的应用。

### Docker架构

Docker是C/S架构。Docker客户端通过与Docker守护进程对话，由守护进程完成构建、运行并发布容器的工作。Docker客户端和守护进程可以运行在同一个系统上，客户端也可以连接远程的Docker守护进程。客户端和守护进程之间通过REST API或者socket进行通信。

![Docker Architecture Diagram](https://docs.docker.com/engine/images/architecture.svg)

#### Docker守护进程

Docker守护进程有两个作用：

- 监听Docker API的请求
- 管理Docker对象，比如镜像、容器、网络和数据卷

Docker守护进程也可以与其他的Docker守护进程交互来管理Docker服务。

#### Docker客户端

Docker客户端是Docker用户与Docker交互的主要方式之一。当使用命令`docker run`，客户端会把命令发送给守护进程`dockerd`。`docker`命令底层使用的是Docker API。docker客户端可以与多个守护进程通信。

#### Docker注册中心

Docker注册中心主要用来存储Docker镜像。Docker Hub是公开的注册中心，任何人都可以访问它，Docker在默认情况下会从Docker Hub下载镜像。你也可以运行私有的注册中心。当使用Docker Datacenter(DDC)时，它一般会包括Docker Trusted Registry(DTR)。

当使用`docker pull` 或`docker run`命令时，所需要的镜像会从配置好的注册中心拉取下来。当使用`docker push`命令时，镜像会推送到配置好的注册中心。

#### Docker对象

当你使用Docker时，其实就是在创建和使用镜像、容器、网络、数据卷、插件以及其他的对象。这一节将对所有这些对象进行概述。

##### 镜像

镜像是由一系列指令组成的只读模板，可以用来创建容器。镜像是分层的，大部分情况下，一个镜像是在另一个镜像基础之上做了定制化。比如可以基于`ubuntu`	镜像构建一个新镜像，包含Apache服务器和应用程序，以及让应用程序运行所必需的的配置信息。

你可以创建自己的镜像，或使用已发布到注册中心的镜像。新建一个Dockerfile文件，按照Dockerfile语法规则定义一些步骤形成的镜像然后运行它，就可以得到自己的镜像。Dockerfile中的每一条指令都会在镜像中创建一个层。当修改Dockerfile文件后，只有改变的那一层会重新构建。这也是Docker相比虚拟机更轻量级、更小、更快的原因之一。

##### 容器

容器是可运行的镜像实例。通过Docker API或客户端能够创建、启动、停止、迁移或移除容器。我们还能够将容器连接到多个网络接口、挂载存储设备，甚至基于它当前的状态生成新的镜像。

默认情况下，容器与其他的容器或宿主机之间能进行很好的隔离。我们能够控制容器如何与其他的容器或宿主机隔离网络、存储以及其他子系统。

创建和启动容器需要镜像和相关的配置项。当容器移除之后，任何还未持久化的容器状态都会丢失。

###### `docker run`命令示例

如下的命令运行一个`ubuntu`容器，在本地命令行session中以交互模式运行`/bin/bash`

```powershell
docker run -i -t ubuntu /bin/bash
```

当运行这个命令时，将会发生下面这些事情(假设配置默认的注册中心)

1. 当你本地没有`ubuntu`镜像时，docker会从配置的注册中心中下载镜像，如同手动执行`docker pull ubuntu`。
2. Docker创建新的容器，如同手动执行`docker container create`
3. Docker为容器申请一个可读写的文件系统，作为最后一层。这样容器就能在它的本地文件系统中新建和修改文件或目录。
4. Docker为容器创建网络接口连接到默认的网络上，这样你就不需要指定任何网络选项。会为容器分配一个IP地址。默认情况下，容器可以借助于宿主机网络连接与外网通信。
5. Docker启动容器并执行`/bin/bash`。由于容器以交互模式启动并启动了终端，因此可以通过键盘输入任何信息同时终端上会显示输出内容。
6. 当在`/bin/bash`中输入`exit`。容器会停止但没有移除。你可以再次启动它，或者移除它。

##### 服务

服务允许你协调多个Docker守护进程间的容器，它们将在多个管理器和工作节点中以集群的方式工作。集群中的每个成员是一个Docker守护进程，守护进程间通过Docker API进行通信。服务让你能按预期定义集群的状态，比如集群的副本数。默认情况下，服务将在所有的工作节点间进行负载均衡。对服务使用方来说，整个集群对外看起来像是一个单一应用。Docker1.12开始支持集群模式。

### 底层技术

Docker是用go编写的，它利用了许多Linux内核的高级特性来交付能力。

#### Namespaces

Docker使用命名空间技术来实现隔离的工作空间----容器。当创建容器的时候，Docker底层创建了一系列命名空间。

这些命名空间保证了层之间的隔离性。容器的每个功能都运行在一个独立的命名空间并且它的访问也限制在该命名空间内。

在Linux中Docker引擎使用如下这些命名空间：

-   `pid`命名空间：进程隔离(PID)
-   `net`命名空间：管理网络接口(NET)
-   `ipc`命名空间：IPC资源的访问管理(IPC)
-   `mnt`命名空间：管理文件系统的挂载点(MNT)
-   `uts`命名空间：隔离内核和版本标识符

#### Cgroups

Linux上的Docker引擎还依赖另外一种技术---cgroups。cgroups能限制应用对某些资源的使用。cgroups允许Docker引擎共享硬件资源同时给它加上限制和约束。比如，对容器可以进行内存使用限制。

#### Union file systems

UnionFS，能够以分层的方式进行操作，使得它既轻量级又快。Docker引擎利用UnionFS来提供容器的构建块。Docker引擎能够使用UnionFS的多种变体，比如AUFS、btrfs、vfs和DeviceMapper。

#### Container format

Docker引擎将namespace、cgroups和UnionFS组合在一起形成一个整体就是容器格式。默认的容器格式是`libcontainer`。未来，Docker引擎通过集成BSD Jails或Solaris Zones等技术来支持其他更多的容器格式。



### 下一步

-   阅读 [installing Docker](https://docs.docker.com/engine/installation/#installation)
-   实践指南 [Getting started with Docker](zh-cn/docker/quickstart/part01.md) 
-   结合示例深入研究 [Docker Engine user guide](https://docs.docker.com/engine/userguide/).





> 参考：[https://docs.docker.com/engine/docker-overview/](https://docs.docker.com/engine/docker-overview/)

