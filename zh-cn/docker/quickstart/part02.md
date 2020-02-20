# 构建并运行镜像

### 先决条件

确保浏览完Docker的概念和设置[part 01](/zh-cn/docker/quickstart/part01.md)

### 简介

现在你已经设置好开发环境，基于Docker桌面，可以开发容器化的应用。一般来说，开发的流程如下所示：

1. 为应用的每个组件创建独立的容器之前需要先创建Docker镜像
2. 组装容器并提供支持完整应用的基础设施
3. 测试、共享并部署容器化应用

在指南的这一阶段，我们聚焦在开发流程的第1步：创建镜像，镜像是容器的模板。Docker镜像拥有容器化应用运行进程的私有化文件系统；创建的镜像需要包含应用程序运行需要的所有资源。

> 当你学会了如何构建镜像，你会发现 **容器化的开发环境 **相比传统的开发环境更容易设置。这是因为容器化的开发环境将应用运行所需要的依赖完全隔离在镜像中；在开发机器上除了安装Docker外不需要再安装任何其他软件。用这种方式，在你的机器上可以轻松开发各种不同技术栈的应用，而不需要任何配置。



### 设置

我们从 [Docker Samples]() 中下载一个示例工程

| Git                                                          | Windows(没有安装Git)                                         | Linux(没有安装Git)                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Git**                                                                                                     如果你在使用Git，可以直接从GitHub中clone该项目                                                  ```git clone https://github.com/dockersamples/node-bulletin-board                       cd node-bulletin-board/bulletin-board-app``` | **Windows (without Git)**                               如果你使用的是Windows的系统，更喜欢不安装Git来下载示例工程，请运行下面的命令 ```curl.exe -LO https://github.com/dockersamples/node-bulletin-board/archive/master.zip tar.exe xf master.zip                cd node-bulletin-board-master\bulletin-board-app``` | **Mac or Linux (without Git)**                       如果你使用的是Mac或者Linux机器，更倾向于在不安装Git的情况下下载示例项目，请在终端运行如下的命令：                             ```curl -LO https://github.com/dockersamples/node-bulletin-board/archive/master.zip unzip master.zip                     cd node-bulletin-board-master/bulletin-board-app``` |

`node-bulletin-board` 项目是用 Node.js写的一个公告板项目。在这个示例中，我们假设是你开发了这个应用，现在你打算容器化它。

### 通过Dockerfile来定义容器

看一下公告板项目中名为 `Dockerfile ` 的文件。Dockerfiles描述了如何为容器组装一个私有文件系统，同时也会包含描述容器如何基于镜像运行的元数据。公告板项目的Dockerfile看起来如下所示：

```dockerfile
# Use the official image as a parent image
FROM node:current-slim

# Set the working directory
WORKDIR /usr/src/app

# Copy the file from your host to your current location
COPY package.json .

# Run the command inside your image filesystem
RUN npm install

# Inform Docker that the container is listening on the specified port at runtime.
EXPOSE 8080

# Run the specified command within the container.
CMD [ "npm", "start" ]

# Copy the rest of your app's source code from your host to your image filesystem.
COPY . .
```

编写`Dockerfile`是容器化应用的第1步。你可以把这些Dockerfile命令看成是一步步的食谱用来构建镜像。这个镜像按照如下步骤构建：

- 用 `FROM` 和已经存在的 `node:current-slim`镜像开头。这是个官方镜像，由node.js供应商构建，并由Docker验证它是一个质量很高的镜像，该镜像包含可以长期获得支持的Node.js解释器和基本依赖。
- 使用 `WORKDIR` 用来指明接下来所有的指令都应该基于镜像文件系统中的目录 `/usr/src/app`（不是宿主机的文件系统）
- 从宿主机`COPY`文件`package.json`到镜像中的当前位置（本例中是只`/usr/src/app/package.json`）
- 在镜像文件系统中`RUN`命令`npm install`(该命令将读取`package.json`来决定应用的node依赖并安装它们)
- 将宿主机中剩余的应用源代码`COPY` 到镜像文件系统中

你可以看出这些步骤与在宿主机中设置并安装应用的步骤基本相同。然而，通过Dockerfile来捕获这些步骤能让你在可移植的、隔离的镜像中完成同样的事情。

上面的步骤构建出了镜像的文件系统，但Dockerfile中还有其他的行。

第1个示例中的`CMD`指令说明了镜像中的一些元数据，描述了基于该镜像的容器如何运行。本例中，它是指该镜像运行的容器是`npm start`。

`EXPOSE 8080`说明Docker 进程运行时正在监听的端口。

上面是组织简单Dockerfile的一种方式；总是以`FROM` 命令开头，紧接着是构建文件系统的其余步骤，并以其他元数据描述结束。上面看到的Dockerfile指令只是很小的一部分。为获取完整的Dockerfile指令，请参考[Dockerfile reference](https://docs.docker.com/engine/reference/builder/)。

### 构建并测试镜像

现在你拥有完整的源码和一个Dockerfile，是时候构建你的第一个镜像，并通过该镜像启动一个能正常工作的容器。

在终端或Powershell中使用`cd`命令确保你位于目录`node-bulletin-board/bulletin-board-app`。 让我们开始构建公告板镜像：

```powershell
docker image build -t bulletinboard:1.0 .
```

你将看到Docker将会一步步执行你在Dockerfile中编写的指令。如果构建成功，构建将会以`Successfully tagged bulletinboard:1.0` 这个提示信息结束构建过程。

### 基于镜像运行一个容器

1. 基于新镜像启动一个容器：

   ```powershell
   docker container run --publish 8000:8080 --detach --name bb bulletinboard:1.0
   ```

   命令中有一些常用的标记：

   - `--publish` 告诉Docker将从主机8000端口来的流量转发到容器的8080端口。容器拥有一批私有的端口，因此如果你想从网络中获取指定的一个，你必须用这种方式来实现流量转发。否则，作为默认安全策略的防火墙规则会阻止网络流量到达容器。
   - `--detach` 告诉Docker在后台运行容器
   - `--name` 指明了将要启动容器的别名，本例中是指 `bb`

   这里必须注意，你并没有指明容器中要运行哪个应用。事实上你不需要指定，因为在Dockerfile中你使用`CMD`指令指明了当Docker容器启动成功后自动运行程序`npm start`

2. 在浏览器中访问`localhost:8000` 。你将看到公告板应用启动成功并运行着。在这个步骤中，你会尽一切努力确保容器按预期运行；现在可以进行单元测试了。

3. 如果你对公告板容器的表现满意，现在可以删除它了

   ```powershell
   docker container rm --force bb
   ```

   `--force` 选项会移除正在运行的容器

### 总结

到这里，你已经成功构建了一个镜像，完成了简单的容器化应用，并确认了该应用在容器中正常工作。接下来会把镜像分享到[Docker Hub](https://hub.docker.com/) ，这样就能很方便的下载镜像并让它在任何目标主机运行。

### CLI 参考

本篇文章中出现的CLI命令的更多详细信息请参考：

- [docker image](https://docs.docker.com/engine/reference/commandline/image/)
- [docker container](https://docs.docker.com/engine/reference/commandline/container/)
- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)





