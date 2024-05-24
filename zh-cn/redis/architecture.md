# Redis 架构

Redis是一种基于内存的键值对数据存储。Redis是最流行的键值存储方案。世界上所有的大型IT品牌都在使用Redis。
Amazon弹性缓存支持Redis，使得Redis成为一个非常强大的、我们必须了解的键值数据存储。在这篇文章中，我将为你简要介绍redis架构。

## 什么是内存键值存储？

键值存储是指数据以键值形式存储的一种系统。我们说内存键值存储，主要是指键值对存储在内存(RAM)中。因此我们可以说Redis是把数据以键值对形式存储在内存中。

在Redis中，键是个字符串，但值可以是字符串、列表、集合、有序集合或哈希表。

如下是Redis键值对的示例

```markdown
name="narayan"
profession=["web", "mobile"]
```

`name` 和 `profession` 是键。与之对应的值位于等号右边。

## 相比DBMS Redis的优势与劣势

关系型数据库把数据存储在二级存储使得读写操作都很慢。Redis把数据存储在一级存储所以数据读写会非常快。

一级存储资源是有限的(相比二级存储空间更小、更昂贵)因此Redis不能存储大文件或二进制数据。它只能存储那些需要快速读取、更新和插入的小文本信息。如果我们
写入的数据超过了现存内存大小将会收到错误。

## Redis单机架构

Redis架构包含2个主要的部分：Redis客户端和Redis服务。

![Redis单机架构](http://qnimate.com/wp-content/uploads/2014/05/redis-client-server.jpg)

Redis客户端和服务端可以位于同一台电脑或两台不同的电脑。

Redis服务致力于把数据存储在内存。它处理各种请求，是架构的主要组成部分。 Redis客户端可能是Redis控制台或任何编程语言的Redis API。

Redis把一切存储于主内存。内存是瞬态的并且在重启完Redis或电脑后我们会丢失所有存储的数据。

## Redis 持久化

Redis持久化支持三种方式: RDB、AOF和SAVE命令。

### RDB机制

RDB按照指定的间隔把内存中所有的数据拷贝一份存储到二级存储。最近一个RDB快照之后设置的数据存在丢失的风险。

### AOF

AOF 日志会记录服务端处理的所有写入操作。这就是说所有的操作都是持久化的。采用AOF的缺点是每个操作会写入磁盘，这个操作代价是很高的，这使得AOF文件比RDB文件大得多。

### SAVE 命令

使用Redis控制台的SAVE命令，可以在任何时候让Redis服务创建一个RDB快照。

可以同时使用RDB和AOF来获得最好的持久化结果。

##  Redis的备份和恢复

Redis没有提供备份与恢复相关的机制。因此，如果有任何硬盘崩溃或任何其他类型的灾难，那么所有的数据都将丢失。需要使用第三方软件来解决备份和恢复这个问题。

如果你搭建的是多副本的环境就不需要考虑备份。

## Redis 复制

复制是一种涉及多台计算机的技术，以实现容错和数据可访问性。在复制环境中，多台计算机会互相共享数据，即使少量计算机宕机，数据仍然可用。

下图展示了Redis主从复制

![](http://qnimate.com/wp-content/uploads/2014/05/redis-replication.jpg)

主从节点是按照如图所示的方式进行配置的。

所有从节点包含与主节点相同的数据。每个主节点可能有多个从节点。当新的从节点加入集群，主节点会自动同步所有的数据到该节点。

所有的请求都会重定向到主节点，主节点会执行这些操作。
当是写请求，主节点会把新写入的数据同步给所有从节点。当是大量的排序和查询请求，主节点会把这些请求分发给其他的从节点，这样请求可以同时执行。

如果从节点失效了，集群环境依然可以工作。当从节点恢复，主节点会把更新的数据同步给从节点。

如果主节点崩溃它将丢失所有的数据，一般来说我们不会通过添加新机器让它成为主节点，而是把其中一个从节点升级成主节点。如果我们让一个新机器升级为主节点，这会让我们丢失所有的数据：
因为主节点没有数据，通过同步会使得从节点也没有数据。当然如果主节点崩溃但开启了持久化，之后只要该主节点能正常重启集群会恢复成正常运行状态。

复制让我们的集群免受磁盘和其他类型的硬件失败的影响。它帮助我们同时执行更多的排序、读请求。

## Redis 主从复制集群的持久化

上文看到持久化特性是如何避免Redis集群因意外故障而失效。如果整个集群环境的机器由于掉电而宕机，所有的数据都会丢失。发生这种情况是因为所有数据都位于内存，因此我们需要持久化。

通过采用RDB(或AOF)的方式开启主节点或任一从节点的持久化特性将数据存到二级存储。如果集群重启，请将开启了持久化特性的节点配置为主节点。

配置持久化和主从复制能够保护我们的数据足够安全，让它免受硬件意外故障的影响。

## Redis集群

集群是把数据通过分片方式分布到多台电脑上的技术。主要优势是集群通过联合多台电脑能存储更多的数据。

如果有一台64G内存的Redis服务，最多只能存储64G的数据。但如果有10台64G内存的Redis集群，则可以存储640G的数据。

![](http://qnimate.com/wp-content/uploads/2014/05/redis-cluster.jpg)

上图中我们可以看到数据分布在4个节点。每个节点都是Redis集群中的一员。

如果任一节点失效整个集群就会停止工作。

## 集群中的持久化

节点的数据存储在内存，因此需要将每个节点都开启持久化。通过使用之前提到的方法(RDB或AOF)将所有集群节点都配置为持久化存储。

## 同时配置集群和主从复制

一块硬盘崩溃，集群中的某个节点下线会让整个集群停止工作并且不会自动恢复。它的数据已经完全丢失所以没有任何办法来恢复该节点。

为避免这个问题可以定期给每个节点手动备份，但这是一项艰巨而不恰当的任务。下面通过引入主从复制来解决这个问题

![](http://qnimate.com/wp-content/uploads/2014/05/cluster-replication-redis.jpg)

把每个集群节点当做主节点。然后为每个主节点配置一个从节点。这样即使任一集群主节点失效，集群都将使用从节点来完成集群操作。

参考链接: [redis-architecture](http://qnimate.com/overview-of-redis-architecture/)