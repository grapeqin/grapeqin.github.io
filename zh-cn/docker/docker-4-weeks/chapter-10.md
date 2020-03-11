

>   **[第9 章](/zh-cn/docker/docker-4-weeks/chapter-9 .md)**                                                  **[第11章](/zh-cn/docker/docker-4-weeks/chapter-11.md)**

# 10 使用Docker Compose部署多套环境

在第7章我们介绍过Docker Compose，学会了如何使用YAML来描述多容器应用，以及如何使用Docker Compose命令行工具来管理这些应用。在那之后我们继续介绍了容器化应用为生产准备的有关健康检查和监控相关的知识。现在我们再回到Compose - 因为并不是所有的环境都需要具备生产级别的要求。可移植性是Docker很重要的一个特性。不管在哪里部署打包成镜像的应用，它的行为都是一样的，这对应用来说非常重要因为它消除了环境之间的差异性。

只要软件部署涉及到手动操作流程环境差异性就会存在。丢失了更新或忘记了依赖 - 因为生产环境与用户测试环境不同，与系统测试环境也不同。部署失败通常都是由于环境的差异导致的，定位缺失的步骤会耗费大量的时间和精力最终才能修正部署。使用Docker就可以避免这个问题因为Docker把应用的依赖也打包到镜像中了，但你依然需要保证不同环境下应用行为的灵活性。在本章我们会介绍Docker Compose 的高级特性，它们能提供这项能力。

## 10.1 使用Docker Compose部署多个应用

Docker Compose是在单个Docker引擎运行多容器应用的工具。对开发者友好，而且大量运用在非生产环境。组织一般会在不同的环境运行多个版本的应用 - 1.5版本在生产环境运行，1.5.1版本在hotfix环境测试，1.6版本在用户测试环境处于测试的尾声，1.7版本在系统测试环境。这些非生产环境的应用不需要生产环境级别的规模和性能，对Docker Compose来说这是一个非常好的使用场景，它能运行这些环境并最大化的利用硬件资源。

尽管如此，环境之间依然会有许多不同的地方。你不能让多个容器都监听80端口的流量，或向服务器上同一位置的文件写数据。通过设计Docker Compose文件可以支持这个特性，但首先需要理解Docker Compose 是如何标记不同的Docker资源属于同一个应用的。它是通过命名规范和标签完成的，如果你只是要运行一个应用的拷贝则只需要使用默认配置即可。

>   **TRY IT NOW**
>
>   开启终端并浏览本章的练习。先运行两个之前已经用过的应用，在运行待办列表应用：

```powershell
cd ./ch10/exercises
 
# run the random number app from chapter 8:
docker-compose -f ./numbers/docker-compose.yml up -d
 
# run the to-do list app from chapter 6:
docker-compose -f ./todo-list/docker-compose.yml up -d
 
# and try another copy of the to-do list:
docker-compose -f ./todo-list/docker-compose.yml up -d
```

输出如图10.1所示。通过不同目录的Compose文件可以启动多个应用，同一目录的Compose文件无法启动第2个应用示例。Docker Compose认为你让它运行一个已经在运行的应用，所以它不会启动新的容器：

**图10.1 重复执行Docker Compose命令不会运行应用的第2份拷贝**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_01.png)

Docker Compose使用项目的概念来标识哪些资源集属于同一个项目，它使用保存Compose文件的目录名作为默认的项目名。当它创建资源时Compose会使用项目名作为前缀，如果是容器还会有个数值计数器作为后缀。因此如果有个`app1`的目录下有个Compose file，它定义了一个`web`的服务和一个`disk`的数据卷，Docker Compose实际上会创建名为`app1_disk`的数据卷和名为`app1_web_1`的容器。容器名后面的计数器支持扩展，因此如果你扩展这个web服务为两个实例，那么新容器名为`app1_web_2`。

**图10.2 显示如何为待办列表应用构建容器名**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_02.png)

可以覆盖Docker Compose默认使用的项目名 - 这也是我们可以在单个Docker引擎上运行应用的多个实例的原因。

>   **TRY IT NOW**
>
>   我们已经运行了一个待办列表应用，通过指定项目名可以启动另外一个。该网站使用随机端口，如果想访问它需要指定一个具体的端口：

```powershell
docker-compose -f ./todo-list/docker-compose.yml -p todo-test up -d
 
docker container ls
 
docker container port todo-test_todo-web_1 80
```

输出如图10.3所示。指定项目名后在Compose的概念里就意味着一个新的应用，没有资源能匹配到这个项目名因此Compose会创建一个新的容器。有了命名规范，所以我知道新容器为`todo-test_todo-web_1`。Docker CLI可以使用`container port`命令找到容器发布的端口，我可以通过该命令和已生成的容器名找到应用端口：

**图10.3 基于一份Compose文件指定项目名让你运行一个应用的多个实例**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_03.png)

该方法允许你为应用运行多个实例。我可以为随机数应用部署另外一个实例，使用同一个Compose文件但指定一个不同的项目名。这非常有用但对大多数场景来说你期望有更多的控制 - 必须为每次发布找到使用的随机端口对于一个好的工作流程和测试团队来说都不是太友好。为了支持各个环境独立的配置，可以创建多份Compose文件然后根据需要进行修改，但Compose提供了一个更好的办法。

## 10.2 Docker Compose覆盖文件

团队一般是通过Docker Compose运行多个不同的应用配置文件来解决这个问题，这会导致存在大量的Compose文件 - 每个文件对应一套环境。这能够解决问题但它缺乏可维护性，因为这些Compose文件90%的内容是重复的，意味着它们很难保持同步更新最终会产生很多环境差异性问题。文件覆盖是个简洁的方式。Docker Compose允许你把多个文件合并起来，在合并过程中位于后面文件的属性会覆盖前面文件的属性。

图10.4 显示了如何利用覆盖来组织易于维护的Compose 文件。我们从一个包含了应用基本结构的`docker-compose.yml`文件开始，它里面包含了所有环境通用的服务定义和配置项。每套环境都添加了独有配置的覆盖文件 - 它没有与核心的`docker-compose.yml`配置项重复的内容。

**图10.4 通过文件覆盖移除重复内容，覆盖的文件包含了环境独有的配置**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_04.png)

这个方法可维护性就很好。如果你想针对所有环境做出改变 - 比如使用最新版本的镜像 - 只需要更改一次核心的Compose文件它就会传播到所有的环境。如果只想改变某个环境只需要改变这个环境独有的文件。每个环境独有的覆盖文件也会提供清晰的文档来描述不同环境的区别。

代码清单10.1 展示了一个非常简单的示例，核心的Compose文件指定了大部分应用属性，覆盖文件部分改变了镜像标签，因此这次部署将使用v2版本的 to-do 应用。

代码清单10.1 更新了单个属性值的Docker Compose覆盖文件

```powershell
# from docker-compose.yml - the core app specification:
services:
  todo-web:
    image: diamol/ch06-todo-list
    ports:
      - 80
    environment:
      - Database:Provider=Sqlite
    networks:
      - app-net
 
# and from docker-compose-v2.yml - the version override file:
services:
  todo-web:
    image: diamol/ch06-todo-list:v2
```

在覆盖文件中只需要指定你关心的属性，但你需要保持与主Compose文件相同的结构，这样Docker Compose才能把文件合并起来。本例中的Docker Compose覆盖文件只改变了`image`属性，它位于`services`块下面的`todo-web`块下，只有这样Compose才能匹配到文件中完整的服务定义。Docker Compose文件是在命令行`docker-compose`中指定多个文件来合并的。这里`config`命令非常有用 - 它验证输入文件的内容，如果输入有效它会给出最终的输出。通过这个命令可以看到合并文件后的效果。

>   **TRY IT NOW**
>
>   在本章的练习目录下，使用Docker Compose合并代码清单10.1所示的文件然后打印输出：

```powershell
docker-compose -f ./todo-list/docker-compose.yml -f ./todo-list/docker-compose-v2.yml config
```

`config`命令并没有部署应用，它只是验证配置。你将看到两个文件合并后的输出。除了`image`属性是被第2个Compose文件覆盖的以外，其他所有的属性都来自于核心的那个Compose文件 - 如图10.5所示：

**图10.5 采用覆盖文件合并Compose文件并显示其结果**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_05.png)

Docker Compose是根据命令中列出的Compose文件的顺序来执行覆盖的，即命令右边的文件会覆盖左边的文件。这非常重要，顺序错误你会得到非预期的结果 - `config`命令在这很有用因为它向你展示了完整的Compose文件。输出内容按照字母顺序排序，你会先看到networks，然后是services，最后是Compose的version号 - 它们没有按照文件定义的顺序显示，这非常有用。你可以把这个命令作为自动化部署的一部分，提交合并后的文件到源码控制系统 - 按字母顺序输出的内容在比较版本方面非常有用。

覆盖镜像的标签是个快速示例。在`numbers`目录下有几个关于随机数应用的具有实际意义的Compose文件：

-   `docker-compose.yml` - 核心的应用定义，它未定义web和api服务需要的端口和网络
-   `docker-compose-dev.yml` - 开发环境中运行的应用。它添加了网络，为服务添加了发布的端口，并禁用了健康和依赖检查。这也是开发者能快速启动并运行的原因。
-   `docker-compose-test.yml` - 测试环境中运行的应用。为web应用指定网络，添加健康检查参数并发布端口，但API服务只对内服务不发布任何端口。
-   `docker-compose-uat.yml` - 用户验收测试环境中运行的应用。设置网络，为网站发布标准的80端口，设置服务重启并为服务设置更严格的健康检查参数

代码清单10.2 显示了dev覆盖文件的全部内容 - 这不是一个完整的应用规范显示因为它里面没有指定镜像。这里定义的这些值都会合并到核心Compose文件中 - 添加新属性，或者如果发现有跟核心Compose文件匹配的属性就覆盖它的值。

代码清单10.2

```powershell
services:
  numbers-api:
    ports:
      - "8087:80"
    healthcheck:
      disable: true
 
  numbers-web:
    entrypoint:
      - dotnet
      - Numbers.Web.dll
    ports:
      - "8088:80"
 
networks:
  app-net:
    name: numbers-dev
```

其他的覆盖文件遵守同样的规范。每套环境为web应用和API指定不同的端口这样就能在一台机器上运行所有这些环境。

>   **TRY IT NOW**
>
>   清理所有存在的容器 ，然后在多个环境运行随机数应用 - 每个环境都需要一个项目名和几个正确的Compose文件：

```powershell
# clear down any existing containers
docker container rm -f $(docker container ls -aq)
 
# run the app in dev configuration:
docker-compose -f ./numbers/docker-compose.yml -f ./numbers/docker-compose-dev.yml -p numbers-dev up -d
 
# and the test setup:
docker-compose -f ./numbers/docker-compose.yml -f ./numbers/docker-compose-test.yml -p numbers-test up -d
 
# and UAT:
docker-compose -f ./numbers/docker-compose.yml -f ./numbers/docker-compose-uat.yml -p numbers-uat up -d
```

现在有三份应用的实例在运行，它们彼此之间是隔离的，因为每个项目都在使用它自己的Docker网络。在一个组织中这些环境会运行在一台服务器上，团队可以使用正确的端口来选择不同的环境 - 比如使用80端口来选择UAT环境，使用8080端口选择系统测试环境，使用8088端口来选择开发团队的集成测试。图10.6 显示了创建的网络和容器：

**图10.6 在一台服务器上通过Docker运行多套相互隔离的应用环境**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_06.png)

现在三个项目位于独立的环境中 - http://localhost是UAT环境，http://localhost:8080/是系统测试，http://localhost:8088/是开发环境。访问任何一个地址都能看到同样的应用，但每个web容器都只能看到同一个网络内部的API容器。这保持了应用的独立性，如果你继续访问开发环境中的随机数生成功能API服务会挂，但系统测试和UAT环境中的API服务仍然正常。各个环境的容器使用DNS域名来通信，Docker会限制容器网络中的流量。图10.7 显示了网络隔离是如何保证环境的隔离的：

**图10.7 在Docker引擎中借助网络隔离同时运行多套环境**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_07.png)

Docker Compose是一个客户端工具，你需要访问所有的Compose文件才能管理应用 - 你也要记得使用的项目名。如果你想清理test环境，只需要运行`docker-compose down`就能移除容器和网络，但对这些环境来说单单这个命令不会凑效，因为Compose需要所有在up命令中使用的同样的文件列表和项目信息来匹配相关资源。

>   **TRY IT NOW**
>
>   我们移除test环境。你可以尝试各种不同版本的`down`命令，但唯一能凑效的是使用与最开始容器启动所需要的同样文件列表和项目名：

```powershell
# this would work if we'd used the default docker-compose.yml file:
docker-compose down
 
# this would work if we'd used override files without a project name:
docker-compose -f ./numbers/docker-compose.yml -f ./numbers/docker-compose-test.yml down
 
# but we specified a project name so we need to include that too:
docker-compose -f ./numbers/docker-compose.yml -f ./numbers/docker-compose-test.yml -p numbers-test down
```

输出如图10.8所示。你可能已经猜到Compose是无法为应用标识运行中的资源除非提供能匹配的文件和项目名，因此第一个命令不能移除任何东西。第二个命令尝试删除容器网络，即使有应用容器连到这个网络上的：

**图10.8 Compose使用同样的文件和项目名来管理应用**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_08.png)

[https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_08.png](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_08.png)

由于在Compose覆盖文件中有定义网络名，所以第二个命令会有这样的结果。在第二个`down`命令中我没有指定项目名，它使用默认的目录名`numbers`作为项目名。Compose会寻找名为`numbers_numbers-web_1`和`numbers_numbers-api_1`的容器，但是找不到，因为它们实际是通过`numbers-test`作为前缀项目名创建的。Compose会认为这些容器已经消失了，它只需要清理网络，该网络是在Compose文件中没有使用项目名前缀命名的。Compose尝试删除该网络，幸运的是Docker不会移除还有容器在使用的网络。

上面用大量篇幅向你介绍了如何使用Docker Compose。对非生产环境而言这是个优秀的工具，它通过在单机上部署几十上百个应用来最大化利用硬件资源。覆盖文件让你重用应用定义文件并能识别不同环境的区别，但你需要了解管理的开销 - 需要编制部署和停止应用的脚本并实现自动化。

## 10.3 使用环境变量和加密文件来注入配置

使用Docker网络和Compose覆盖文件来获取不同环境下的区别使你能轻松隔离应用，但你仍然需要改变环境之间的应用配置。大部分应用都能通过环境变量或文件来读取配置项，Compose对这些方法有很好的支持。这一节我会介绍所有的选项，这里我们会继续深入Compose - 它能帮你理解在做应用配置时所有的选择，这样就能选择最适合你的方案。

继续回到待办列表应用来做这些练习。这个应用的Docker镜像用来从环境变量和文件中获取配置，这里有三个配置需要区分不同的环境：

-   日志 - 日志级别的详细信息。开发环境中的日志一般比较冗长，但测试和生产环境就会少很多
-   数据库提供者 - 在容器中使用简单的数据文件，或者独立的数据库
-   数据库连接字符串 - 如果应用没有使用本地数据文件，提供连接数据库的详细信息

我将使用覆盖文件为不同的环境注入配置，对每个配置我都使用不同的方法，这样就能向你展示Docker Compose提供的所有方法。代码清单10.3向你展示了核心的Compose文件；它提供了web应用的基本信息，以及一个加密的配置文件：

代码清单10.3 Compose文件指定了需要读取加密内容的web服务 

```powershell
services:
  todo-web:
    image: diamol/ch06-todo-list
    secrets:
      - source: todo-db-connection
        target: /app/config/secrets.json
```

Secrets是注入配置的一种有效方式 - Docker Compose，Docker Swarm和Kubernetes都支持这种方式。在Compose文件中为secret指明了source和target。source是从容器运行时加载的位置，target是容器内呈现的文件路径。

该secret指定来自`todo-db-connection`，意味着在Compose文件中需要定义一个该名称的secret。该secret的内容会从`/app/config/secrets.json`这个目标路径加载到容器中 - 这也是应用搜索配置的路径之一。

这个Compose文件无效因为没有secrets部分，因为服务定义中有使用所以`todo-db-connection`是必要的。代码清单10.4 显示了用于开发的覆盖文件，它为服务设置了许多配置并指定了secret：

代码清单10.4 开发环境重写了配置和加密设置

```powershell
services:
  todo-web:
    ports:
      - 8089:80
    environment:
      - Database:Provider=Sqlite
    env_file:
      - ./config/logging.debug.env
 
secrets:
  todo-db-connection:
    file: ./config/empty.json
```

在这个覆盖文件中有三个属性会注入应用配置来改变容器中应用的行为，你可以使用任何一种方法，每一种方法其实都能凑效：

-   `environment` 向容器添加一个环境变量；这个设置配置应用使用SQLite数据库 - 一个简单的数据文件。这是设置配置值的最简单方式，从Compose文件中可以清楚的看到配置值
-   `env_file`包含了一个文本文件的路径，该文本文件的内容会加载到容器中作为环境变量。文本文件中的每一行会作为环境变量，通过等于号来分离名字和值。该文件的内容设置日志配置。使用环境变量配置文件是在不同组件间共享配置的一种简单方式，因为每个组件都是引用这个文件而不是重复一堆环境变量
-   `secrets` 在Compose YAML文件中是一个顶级资源，就像`services`和`networks`一样。它包含了`todo-db-connection`这个实际的source，是本地文件系统的一个文件。本例中没有独立的数据库供应用来连接，因此它为secret提供了一个空的JSON文件 - 应用会读取这个文件，但不会生效任何配置。

>   **TRY IT NOW**
>
>   你可以使用Compose文件和`todo-list-configured`目录下的覆盖文件来运行开发配置下的应用。通过cURL发送请求到web应用，然后检查容器正在记录许多明细：

```powershell
# remove existing containers:
docker container rm -f $(docker container ls -aq)
 
# bring up the app with config overrides - for Linux containers:
docker-compose -f ./todo-list-configured/docker-compose.yml -f ./todo-list-configured/docker-compose-dev.yml -p todo-dev up -d
 
# OR for Windows container, which use different file paths for secrets:
docker-compose -f ./todo-list-configured/docker-compose.yml -f ./todo-list-configured/docker-compose-dev.yml -f ./todo-list-configured/docker-compose-dev-windows.yml -p todo-dev up -d
 
# send some traffic into the app:
curl http://localhost:8089/list
 
# check the logs:
docker container logs --tail 4 todo-dev_todo-web_1
```

输出如图10.9所示。Docker Compose为每个应用都会使用一个网络，它会创建一个默认的网络并让所有的容器连接到它上面 - 即使在Compose文件中没有设置网络。本例中最新的几行日志显示了应用使用的SQL数据库命令 - 你那里的日志可能会有些不同，但如果你检查整个日志文件也会找到SQL语句。这表明日志配置已经生效。

**图10.9 在Docker Compose中通过配置设置来改变应用行为**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_09.png)

开发环境为应用使用环境变量和secrets - 值来自于Compose文件和加载容器中的配置文件。这里还有一个test环境采用了Compose支持的另外一种方法，使用宿主机的环境变量为容器提供属性值。这使得部署的可移植性更高，因为在不改变Compose文件自身的前提下可以改变环境变量值 - 这非常有用如果你想用不同的配置在其他服务器上复制第二套测试环境。

代码清单10.5 显示了`todo-web`服务的规范：

代码清单10.5 

```powershell
todo-web:
    ports:
      - "${TODO_WEB_PORT}:80"
    environment:
      - Database:Provider=Postgres
    env_file:
      - ./config/logging.information.env
    networks:
      - app-net
```

$和{} 表示的端口会用同名的环境变量值来代替。如果在运行Docker Compose的机器上设置一个变量，名字为`TODO_WEB_PORT`值为`8877`，接着Compose会注入该值，端口规范会变成`"8877:80"`。服务规范位于文件`docker-compose-test.yml`，它包含一个数据库服务，以及一个用在连接数据库容器的secret。

可以采用与开发环境同样的方式，指定Compose文件和项目名来运行test环境，还有关于Compose最后一种配置特性能帮你简化配置。如果Compose在当前目录找到一个名为`.env`的文件，它会被当做环境变量文件，从中读取的内容作为环境变量，在运行命令前会填充环境变量。

>   **TRY IT NOW**
>
>   进入configured的to-do 列表应用，不指定任何参数运行Docker Compose：

```powershell
cd ./todo-list-configured
 
# OR for Windows containers - using different file paths:
cd ./todo-list-configured-windows
 
 
docker-compose up -d
```

图10.10 显示Compose已经创建了web和database容器，尽管核心Compose文件并没有指定数据库服务，同时尽管我没有指定项目名，它使用`todo_ch10`作为项目名。.env文件指定Compose默认情况下运行test环境，并且不需要你指定test覆盖文件：

**图10.10 使用环境文件为Docker指定默认的Compose文件和项目名**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_10.png)

这是因为`.env`文件包含环境变量配置，可以用在不同的地方。第1个用途是作为容器属性配置，比如web应用的端口；第2个用途是为Compose命令自身，列出了要使用的文件和项目名。代码清单10.6显示了完整的`.env`文件：

代码清单10.6 使用环境文件配置容器和Compose

```powershell
# container configuration - ports to publish:
TODO_WEB_PORT=8877
TODO_DB_PORT=5432
 
# compose configuration - files and project name:
COMPOSE_PATH_SEPARATOR=;
COMPOSE_FILE=docker-compose.yml;docker-compose-test.yml
COMPOSE_PROJECT_NAME=todo_ch10
```

环境文件配置好在测试环境中运行的应用程序的默认设置 - 你可以轻松地修改它，使开发环境配置成为默认配置。将环境文件与Compose文件放在一起有助于记录哪些文件表示哪些环境，但请注意，Docker Compose只会查找名为`.env`的文件，你无法指定文件名，因此无法在多个环境文件的环境之间轻松切换。

浏览Docker Compose的配置选项花了我们一些时间。在使用Docker的过程中，将会处理大量的Compose文件，因此你需要熟悉所有的选项 - 这里我对它们进行一下总结，实践表明有一些选项比其他的更有用：

-   使用`environment`属性来设置环境变量值是最简单的，它让来自Compose文件的应用配置易读。这些设置是纯文本，因此你不能使用它们来配置敏感的数据如连接字符串或API秘钥；
-   加载配置中的`secret`属性是最可靠的，因为所有的容器运行时都支持该特性而且这种方式也能用于配置敏感数据。secret的来源在使用Compose时可以是一个本地文件，或者是存储在Docker Swarm或Kubernetes加密的数据。不管来源是什么，secret的内容都会加载到容器中的文件供应用读取；
-   在文件中存储设置然后通过`environment_file`把它们加载到容器中，对于需要在不同环境间共享很多配置的场景非常有用。Compose读取本地文件，然后设置这些独立的值作为环境变量属性，当连接远程Docker引擎时也能使用本地环境文件；
-   Compose的环境文件`.env`对想把任一环境的配置设成默认的部署目标非常有用。

## 10.4 采用扩展字段减少重复

此时，你可能会认为Docker Compose已经为所有的场景提供了足够的配置方案。但实际上它只是提供了一个简单的规范，在使用它的过程中会遇到许多限制。最常见的问题之一就是如何解决服务之间共享大量的相同配置时Compose文件的膨胀。

在本节我会介绍Compose的最后一个重要特性，该特性解决了这个问题 - 使用扩展字段在某个位置定义一个YAML块，这样可以在整个Compose文件中使用这些块。扩展字段是Compose的一个强大但未得到充分利用的特性。它们消除了大量的重复和潜在的错误，在习惯了YAML合并语法之后使用起来非常简单。

在本章练习的`image-gallery`目录，有个`docker-compose-prod.yml`文件使用了扩展字段。代码清单10.7展示了如何定义扩展字段，将它们声明在任何顶级块(服务、网络等)之外，并使用`&`符号为它们命名：

代码清单10.7 在Docker Compose文件的开头定义定义扩展字段

```powershell
x-labels: &logging
  logging: 
    options:
      max-size: '100m'
      max-file: '10'
 
x-labels: &labels
  app-name: image-gallery
```

这两个扩展字段叫做`logging`和`labels`。日志扩展为容器指定配置，可以直接用在服务定义里面。标签扩展指定了一对key-value标签可以用在服务定义中的`labels`字段里面。需要注意这两个定义的区别 - 日志字段包含日志属性，意味着它可以直接用在服务中。标签字段没有包含标签属性，它必须在已存在的标签里面使用。代码清单10.8 清楚的展示了一个服务定义如何使用这两种扩展：

代码清单10.8 通过YAML合并在服务定义中使用扩展

```powershell
services:
 
  iotd:
    ports:
      - 8080:80
    <<: *logging
    labels:
      <<: *labels
      public: api
```

扩展字段与`<<:`语法一起使用，然后是字段名 - 以`*`作为前缀。`<<:*logging`将在YAML引用的地方合并`logging`扩展字段。当Compose处理此文件时，它将从日志扩展中将日志部分添加到服务中，并将额外的标签添加到现有的标签部分，合并来自`labels`扩展字段的值。

>   **TRY IT NOW**
>
>   不需要运行应用也能了解Compose是如何处理这个文件的，只需要运行`config`命令即可。它会验证所有的输入并输出最终的Compose文件，包含合并到服务中的扩展字段：

```powershell
# browse to the image-gallery folder under ch10/exercises:
cd ../image-gallery
 
# check config for the production override:
docker-compose -f ./docker-compose.yml -f ./docker-compose-prod.yml config
```

输出如图10.11所示 - 我没有显示完整的输出，有足够的内容来显示正在合并的扩展字段即可：

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_11.png)

扩展字段是确保Compose文件最佳实践的一种有用方法 - 使用相同的日志设置和容器标签是为所有服务设置标准的一个好例子。它不是每个应用程序都会使用的工具，但是当你准备复制和粘贴大块YAML内容时，最好选择这个较好的方法。它有个很大的限制，扩展字段不能跨多个Compose文件使用，所以你不能在core Compose文件中定义一个扩展，然后在覆盖文件中使用它。这是YAML而不是Compose的限制，需要注意。

## 10.5 理解Docker的配置工作流程

在源码控制系统的一组工件中管理整个系统的部署配置是非常有价值的。只需获取该版本的源代码并运行部署脚本，就能部署任何版本的应用程序。它还允许开发人员在本地运行生产堆栈并在他们自己的环境中重现bug以快速修复它。

环境之间总是存在差异，Docker Compose允许你配置环境之间的差异，同时提供源代码控制的部署构件。本章我们已经看到Docker Compose定义不同环境的差异性主要聚焦在三个领域：

应用组合 - 不是每个环境都要运行完整的技术栈，开发人员不会使用监控仪表盘功能，应用在test环境会使用容器化的数据库，但是在生产会使用云数据库。覆盖文件使你可以整洁地完成这一任务，共享公共服务并在每个环境中添加特定的服务；

容器配置 - 更改属性以匹配环境的需求和功能。发布的端口必须是唯一的，这样它们才不会与其他容器发生冲突，数据卷路径可能在测试环境中使用本地驱动器，但在生产环境中使用共享存储。覆盖文件让每个应用程序有独立的Docker网络，允许你在一个服务器上运行多个环境；

应用配置 - 容器中应用的行为会伴随环境的改变而改变。这可能会改变应用程序的日志记录数量，或者用来存储本地数据的缓存大小，或者打开或关闭整个功能。你可以使用覆盖文件、环境文件和secrets的任意一种方案让Compose来实现。

图10.12显示了我们在10.3节中运行的待办事项列表应用程序。开发环境和测试环境完全不同 - 在开发环境中，应用程序被配置为使用本地数据库文件，在测试环境中，Compose配置为使用容器数据库。但是每个环境使用独立的网络和唯一的端口，所以它们可以在同一台机器上运行，如果开发人员需要启动一个本地测试环境，并查看它与开发版本的区别，这是完美的方案。

**图10.12 使用Docker Compose为同样的应用定义不同的环境**

![img](https://dpzbhybb2pdcj.cloudfront.net/stoneman/v-8/Figures/10_12.png)

最重要的一点是，配置工作流在每个环境中使用相同的Docker镜像。构建过程将生成已通过所有自动化测试的容器镜像的标记版本。这是一个候选版本，你可以使用Compose文件中的配置将其部署到冒烟测试环境中。当它通过冒烟测试时，它会移动到下一个环境，该环境使用相同的镜像并应用来自Compose的新配置。如果测试全部通过，那么你最终将发布该版本，然后使用Docker Swarm或Kubernetes部署清单将同样的容器镜像部署到生产环境中。发布的软件与通过所有测试的软件完全相同，只是它的生产行为由容器平台提供。

## 10.6 实验

本实验我希望你能为待办应用构建自己的环境变量集合。接下来你将合并开发环境和测试环境，使它们能运行在一台机器上。

开发环境是默认的环境，可以使用`docker-compose up`来运行它，它的配置为：

-   使用本地数据库文件
-   发布端口`8089`
-   运行v2版本的待办应用

测试环境需要指定特定的Compose文件和项目名，它的配置为：

-   使用独立的数据库容器
-   为数据库存储使用数据卷
-   发布端口`8080`
-   使用最近一个版本的待办应用镜像

它们与本章中的`todo-list-configured`练习非常相似。主要的区别是数据卷 - 数据库容器使用名为`PGDATA`的环境变量来设置数据文件应该写到哪里。你可以参考数据卷规范查看如何在Compose文件中使用。

如本章已经介绍的，你有许多的方法可以解决这个问题。我的解决方法为：

https://github.com/sixeyed/diamol/blob/master/ch10/lab/README.md

>   **[第9 章](/zh-cn/docker/docker-4-weeks/chapter-9 .md)**                                                  **[第11章](/zh-cn/docker/docker-4-weeks/chapter-11.md)**

