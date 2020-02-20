## Part 03:在Docker Hub中共享镜像

| Part01                                                | Part02                                              | Part03                                                      |
| ----------------------------------------------------- | --------------------------------------------------- | ----------------------------------------------------------- |
| [Docker概念和设置](zh-cn/docker/quickstart/part01.md) | [构建并运行镜像](zh-cn/docker/quickstart/part02.md) | [在Docker Hub中共享镜像](zh-cn/docker/quickstart/part03.md) |

### 先决条件

浏览完构建并运行镜像[part 02](/zh-cn/docker/quickstart/part02.md)一节

### 简介

到目前为止，在Docker桌面的帮助下，在[part 02](zh-cn/docker/quickstart/part02.md) 中已经构建完一个容器化的应用。开发容器化应用的最后一步是将镜像分享到类似于[Docker Hub]()的注册中心中，这样就能轻松下载并在任一机器上运行它。

### 设置Docker Hub账号

如果你没有Docker ID，参考下面的步骤创建一个，这样就能把你的镜像分享到Docker Hub了。

- 访问Docker Hub 注册页面 [https://hub.docker.com/signup](https://hub.docker.com/signup)

- 填写并提交表单以获得Docker ID

- 验证邮箱地址以完成注册流程

- 点击系统托盘或工具栏中的Docker图标，并单击**登录/创建Docker ID**

- 填上Docker ID和密码。Docker ID认证成功之后，你的Docker ID就会出现在Docker桌面的菜单处并替换掉你使用的`登录`选项

  > 在命令行键入 `docker login`同样能完成登录

### 创建Docker Hub仓库并推送镜像

现在我们可以制作第一个仓库了，并将公告板镜像分享到仓库中。

1. 点击菜单栏的Docker 图标，导航到 **Repositories > Create** 。你将被带到Docker Hub页用以创建一个仓库

2. 给仓库命名`bulletinboard` 。其他选项保持不变，点击底部的 **Create** 按钮。

   ![make a repo](https://docs.docker.com/get-started/images/newrepo.png)

3. 现在你还必须要做一件事：镜像必须有命名空间这样才能正确的在Docker Hub中共享。一般来说，你可以像这样来命名`<Docker ID>/<Repository Name>:<tag>` 。你可以像这样重新标记 `bulletinboard:1.0` 镜像：

   ```powershell
   docker image tag bulletinboard:1.0 gordon/bulletinboard:1.0
   ```

4. 最后一步，将镜像推送到Docker Hub

   ```powershell
   docker image push gordon/bulletinboard:1.0
   ```

   访问Docker Hub中的仓库，你将看到新的镜像。默认Docker Hub中的仓库是公开的。

   > **推送遇到问题？** 首先必须先登录到Docker Hub，其次要把镜像正确的命名，参考前面的步骤。如果push命令看起来没有问题，但Docker Hub中看不到，过几分钟后刷新下浏览器再试试。

### 总结

现在你的镜像位于Docker Hub了，接下来你可以在任何地方运行它。如果你在新的机器上使用它，Docker会自动从Docker Hub中下载该镜像。通过镜像，在你要运行软件的机器上除了安装Docker外不再安装任何其他依赖。镜像中封装和隔离了容器化应用的依赖，它可以通过Docker Hub进行共享。

另外还有一件事你必须知道：到目前为止，你仅仅只是把镜像推送到了Docker Hub；那你的Dockerfile呢？一个最佳实践是让它跟源代码一起做好版本控制。在Docker Hub仓库中添加一个链接或说明来指明你可以在哪里找到Dockerfile文件，记录好如何构建镜像，以及如何作为一个完整的应用来运行的。



> 参考：[https://docs.docker.com/get-started/part03/](https://docs.docker.com/get-started/part3/)

