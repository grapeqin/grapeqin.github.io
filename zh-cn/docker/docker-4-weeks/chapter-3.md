



>   **[第2章](/zh-cn/docker/docker-4-weeks/chapter-2.md)**                                                  **[第4章](/zh-cn/docker/docker-4-weeks/chapter-4.md)**

# 3 构建自己的镜像

上一章已经运行了多个采用Docker管理的容器。不管使用哪种技术栈，容器提供了跨应用的一致性体验。现在你已使用过我构建并分享的镜像，这一章将介绍如何构建自己的镜像。在这里你将学习Dockerfile语法，以及在容器化应用过程中需要关注的要点。

## 3.1 使用来自Docker Hub的镜像

我们通过你在本章将要构建的镜像开始，这样你就能看到它是怎么与Docker协作。try-it-now 练习使用一个叫做`web-ping`的应用来检查网站是否在运行。这个应用通过容器运行，每3秒向我的博客发送一个HTTP请求直到容器停止运行。

在第2章我们已经知道，当运行`docker container run`命令时，如果当前宿主机没有镜像它会自动把它下载到本地，这是因为Docker平台内置了应用分发功能。离开了docker我们仍然能管理这些镜像，在必要时它能及时拉取镜像，当然也可以通过CLI直接拉取。

>   TRY IT NOW
>
>   拉取 web-ping 应用镜像：

```powershell
docker image pull diamol/ch03-web-ping
```

你将看到如图3.1所示相似的输出内容：

**图3.1 从Docker Hub拉取镜像**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_15.jpg)

镜像名为`diamol/ch03-web-ping`。它存储在Docker Hub，Docker默认从这里拉取镜像。镜像服务器称为注册表，Docker Hub是大家免费使用的公开注册表。Docker Hub也有个网站，你可以访问[https://hub.docker.com/r/diamol/ch03-web-ping](https://hub.docker.com/r/diamol/ch03-web-ping)  来获得该镜像的详细信息。

`docker image pull`命令的输出比较有趣，它将显示镜像是怎么存储的。Docker镜像逻辑上来看是一个整体-你可以把它看做是一个包含完整应用的zip打包文件-该应用包含NodeJS运行时环境以及应用代码。

拉取过程中你会看到并不是下载一个单独的文件，而是好几个文件在下载，称为镜像分层。 Docker镜像在物理结构上存储着许多小文件，Docker把它们组装起来形成完整的容器文件系统。当拉取完所有的镜像层后，完整的镜像就可以使用了。

>   **TRY IT NOW**
>
>   我们运行下这个镜像看看它做了些什么事情：

```powershell
docker container run -d --name web-ping diamol/ch03-web-ping
```

`-d`选项是`--detach`的简写形式，所以该容器将在后台运行。应用通过类似无用户界面的批处理任务来运行。与第2章中的网站容器不同，这个容器不需要接收外界的网络流量因此你不需要向外暴露端口。

这个命令中有个新选项`--name`。我们之前都是通过Docker生成的Docker ID来与Docker协同工作，与此同时你也可以给容器取一个对人友好的名字。该容器命名为`web-ping`，你可以使用名字而不是随机生成的ID来引用该容器。

容器中的应用会ping我的博客。应用以无限循环的方式运行，你可以通过在第2章中已经熟悉的`docker container`命令来观察应用的运行情况。

>   **TRY IT NOW**
>
>   看看Docker收集的应用日志：

```powershell
docker container logs web-ping
```

你会看到如图3.2所示的内容，应用正在向blog.sixeyed.com 发送HTTP请求：

**图3.2 web-ping容器正在运行，向我的博客发起固定频率的请求流量**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_16.jpg)

该应用发起web请求并记录应用的响应时间，有时可以采用这种方式来监控应用的存活状态。这个应用是硬编码发请求给我的博客，因此只对我有用。

除了监控我的网站外。该应用其实还可以配置成使用不同的URL，不同的请求间隔，甚至不同类型的HTTP调用。该应用会从系统环境变量中读取配置项。

环境变量是由操作系统提供的键值对。它们在Windows和Linux下能以同样的方式工作，并且非常适合存储小片数据。Docker容器也有环境变量，它们会覆盖来自操作系统的环境变量，它们是以Docker创建主机名和IP地址同样的方式创建的。

`web-ping`镜像设置了一些默认的环境变量。当运行容器的时候，Docker会读取这些环境变量，这样应用就能使用它们来配置网站URL。在创建容器的时候可以指定不同的环境变量值，这样就能改变应用的行为。

>   **TRY IT NOW**
>
>   移除现有容器，为TARGET环境变量指定新的值来运行容器：

```powershell
docker container rm -f web-ping
docker container run --env TARGET=google.com diamol/ch03-web-ping
```

这次你的输出与我的就比较相似，如图3.3：

**图3.3 来自同一个镜像的容器，发送请求给Google**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_17.jpg)

容器以不同的方式运行。首先它以交互方式来运行，由于没有使用`--detach`选项，来自应用的输出直接显示在控制台上。应用会一直运行直到通过CTRL-C结束。第二，它由ping blog.sixeyed.com改为ping google.com。

这是本章的一个要点 - Docker镜像将应用的一系列默认配置打包起来，但运行容器的时候也可以提供不同的配置。

环境变量是达到这个目标的一种简单方式。`web-ping`应用会搜索`TARGET`环境变量。该镜像为这个变量设置的值为`blog.sixeyed.com`，可以在`docker container run`命令运行容器的时候通过`--env`选项来提供不同的值。图3.4显示镜像中的容器拥有不同的配置。

**图3.4 镜像和容器中的环境变量**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_18.jpg)

宿主机也有它自己的环境变量，只是它们与容器是隔离的。每个容器都有自己的环境变量值。图3.4展示出`web-ping`应用在每个容器中都是相同的-它们使用同样的镜像，运行同样的二进制代码，由于配置不同表现出不同的行为而已。

感谢Docker镜像的作者提供的便利性，当你通过Dockerfile构建完自己的镜像后，你就知道怎么达成这个目标了。

## 3.2 编写你的第一个Dockerfile

Dockerfile是用来打包应用而编写的一段简单脚本-它包括一系列指令，镜像是它的输出成果。Dockerfile语法非常简单，使用Dockerfile能打包任何类型的应用。由于脚本语言go有很高的灵活性。对常用的任务它提供有对应的命令，对自定义的任务可以使用标准的shell命令来完成(Linux中的shell或Windows中的PowerShell)。

**代码清单3.1显示了打包`web-ping`应用的完整Dockerfile：**

```powershell
FROM diamol/node
 
ENV TARGET="blog.sixeyed.com"
ENV METHOD="HEAD"
ENV INTERVAL="3000"
 
WORKDIR /web-ping
COPY app.js .
 
CMD ["node", "/web-ping/app.js"]
```

即使你是第一次看到这个Dockerfile文件，我想你也能大致猜出它们分别是干什么的。Dockerfile指令包括`FROM`、`ENV`、`WORKDIR`、`COPY`、`CMD`；它们都是大写字母，其实这并不是必须的，只是一个约定而已。下面是每个指令的说明：

`FROM` - 每个镜像都必须继承其他的镜像。在本例中，`web-ping`镜像将使用`diamol/node`镜像作为基础镜像。该镜像安装好了NodeJS，这是`web-ping`应用运行需要的环境。

`ENV`- 设置环境变量的值。语法是 `[key] = "[value]"`,该例有三个`ENV`指令，表示设置了三种不同的环境变量。

`WORKDIR` - 在容器镜像文件系统中创建目录，并把它设置成当前的工作目录。 斜杠语法在Linux和Windows容器中都能正常工作，就是说在Linux中会创建`/web-ping`，在Windows中会创建`C:\web-ping`。

`COPY`- 从本地文件系统中拷贝文件和目录到容器镜像中。语法是 `[source path] [target path]` - 在该例中，我拷贝本地机器上的`app.js`文件到镜像中的工作目录下。

`CMD` - 指定通过镜像启动容器的时候运行的命令。 本例是指使用NodeJS来启动应用代码`app.js`。

上面这些就是在Docker中打包应用时经常会用到的指令，上面5个指令是一种很好的实践。

>   **TRY IT NOW**
>
>   不需要拷贝粘贴这个Dockerfile，在第1章下载的本书源码已经包含了这个Dockerfile文件。找到本地已下载的这个文件开始构建镜像：

```powershell
cd ch03/exercises/web-ping
ls
```

会显示三个文件：

`Dockerfile` - 包含代码清单3.1所示的内容

`app.js` - 包含应用代码

`README.md` - 空白的说明文件

如图3.5所示：

**图3.5 构建Docker镜像需要的内容**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_19.jpg)

不需要了解NodeJS和Javascript就能打包应用并在容器中运行。如果查看下`app.js`，你会发现它使用标准的NodeJS库来发起HTTP调用，并从环境变量中读取配置项。

在这个目录中包含了构建`web-ping`镜像需要的所有资源。

## 3.3 构建自己的镜像

基于Dockerfile构建镜像之前Docker需要了解一些镜像的基本情况。它需要知道镜像的名称，知道要打包成镜像的文件位置。前面已经通过终端找到了要打包镜像的文件路径，现在我们准备开始吧。

>   **TRY IT NOW**
>
>   通过运行`docker image build` 基于Dockerfile构建镜像

```powershell
docker image build --tag web-ping .
```

`--tag`参数指明了镜像的名称，最后一个参数是Dockerfile和相关文件的目录。Docker称该目录为"上下文"，点符号表示当前目录。你会看到构建命令的输出，执行Dockerfile文件中的所有指令。我的构建过程如图3.6：

**图3.6 构建web-ping Docker镜像的输出**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_20.jpg)

如果运行构建命令时出错了，首先你需要检查Docker引擎是否已经启动。确保Docker桌面应用程序在Windows或Mac下正常运行。然后检查你是否位于正确的目录。需要在有Dockerfile和app.js的`ch03-web-ping`目录下。最后检查下键入的构建命令是否正确 - 命令最后的那个点符号不能缺少 - 它会告诉Docker构建上下文是当前这个目录。

如果构建时弹出一些文件许可的警告，那是因为你正在通过Windows环境下的Docker命令构建Linux容器，幸亏Docker桌面应用程序的Linux容器模式才能执行这种操作。由于Windows没有记录类似Linux的文件权限，这个警告是告诉你从Windows机器上拷贝的文件在Docker镜像中将拥有完全的读写权限。

当看到命令行中输出类似`successfully built` 或 `successfully tagged`这样的提示语，表示镜像构建完成。它位于本地的镜像缓存中，通过Docker命令能将它显示出来。

>   **TRY IT NOW**
>
>   列出镜像名以 'w' 开头的镜像：

```powershell
docker image ls "w*"
```

你会看到`web-ping`镜像显示出来了

```powershell
docker image ls w*
REPOSITORY  TAG     IMAGE ID      CREATED          SIZE  
web-ping    latest  f2a5c430ab2a  14 minutes ago   75.3MB
```

你可以像使用从Docker Hub中下载的镜像那样使用这个镜像，应用的内容相同，也可以通过环境变量来设置配置项。

>   **TRY IT NOW**
>
>   运行自己构建的镜像容器每5秒 ping 下Docker的网站

```powershell
docker container run -e TARGET=docker.com -e INTERVAL=5000 web-ping
```

你的输出预计会与我的一样，如图3.7所示，日志的第1行会确认目标web URL是docker.com，然后每5秒ping一次。

**图3.7 运行自己构建镜像的容器**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_21.jpg)

容器是以前台方式运行，需要通过CTRL - C 方式来停止它。这会终止应用，同时容器也会进入终止状态。

你已经打包了一个能在容器中运行的简单应用，复杂应用的构建过程其实也类似。在Dockerfile文件中编写打包应用的步骤，收集需要打包到镜像中的所有资源，并决定镜像使用者如何配置应用的行为。

## 3.4 理解容器镜像以及镜像分层

在阅读本书的过程中，你会建立更多有关镜像的印象。在本章的剩余部分，我们继续使用这个简单的方法来理解镜像是如何工作的，并弄清楚容器和镜像间的关系。

Docker镜像包含打包的所有文件，这形成了容器的文件系统 - 它同时包含了镜像自身的一些元数据。包含镜像构建的短暂历史。通过这些元数据你能看清镜像每一层的命令。

>   **TRY IT NOW**
>
>   检查 web-ping 镜像的历史：

```powershell
docker image history web-ping
```

你将看到镜像分层的每一行输出，下面是我的镜像分层中的前面几行：

```powershell
> docker image history diamol/ch03-web-ping
IMAGE         CREATED       CREATED BY                                     
47eeeb7cd600  30 hours ago  /bin/sh -c #(nop) CMD ["node" "/web-ping/ap…  
<missing>     30 hours ago  /bin/sh -c #(nop) COPY file:a7cae366c9996502…  
<missing>     30 hours ago  /bin/sh -c #(nop) WORKDIR /web-ping
```

`CREATED BY` 命令是Dockerfile的指令 - 它与Dockerfile中的指令是一一对应的关系 -  因此Dockerfile中的每一行会创建一个镜像层。在这里我们将会深入介绍理论知识，因为理解镜像分层是高效使用Docker的核心知识点。

Docker镜像是一个逻辑上的镜像分层集合。镜像分层文件存储在Docker引擎的文件系统中。这就是它这么重要的原因：不同的镜像和容器能共享这些分层的镜像。假如你有许多基于NodeJS的应用，它们将共享包含NodeJS运行时环境的镜像层。图3.8显示了它是如何工作的：

**图3.8 镜像层如何逻辑性的构建到Docker镜像中**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_22.jpg)

`diamol/node`镜像底层是操作系统层，然后是NodeJS运行时环境。Linux镜像占用75M的磁盘空间(Windows容器的操作系统层稍微大点，Windows版本的镜像接近300M)。`web-ping`镜像基于`diamol/node`，所以在`web-ping`的Dockerfile中`FROM`指令引用的是`diamol/node`。在基础镜像之上打包的`app.js`文件只有几KB，那 `web-ping` 镜像总共有多大呢？ 

>   **TRY IT NOW**
>
>   通过 `docker image ls`能列出镜像列表，它也会展示镜像的大小。如果命令行没有包含过滤字符串，它将显示所有的镜像：

```powershell
docker image ls
```

输出可能与我在图3.9中显示的类似：

**图3.9 列出镜像来观察他们的大小**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_23.jpg)

看起来所有的NodeJS镜像占用了同样大小的空间 - 在Linux上占用 75M。三个镜像分别是：`diamol/node` ,在`diamol/ch03-web-ping` 中从Docker Hub拉取下来的原始示例应用，以及你自己构建的版本`web-ping`。它们应该共享基础镜像层，但是`docker image ls`命令的输出显示它们每个都是 75M，合起来就是 75 * 3 = 225M。

其实，Size 那一列显示的是镜像的逻辑大小 - 意思是当你的宿主机上没有其他任何镜像时占用的磁盘空间大小。如果磁盘上有其他镜像共享的层，Docker占用的空间会小很多。在镜像列表中无法看到这一点，但是Docker的系统命令会告诉你这些详细信息。

>   **TRY IT NOW**
>
>   镜像列表展示当前镜像大小总和为363.96M，这仅仅是总的逻辑大小。`system df`命令能准确显示Docker占用了多少空间。

```powershell
docker system df 
```

通过图3.10可以看出，镜像缓存实际使用了202.2M - 意味着163M的镜像层在镜像之间共享，节省了45%的空间。当在运行阶段有大量的应用镜像共享同样的基础镜像，通过重用就能节省大量的磁盘空间。基础镜像层可能有Java、.NET Core或PHP - 不管是哪种技术栈，Docker都能提供一致的行为。

**图3.10 检查Docker镜像的磁盘空间占用**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_24.jpg)

现在到了理论知识的最后一部分了。如果镜像分层正在共享中，意味着它不能修改 -  否则一个镜像中的改变就会级联影响到共享镜像层的其他镜像中。因此Docker引擎强制镜像层是只读的。利用这个特性通过优化Dockerfile可以构建更小、更快的镜像。

## 3.5 优化Dockerfile 充分利用镜像缓存

`web-ping`镜像中有一层包含应用的JavaScript脚本。如果你修改这个文件并重新构建镜像，就能得到新的镜像。Docker为镜像中的分层定义了一个编号，因此当你改变位于中间编号的层，Docker会重新构建该层之后的每一层。

>   **TRY IT NOW**
>
>   修改位于 `ch03-web-ping`目录下的 `app.js`文件。甚至不需要修改代码， 只需要在文件的最后添加一个空行。然后构建一个新版本的镜像：

```powershell
docker image build -t web-ping:v2 .
```

你将看到如图3.11所示输出的内容。第2 - 5步构建使用缓存中的层，第6 - 7 步生成新的层：

**图3.11 构建一个能利用缓存的镜像**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-7/Figures/Images_01-03_25.png)

每一条Dockerfile中的指令都会生成一个镜像层，如果在构建时指令没有改过，即将构建的指令内容是相同的，Docker会知道使用缓存中的镜像，否则它将执行Dockerfile中的指令并重新生成镜像。当输入和输出都相同时，Docker会使用缓存中存在的镜像。

Docker通过对输入内容生成hash值来判断该镜像是否与缓存中的镜像匹配，就像是给输出内容添加一个数字手印。Hash值是由Dockerfile中的指令和将要复制到文件中的内容一起产生的。如果在现有的镜像中没有匹配的hash值，Docker会执行所有的指令 - 这将使缓存失效。如果缓存失效了，Docker将执行所有的指令，即使它们实际上没有改变。

在示例镜像中做很小的改动也会对镜像的构建产生影响。自从上一次构建完，`app.js` 文件发生了改变，因此第6步的 `COPY`指令需要执行，第7步的`CMD` 指令虽然与上一次的构建相同，但由于在第6步缓存被打破了，这个指令依然要重新执行一遍。

编写Dockerfile的指令顺序应该按照指令被修改的频率来排列 - 不经常改变的指令应该位于Dockerfile的前面，经常会改变的指令位于Dockerfile的末尾。这样做的目的是我们尽量只执行最后几个指令，其他的都使用缓存即可。共享镜像可以给你节省时间、磁盘空间以及网络带宽。

虽然在`web-ping`镜像中只有7条指令，依然可以对其进行优化。`CMD`指令不必位于Dockerfile的末尾，它可以在`FROM`之后的任何其他地方，对结果都不会产生影响。其实不需要改动这个指令，只需要把它往文件开头的位置移动下。一个 `ENV`指令可以设置多个环境变量，因此三个独立的`ENV`指令可以合并起来。优化后的Dockerfile如代码清单3.2所示：

**代码清单3.2 优化后的web-ping Dockerfile文件**

```powershell
FROM diamol/node
 
CMD ["node", "/web-ping/app.js"]
 
ENV TARGET="blog.sixeyed.com" \
    METHOD="HEAD" \
    INTERVAL="3000"
 
WORKDIR /web-ping
COPY app.js .
```

>   **TRY IT NOW**
>
>   优化后的Dockerfile也位于源码仓库中这一章下面，切换到web-ping-optimized目录下，基于新的Dockerfile构建镜像：

```powershell
cd ../web-ping-optimized
docker image build -t web-ping:v3 .
```

不需要太在意与上次构建不同的地方。相比上次的7个步骤这次只有5步，但最终的结果是一样的 - 通过该镜像运行的容器它的行为与之前的版本是一致的。一旦改变了app.js中的代码并重新构建，除了最后一步外所有其他的步骤都会利用缓存，因为只有这一步涉及到的文件发生了改变。

以上就是本章关于镜像构建的所有内容。你已经了解了Dockerfile的语法、熟悉了编写Dockerfile的关键指令，并学会了如何通过Docker CLI构建、运行镜像。

学完这一章，有两件事希望你在每次构建镜像的过程中要记住：优化Dockerfile，确保你的镜像是可移植的这样在部署到不同的环境时可以重用相同的镜像。其实你只需要关注如何构建Dockerfile指令，并确保应用程序能从容器中读取配置值就够了。这意味着你可以快速构建镜像，可以将在测试环境进行优化后的镜像直接部署到生产环境。

## 3.6 实验

现在，到了实验时间。这次的目标是回答这个问题：如何在不使用Dockerfile的情况下构建镜像？Dockerfile的作用是完成应用的自动化部署，但并不是所有事情都能自动化。有时你需要运行一些应用，手动完成一些步骤，因为它们无法通过脚本来完成。

该实验是一个简单版本。你将以Docker Hub中名为`diamol/ch03-lab`的镜像开始，该镜像在路径` /diamol/ch03.txt`上有个文件。你需要更新该文件并在文件末尾加上你的名字。然后利用这个更改后的文件来生成自己的镜像。禁止使用Dockerfile :)

在本书的代码仓库中有一个示例解决方案，如果你需要的话可以在[https://github.com/sixeyed/diamol/tree/master/cha03/lab](https://github.com/sixeyed/diamol/tree/master/cha03/lab) 找到。

下面是一些提示希望你能用到：

记得通过 `-it` 参数以交互模式运行容器

容器退出后容器的文件系统依然存在

依然有许多容器相关的命令你没有用过， `docker container --help` 能向你展示以上两条提示的详细内容来帮助你完成该实验。



## 本章总结：

![img](_media/chapter3-summary.png)

>   **[第2章](/zh-cn/docker/docker-4-weeks/chapter-2.md)**                                                  **[第4章](/zh-cn/docker/docker-4-weeks/chapter-4.md)**

