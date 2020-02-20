## Part 1:Docker概念和设置

### Docker 相关概念

​        Docker是为开发人员和系统运维人员提供的以容器化形式**构建、运行、分享**应用程序的平台。通过容器来发布应用称之为*容器化*。容器并不是新概念，但使用容器来部署应用确是一个新概念。

​		容器化越来越流行，主要有以下几方面原因：

- **灵活性好**：就算是最复杂的应用也可以容器化
- **更轻量级** ：容器通过使用并共享宿主机内核，使得它在系统资源的占用上比虚拟机更高效
- **可移植性好**: 本地构建，云上部署，到处运行
- **松散耦合**：每个容器都是封装好并高度自治的，替换或升级任何其中任一个都不会影响到其他的容器
- **可扩展性好**：可以任意增加或通过数据中心自动发布容器副本
- **安全性高**：容器会对进程主动执行严格的限制和隔离，不需要用户进行任何配置

#### 镜像和容器

​		从根本上来说，容器就是通过封装各种特性来隔离宿主机和其他容器的一个进程。容器隔离性的一个最重要的方面是每个容器会与它自己私有的文件系统进行交互；这个文件系统是Docker*镜像*提供的。镜像包含运行应用所需要的一切资源，包括代码，运行时环境，依赖项以及其他必须的文件系统。

#### 容器和虚拟机

​		容器天然运行在Linux环境中，并与其他容器一起共享宿主机的内核。它运行独立的进程，相比其他可执行程序占用更少的内存使它更轻量级。

​		相反，虚拟机运行成熟的操作系统，借助 *hypervisor* 虚拟访问宿主机资源。通常，虚拟机锁运行的消耗会超过应用程序运行所需要的资源消耗。

|                            Docker                            |                              VM                              |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![Container stack example](https://docs.docker.com/images/Container%402x.png) | ![Virtual machine stack example](https://docs.docker.com/images/VM%402x.png) |

### 设置Docker环境

#### 下载并安装Docker桌面

Docker桌面是在Mac和Windows环境中运行的一个应用程序，可以让你在几分钟就能开始编码和容器化。

Docker桌面包含了从你的机器构建、运行和分享容器化的应用需要的所有内容。



结合操作系统选择合适的步骤来下载并安装Docker桌面：

- [Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/install/)
- [Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/)

#### 测试Docker版本

成功安装好Docker桌面后，开启一个终端并执行 *docker --version* 来检测机器上安装的docker版本

```powershell
$ docker --version
Docker version 19.03.5, build 633a0ea
```

#### 测试Docker安装

  1. 通过运行 [hello-world](https://hub.docker.com/_/hello-world/) 镜像来测试Docker的安装

     ```powershell
     $ docker run hello-world
     Unable to find image 'hello-world:latest' locally
         latest: Pulling from library/hello-world
         ca4f61b1923c: Pull complete
         Digest: sha256:ca0eeb6fb05351dfc8759c20733c91def84cb8007aa89a5bf606bc8b315b9fc7
         Status: Downloaded newer image for hello-world:latest
     
         Hello from Docker!
         This message shows that your installation appears to be working correctly.
         ...
     ```

  2. 运行 *docker image ls* 来列出你下载到本机的 *hello-world* 镜像

  3. 列出 hello-world 容器(该容器显示完消息后会立马退出)。如果它还在运行，就不需要添加 --all 选项

     ```powershell
     $ docker container ls --all
     CONTAINER ID     IMAGE           COMMAND      CREATED            STATUS
         54f4984ed6a8     hello-world     "/hello"     20 seconds ago     Exited (0) 19 seconds ago
     ```

#### 总结

到目前为止，你已经在开发机器上安装完Docker桌面，运行了一个用例来验证已设置好构建、运行容器化的应用



#### CLI 参考

参考文档中接下来的这些主题以了解本篇文章中使用到的客户端命令：

- [docker version](https://docs.docker.com/engine/reference/commandline/version/)
- [docker run](https://docs.docker.com/engine/reference/commandline/run/)
- [docker image](https://docs.docker.com/engine/reference/commandline/image/)
- [docker container](https://docs.docker.com/engine/reference/commandline/container/)



### 参考

[https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)
