# 2 了解Docker并运行Hello World

现在可以动手使用Docker了。在本章中，您将积累使用Docker核心特性的许多经验：使用容器来运行应用。首先会向您介绍一些背景知识，帮助您准确的理解Docker是什么，以及为什么Docker能以轻量级的方式来运行应用。通常，只要按照 TRY-IT-NOW 的练习来安排学习，执行简单的命令就能了解运行应用程序的新方法。

## 2.1 在容器中运行Hello World

让我们以与任何新计算机概念相同的方式开始使用Docker：运行Hello World。在第1章我们已经开始运行Docker，因此请打开您喜欢的终端-在Mac上可以是Terminal，在Linux上可以是Bash shell，我建议在Windows中使用PowerShell。

您将向Docker发送一条命令，告诉它运行一个容器，该容器会打印出一些简单的“ Hello，World”文本。

>   **TRY-IT-NOW**
>
>   输入此命令，它将运行Hello World容器：

```powershell
$ docker container run diamol/ch02-hello-diamol
```

当我们完成本章的学习后，你就能准确的理解这行命令执行背后发生了什么事情。现在只需要简单看下输出，它看起来像下面这样(见图2.1)：

**图2.1运行Hello World容器的输出。您可以看到Docker下载应用程序包（称为“图像”），在容器中运行该应用程序并显示输出。**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_6.png)

该输出中有太多内容。我将缩减代码以使它们看起来更简短，由于这是第一个清单，我想完整展示它，以便我们对其进行剖析。

命令执行背后到底发生了什么？`docker container run`命令只是告诉Docker在容器中运行应用程序。该应用程序已打包为可在Docker中运行，并已发布在任何人都可以访问的公共站点上。容器包（Docker称之为“镜像”）被命名为`diamol/ch02-hello-diamol`（`diamol` 是本书标题Docker in A Month of Lunches首字母缩略词）。您刚刚输入的命令就是告诉Docker基于该镜像运行容器。

Docker首先在本地拥有镜像副本，然后才能使用该镜像运行容器。第一次运行此命令时，您本地没有该镜像的副本，所以会在输出的第一行中看到：“Unable to find image”。然后Docker下载镜像（Docker称为“pull”），您会看到镜像已完成下载。

现在Docker可以使用镜像来启动容器。镜像包含应用的全部内容，指令会告诉Docker如何运行应用。镜像中的应用只是一个简单的脚本，你已经看到上面输出了以`Hello from Chapter 2！`开头的内容。该应用主要输出当前正在运行电脑的基本信息：

机器的主机名，本例是`2cff9e95ce83`

操作系统，本例是`Linux 4.9.125-linuxkit x86_64`

网络地址，本例是`172.17.0.3`

我在前文说过您的输出可能跟上面类似，但并不完全相同，这是因为输出的信息依赖于你自己的电脑。我的电脑是64位inter处理器并且运行着Linux操作系统。如果你在Windows电脑上运行，那么`I’m running on` 这行会如下面这样显示：

```powershell
---------------------
I'm running on:
Microsoft Windows [Version 10.0.17763.557]
---------------------
```

如果运行在Raspberry Pi，那么会输出使用的处理器：

```powershell
---------------------
I'm running on:
Linux 4.19.42-v7+ armv7l
---------------------
```

这个应用非常简单， 但却展示了Docker核心的工作流程。有些人把他们的应用打包成容器(这章由我提供这个应用，下一章你将亲自完成这样一个应用)，把它发布出去供其他人使用，这样任何能访问仓库的人都能在容器中运行这个应用。Docker把这称为构建、共享与运行。

这个概念非常强大，因为不管面对多复杂的应用，交付流程始终是一致的。应用可能只是一个简单的脚本，也可能是一个由许多组件，配置文件和依赖库组成的Java应用。但整个交付流程都是一致的。Docker镜像能运行在它支持的任何电脑上，这也使得应用变得完全可移植-可移植性是Docker的另外一个特性。

当再次运行同样的命令来启动容器会发生什么呢？

>   **TRY-IT-NOW**
>
>   重复执行同一个Docker命令：

```powershell
docker container run diamol/ch02-hello-diamol
```

你将看到与第一次运行完全一样的输出内容，但还是能发现一些不同的地方。Docker本地已经有该镜像的副本，因此它不会像第一次运行命令时一样会首先从远端下载镜像，它会直接运行容器。当然你是在同一台机器上运行该容器，所以容器会输出同样的操作系统信息。但容器的名称和ip地址将会不同：

```
---------------------
Hello from chapter 2!
---------------------
My name is:
858a26ee2741
---------------------
Im running on:
Linux 4.9.125-linuxkit x86_64
---------------------
My address is:
inet addr:172.17.0.5 Bcast:172.17.255.255 Mask:255.255.0.0
```

现在我的应用运行在主机名为`858a26ee2741`、IP为`172.17.0.5`的容器上。容器的名称每次都会改变，IP
地址经常会发生改变，但是每个容器都运行在同一台物理机上，那这些不同的容器名和IP地址是怎么回事呢？接下来我们将深入探讨这一原因，然后再继续做练习。

## 2.2 容器是什么？

Docker容器的概念与现实世界中的容器思想很相似-它相当于一个装应用的盒子。在盒子中，应用就像是拥有一台完整的计算机，有自己的主机名，自己的IP地址以及自己的磁盘。图2.2显示了应用是如何封装到容器中的：

**图2.2 容器中的应用**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_7.jpg)

Docker创建的主机名，IP地址和文件系统都是虚拟资源。Docker管理这些逻辑对象，它们一起协作就创造了能让应用运行的环境。这就是容器的封装性。

盒子中的应用无法看到盒子外的任何事物，然而盒子运行在物理主机上，同一台物理机上能运行许多盒子。那些盒子中的应用都有它们自己的运行环境(由Docker管理)，它们都看不到盒子外面的事物。

关于容器还有一个非常重要的特性-即它们拥有独立的环境，但它们还是会共享宿主机的CPU和内存，也会共享宿主机的操作系统-通过图2.3你将看到同一主机上的容器是如何隔离的：

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_8.jpg)

为什么隔离这么重要呢？其实这涉及到计算机领域一对矛盾的问题：隔离和密度。密度就是说为了重复利用处理器和内存资源，会在一台宿主机上运行尽可能多的应用。但现实中不同的应用之间可能没法共存-它们可能使用不同版本的Java或.NET框架，它们依赖不兼容的工具或库，它们有的消耗大量内存，有的消耗大量CPU。但实际上应用之间确实需要隔离开，这就导致一台宿主机上不能运行太多的应用，因此就无法获得足够的密度。

最开始想到的解决方案是使用虚拟机。虚拟机的思想与容器类似，在虚拟机中它也会提供一个类似盒子的容器让你运行容器-但虚拟机下的容器需要运行操作系统，它们无法共享宿主机的操作系统。图2.4展示了容器和虚拟机的不同：

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_9.jpg)

从上图来看，虚拟机相比容器只有很小的区别，但从设计思想上来看其实有区别很大。每个虚拟机都有自己的操作系统，每个操作系统会占用大量的内存和CPU资源-目的是为支持应用运行所必须的运算能力。另外，对操作系统的许可费用以及安装、升级OS的维护费用也将是不得不考虑的一方面。虚拟机牺牲应用部署密度来提供隔离性。

容器同时提供了隔离性和密度。容器共享宿主机的操作系统，这就使容器特别轻量级。容器启动速度快、运行消耗少，在同样配置的硬件资源上能运行比虚拟机更多的容器，多达5-10倍。部署足够多的应用，并且每个应用都在它自己的容器中，因此也获得了应用之间的隔离性。这是Docker的另一个重要特性：高效。

到这里您应该清楚Docker背后的魔术了，在接下来的练习中我们将完成更多的容器操作。

## 2.3 像连接远程计算机一样连接容器

我们运行的第1个容器只做了一件事-应用打印了一些文本然后就结束了。在很多情况下，你只需要做一件事。比如通过脚本来自动化一个流程。由于这些脚本依赖工具运行，因此你无法与同事共享这些脚本，你还需要提供这些工具的安装文档，与此同时你的同事还需要花费几个小时来安装这些工具。相反，如果你把这些工具和脚本打包到docker镜像中，与同事共享该镜像，那么你的同事不需要任何额外的安装工作就能运行脚本。

你也可以用其他方式来与容器协同工作。接下来你将启动容器并通过终端连接到容器内部，就如同连接到远程机器上一样。使用`docker container run`命令，传入可选参数，在终端会话中以交互方式运行容器。

>   **TRY-IT-NOW**
>
>   在终端中运行下面的命令：

```powershell
docker container run --interactive --tty diamol/base
```

`--interactive`参数告诉Docker你要与容器建立连接，`--tty`参数意思是在容器中建立一个终端会话。输出里面展示Docker拉取镜像的过程，然后会弹出一个命令提示符。该命令提示符是位于容器内的终端，如图2.5所示：

**图2.5 以交互方式运行容器并连接到容器终端**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_10.jpg)

在Windows中运行该命令会完成同样的工作。除了弹出的是Windows命令行提示符：

```powershell
Microsoft Windows [Version 10.0.17763.557]
(c) 2018 Microsoft Corporation. All rights reserved.
C:\>
```

无论以哪种方式运行，现在你已经进入容器内部，可以像操作系统命令行一样运行任何命令。

>   **TRY-IT-NOW**
>
>   运行`hostname`和`date`命令，你将看到容器环境的详细信息：

```powershell
/ # hostname
f1695delf2ec
/ # date
Thu Jun 20 12:18:26 UTC 2019
```

如果你想探索更多信息那么你需要熟悉下这个命令行，其实这就是个能连接到远程主机的本地终端，只是这台机器是运行在宿主机上的容器。通过SSH连接远程Linux主机，通过RDP连接远程Windows Server Core主机，这与在Docker中的效果是一样的。

由于容器会共享主机的操作系统，所以在Linux中你会看到Linux shell，在Windows中你会看到Windows command。有些命令在不同的操作系统上是一样的(比如`ping google.com`)，但也有一些命令语法不同(在Linux中使用`ls`列出目录内容，在Windows中使用`dir`)。

不管你使用的是什么操作系统或处理器，Docker表现出来的行为都是一样的，容器中的应用看起来就像是运行在基于Intel的Windows机器或基于Arm的Linux机器之上。无论容器中运行的是什么，Docker容器的表现行为都是一样的。

>    **TRY-IT-NOW**
>
>   开启一个新的终端，通过以下命令能得到所有正在运行的容器详情：

```powershell
docker container ls
```

输出会展示每个容器的详细信息，包括正在使用的镜像，容器ID，容器中运行的Docker命令：

```powershell
CONTAINER ID		IMAGE				COMMAND				CREATED					STATUS	
f1695de1f2ec  diamol/base  "/bin/sh"  16 minutes ago  Up 16 minutes
```

如果你仔细看一眼，会注意到容器ID与容器中的主机名是一样的。Docker为每个创建的容器分配一个随机ID，这个ID的一部分会用作主机名。Docker提供了许多与容器操作相关的命令让你能与某一个特定的容器交互，通过容器ID的前几位字母也能标识出容器ID。

>   **TRY-IT-NOW**
>
>   `docker container top` 列出了容器中运行的进程-我使用`f1`作为容器ID`f1695de1f2ec`的缩减格式。

```powershell
> docker container top f1
PID                 USER                TIME                COMMAND
69622               root                0:00                /bin/sh
```

如果容器中正在运行多个进程，Docker会把它们都显示出来。对于Windows容器也是一样，除容器应用之外，总还是会运行几个后台进程。

>   **TRY-IT-NOW**
>
>   `docker container logs` 显示了容器收集到的所有日志条目：

```powershell
> docker container logs f1
/ # hostname
f1695de1f2ec
```

Docker通过使用容器中应用的输出收集日志条目。在本例中终端展示了我运行的命令以及它们运行的结果，但对实际应用来说你将看到代码的日志内容。比如对Web应用来说对于每一条HTTP请求都会记录一条日志，这些都会在容器日志中显示出来。

>   **TRY-IT-NOW**
>
>   `docker container inspect` 能展示容器的各种基本信息：

```powershell
> docker container inspect f1
> docker container inspect f1
[
    {
        "Id": "f1695de1f2ecd493d17849a709ffb78f5647a0bcd9d10f0d97ada0fcb7b05e98",
        "Created": "2019-06-20T12:13:52.8360567Z"
```

完整的输出会包含很多底层的信息，包括容器的虚拟文件系统的路径，以及容器连接到的虚拟Docker网络。它以JSON的形式组织，对自动化脚本来说比较友好，但如果作为书本中的代码清单就不太合适了--因此我只列出开始的几行。

上面这些就是你与容器一起工作时需要使用的命令，当你要定位应用问题，检查进程占用过多的CPU资源，查看容器建立的网络情况，都需要使用这些命令。

关于这些练习，你应该能体会到容器的理念，所有的容器看起来都比较相似。Docker在每个应用上加了一致的管理层。在Linux容器中运行着一个跑了10年的老Java应用，在Windows容器中运行着一个跑了15年的老.NET应用，以及在Raspberry Pi上运行着新的 Go 应用。你将使用同样的命令来管理它们：`run`来启动应用，`logs`查看日志，`top`查看进程，`inspect`查看应用详情。

到现在为止我们已经发现了更多Docker能做的事情，接下来我们将通过更多的练习来完成一个更有意义的应用。现在可以关闭第2个终端窗口(就是执行docker container logs的终端)，回到第1个终端窗口-仍然连接到容器中--执行`exit`将结束终端会话。

## 2.4 在容器中托管网站

现在我们已经运行了好几个容器。第1个容器运行一个打印文本然后退出的应用。接下来使用交互模式让我们进入容器终端，该终端会一直运行直到我们退出会话。`docker container ls` 显示当前宿主机中没有容器了，这是因为该命令只显示当前正在运行的容器。

>   **TRY-IT-NOW**
>
>   `docker container ls --all` 该命令将列出所有状态的容器：

```powershell
> docker container ls --all
CONTAINER ID  IMAGE                     COMMAND                CREATED            STATUS   
f1695de1f2ec  diamol/base               "/bin/sh"              About an hour ago  Exited (0)
858a26ee2741  diamol/ch02-hello-diamol  "/bin/sh -c ./cmd.sh"  3 hours ago        Exited (0)
2cff9e95ce83  diamol/ch02-hello-diamol  "/bin/sh -c ./cmd.sh"  4 hours ago        Exited (0)
```

容器都有一个状态：`Exited`。这里有一些重要的概念必须要搞清楚。

首先：只有容器中的应用处于运行状态才代表容器是运行的。只要应用执行完成，容器就会进入退出状态。退出状态的容器不会使用CPU和内存资源。”Hello World“容器在脚本执行完后自动进入退出状态。我们连接的那个交互容器只要我们退出终端容器会立即进入退出状态。

其次：容器退出后并不会消失。处于退出状态的容器依然存在，这意味着你可以再次启动它，检查日志，在容器与宿主机之间完成文件的拷贝。查看运行中的容器请使用`docker container ls`,但Docker不会移除已退出的容器，除非你告诉它要这么做。已退出的容器仍然会占用磁盘空间，因为它们的文件系统仍然存在宿主机的电脑磁盘上。

那么如何启动容器让它在后台保持并一直处于运行状态呢？这是Docker最主要的用法：运行像网站、批处理和数据库这样的服务应用。

>   **TRY-IT-NOW**
>
>   下面是一个简单的示例，在容器中运行一个网站：

```powershell
docker container run --detach --publish 8088:80 diamol/ch02-hello-diamol-web
```

这一次输出中只能看到一串长长的容器ID，同时会返回到命令行。容器仍然在后台运行。

>   **Try it now**
>
>   运行`docker container ls`你将看到这个新容器的状态是"Up"：

```powershell
> docker container ls
CONTAINER ID        IMAGE                                                                        COMMAND                   CREATED             STATUS              PORTS                           NAMES
e53085ff0cc4        diamol/ch02-hello-diamol-web                                                 "bin\\httpd.exe -DFOR…"   52 seconds ago      Up 50 seconds       443/tcp, 0.0.0.0:8088->80/tcp   reverent_dubinsky
```

你使用的镜像是`diamol/ch02-hello-diamol-web`。该镜像包括一个Apache web服务器和一个简单的网页。当启动完这个容器，就启动了一个web服务器，同时托管了一个自定义的网站。容器在后台运行和监听网络事件需要docker container run 命令配合额外的参数才能实现：

`--detach` - 在后台启动容器并显示容器ID

`--publish` - 建立容器到宿主机间的端口映射

运行detach容器是指把容器放入后台启动，不向前台展示，就像Linux守护进程或Windows服务一样。端口映射需要解释下。当安装完Docker，它会把自己注入到宿主机的网络层。任何进入宿主机的网络包都会被Docker拦截，然后Docker会把拦截到的网络包发送到容器。

默认情况下容器不对外暴露。它们有自己的IP地址，这个IP地址是Docker创建的，主要是为了让Docker来管理网络-容器不会连接到宿主机的物理网络上。发布容器端口意味着容器监听网络宿主机指定端口上的网络流量，并把它发送给容器。在本例中，通过端口8088发送到宿主机的网络流量都会被送到容器中的80端口-图2.6能看到这个网络流：

**图2.6 宿主机和容器之间的物理和虚拟网络**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_11.jpg)

本例中我的电脑就是运行Docker的机器，它有一个IP地址`192.168.2.150`。这个IP地址是我的物理网络，它是宿主机连接到路由器的时候分配的。这台电脑上只运行着一个容器，容器的IP地址是`172.0.5.1`。这个地址是Docker为了管理虚拟网络为容器分配的。网络中的其他电脑都不能连接这个容器IP，因为它只存在当前这个Docker容器中-由于容器已经发布了端口，所以网络中的电脑都能向该容器发送流量。

>   **TRY-IT-NOW**
>
>   在浏览器中输入 `http://localhost:8088` 。这是本机上的一个HTTP请求，但响应来自于容器(请看图2.7)：

**图2.7 本机容器托管的web应用**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_12.jpg)

在本书中我不会深入介绍网站设计相关的内容。

这是个非常简单的网站，但它仍受益于Docker带来的可移植性和高效性。Docker镜像中包含web站点和web服务器，这包含它运行需要的所有内容。web开发人员在实验室运行包含完整应用(从HTML到web服务器技术栈)容器的步骤，与运维人员将应用部署到100个容器的线上服务集群中完全一样。

容器中的应用将会一直运行，因此容器会一直运行。可以使用`docker container`命令来管理它。

>   **TRY-IT-NOW**
>
>   `docker container stats`是另一个有用的命令，它将显示当前容器占用的CPU、内存、网络和磁盘容量：

```powershell
> docker container stats e53
CONTAINER ID  NAME               CPU %  PRIV WORKING SET  NET I/O             BLOCK I/O      
e53085ff0cc4  reverent_dubinsky  0.36%  16.88MiB          250kB / 53.2kB      19.4MB / 6.21MB
```

当你通过容器完成了工作，可以通过`docker container rm`命令和容器ID来移除它，加上`--force`参数后即使容器还在运行也会把它强制移除。下面以我们经常使用的最后一个命令来结束这次练习。

>   **TRY-IT-NOW**
>
>   运行这个命令来移除所有的容器：

```powershell
docker container rm --force $(docker container ls --all --quiet)
```

`$()` 语法是把一个命令的输出发送给另一个命令-这种语法在Linux、Mac终端和Windows PowerShell上都能正常工作。组合这两个命令能获得本机上的所有容器ID，并强制移除它们。使用这个命令尤其要注意，因为它不会弹出确认，但这是一种很好的整理容器方法。

## 2.5 理解Docker如何运行容器

在本章我们已完成了许多 TRY-IT-NOW 练习，现在你应该已掌握容器的基本使用方法。

在本章的第1个 TRY-IT-NOW 练习中我提到了**构建、共享、运行**的工作流是Docker的核心。这个工作流让软件发布非常容易-我已经构建了所有的示例镜像并把它们予以共享，所以你能在容器中运行这些镜像，就像我本地所做的一样。大量的项目现在都使用Docker作为优先的发布方式。能列举许多软件-比如ElasticSearch、最新版的SQL Server以及Ghost博客引擎-都是使用你前面用过的`docker container run `命令方式来运行。

我们将介绍更多的背景知识，用以深刻理解Docker运行容器的时候实际发生了什么事情。安装Docker和运行容器表面上看起来很简单-但背后实际上是不同的组件在支持，如图2.8所示：

**图2.8 Docker组件**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_13.jpg)

**Docker引擎**是Docker的管理组件。它负责处理本地镜像缓存，在本地没有时下载镜像，本地有的时候重用镜像。它也会与操作系统协作来创建容器，虚拟网络和其他的Docker资源。引擎是一个在后台一直运行的进程(如同Linux的守护进程或Windows服务)。

Docker引擎通过**Docker API**来实现所有的功能，Docker API 就是标准的基于HTTP的REST API。你可以通过配置来控制本机访问这些API(这是默认配置)，也可以控制网络上的其他计算机访问这些API。

Docker命令行窗口(CLI)是Docker API的客户端。当执行docker命令时，CLI会向Docker引擎发送Docker API，然后Docker 引擎会完成对应的工作。

理解Docker的架构是有益的。与Docker引擎交互的唯一方式是通过API，有许多选项参数用来控制API的访问并保护它。CLI就是通过发送请求给Docker API来工作的。

到目前为止我们只是通过CLI来管理当前正在运行Docker宿主机上的容器，其实也能通过CLI调用远程正在运行Docker的主机并控制那台主机上的容器-这就是接下来我们要做的管理不同环境(像构建服务器、测试环境和生产环境)下的容器。不同操作系统上的Docker API是一样的，因此可以使用Windows电脑上的CLI来管理Raspberry PI或者云端Linux Server上的容器。

Docker API有个发布规范，Docker CLI也不是唯一的客户端。有许多种可视化的图形用户界面能帮你连接到Docker API，并提供给你可视化方式来与容器交互。API暴露了容器、镜像和Docker管理的其他资源的所有详细信息因此它可以提供丰富的仪表盘如图2.9所示：

**图2.9 Docker统一控制面板，一个面向容器的图形化用户界面**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_14.jpg)

这是Universal Control Plane(UCP)，Docker旗下公司的一款商业化产品。Portainer是另外一个产品，它是开源项目。UCP和Portainer都是以容器方式运行，因此它们都能很方便的部署并管理。

关于Docker架构我们不会再继续深入。Docker引擎使用了一个叫做 `containerd` 的组件来管理容器，`containerd`反过来使用操作系统的特性来创建容器这个虚拟环境。

你不需要理解容器底层的太多细节，但必须知道：containerd是被Cloud Native Computing Foundation监督的一个开源组件，运行容器的规范是公开的，称为Open Container Initiative(OCI)。

Docker是迄今为止最受欢迎和最容易使用的容器平台，但它不是唯一的容器技术。你大可不必担心会被某一个容器平台厂商绑定。

## 2.6 实验：探索容器文件系统

这是本书的第1个实验。这个实验任务需要你自己独立完成，这将有助于你巩固本章所学的内容。我会给出一些指导或一点提示，但最主要是禁止像做 TRY-IT-NOW 练习一样，你需要找到你自己解决问题的方式。

每个实验在这本书的GitHub仓库都会有个示例解决方案。这个实验非常值得你花点时间自己解决一遍，但如果你想查看我的解决方案，请到[https://github.com/sixeyed/diamol/tree/master/cha02/lab](https://github.com/sixeyed/diamol/tree/master/cha02/lab)。

现在进入正题：你的任务就是运行本章中的网站容器，但是要替换`index.html`文件，这样当你浏览的时候你将看到不同的主页(可以替换成任何你喜欢的内容)。记住容器有自己的文件系统，在这个应用中网站的文件位于容器的文件系统中。

下面是一些提示来希望对你有所帮助：

你可以运行`docker container`来得到所有操作容器的指令

追加`--help` 在任何`docker`命令后能看到更多关于该命令的详细帮助信息

在`diamol/ch02-hello-diamol-web`Docker镜像中，容器中运行的网站是通过目录`/usr/local/apache2/htdocs`(在Windows中是`c:\usr\local\apache2\htdocs`)来提供服务的

祝你好运 :)
