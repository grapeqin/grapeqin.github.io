# Mysql OOM案例之Mybatis-plus in传空列表参数

## 现象描述

在一个普通的工作日晚上，运维监控群突然发出一系列告警，提示生产K8S上有一个pod频繁重启，登陆到ELK上查看那段时间的FULL GC异常频繁，老年代几乎回收不了内存，如下图所示：
![](@attachment/Clipboard_2021-09-17-18-29-39.png)
初步判断是发生了内存泄漏，由于K8S自动重启pod，导致问题出现时的堆dump无法获取，经过与运维协商，开发脚本以支持在pod异常重启前将dump转储出来以方便开发定位问题。

## 分析过程

在之后的几天，同一个pod果不其然又出现了多次异常重启，这次我们拿到了对应的dump。

通过MAT将dump导入进来，结果如下图所示：
![](@attachment/Clipboard_2021-09-17-18-48-44.png)

现在可以确定正是因为这个大对象导致K8S中这个pod持续崩溃重启，接下来我们一步步分析究竟是程序的哪个环节产生了这么大的对象。

首先点击上图中的”Leak Suspects”，借助MAT工具帮我们找出产生这个大对象的程序调用堆栈，点击之后，如下图所示：
![](@attachment/Clipboard_2021-09-17-18-49-47.png)

上图中经过MAT的分析，它推测有个本地变量持有大量的对象，而且也给出了产生这些对象的调用堆栈，点击“See stacktrace”，如下图所示：

![](@attachment/Clipboard_2021-09-17-18-50-49.png)
重点请观察我在上图中标识出来的部分，到这里我们就定位出来了产生问题的代码入口为DevopsMissionQueryController.getMissionAndRepairRecord 方法，更具体的说则是调用 
DevopsMissionExtServiceImpl.getListByMissionIds方法时查出了大量对象。方法实现如下所示：
![](@attachment/Clipboard_2021-09-17-18-51-42.png)

该方法传入一个list集合通过in的方式去查询数据，如果这个方法返回大量的数据，有两种可能：
1、list参数集合传了一个很大的集合，导致SQL查询出大量的数据
2、当list集合为空时，SQL没有带任何参数，查全表查出了大量的数据
到目前为止没有更多的线索来帮我们推断到底是上面的哪一种原因导致的，当然也有可能是另外的原因导致的，我们继续排查。

接下来，我们通过“Dominator Tree”来分析大对象的引用关系，看是否能找到更多的线索，在MAT的首页，按下图红框所示打开新的视图：

![](@attachment/Clipboard_2021-09-17-18-52-55.png)
点击之后，结果展示如下：
![](@attachment/Clipboard_2021-09-17-18-53-12.png)
通过上面的Stacktrace分析，我们已经知道这是由于SQL查询大量数据导致的，所以我们重点关注”com.mysql.cj.jdbc.ResultSetImpl”代表的对象。
对象为com.mysql.cj.jdbc.result.ResultSetImpl，可以得知这是一个SQL的查询结果集，要想回答前面提出的问题，只有找到执行的原始SQL，才可以找出问题的原因所在。既然该对象是查询结果集，那无法通过查询它本身引用的对象找到对应的SQL语句，只有通过持有它的对象去查找。
在com.mysql.cj.jdbc.result.ResultSetImpl对象上点键右键，选择“List Objects -> with outgoing references”查看持有该对象的外部对象，如下图所示：
![](@attachment/Clipboard_2021-09-17-18-53-30.png)

展开该对象可以看到该对象中有一个类型为com.mysql.cj.jdbc.ClientPreparedStatement的对象引用，看到这个就说明有希望了，因为该对象中肯定包含了SQL语句和相关的参数。	继续查看ClientPreparedStatement对象引用了哪些对象，操作通过右击在com.mysql.cj.jdbc.ClientPreparedStatement对象，选择“List Objects -> with outgoing references”，展开该对象：

![](@attachment/Clipboard_2021-09-17-18-53-48.png)
再次展开红色框中的query参数：
![](@attachment/Clipboard_2021-09-17-18-53-58.png)
可以看到其中的originalSql，代表原始的SQL，把这个参数的值拿出来，SQL语句为：

```sql
SELECT  id,`missionId`,`useStocks`,`recycleStocks`,`originMissionType`,`hasTransferred`,`hotRepairFlag`,`repairExceptionFlag`,`completeExceptionFlag`,`lat`,`lng`,`updateTime`,`createTime`  
FROM devops_mission_ext_11
```

进一步展开queryBudings，可以看到该语句没有带任何查询条件：

![](@attachment/Clipboard_2021-09-17-18-54-39.png)

那么实际执行的SQL为：

```sql
SELECT  id,`missionId`,`useStocks`,`recycleStocks`,`originMissionType`,`hasTransferred`,`hotRepairFlag`,`repairExceptionFlag`,`completeExceptionFlag`,`lat`,`lng`,`updateTime`,`createTime`  
FROM devops_mission_ext_11
```
把这条SQL语句拿出来执行一下统计查询，其结果将近10万条：

![](@attachment/Clipboard_2021-09-17-18-55-07.png)

再回到下面这张图，我们看到有16个com.mysql.cj.jdbc.result.ResultSetImpl的对象，说明这张表一共分了16张表，每一张表将近10万数据，16张就是160万数据，一次性查询出来放入内存，这是导致内存溢出的根本原因。
![](@attachment/Clipboard_2021-09-17-18-55-18.png)

## 改进方案

综上，我们知道了本次内存溢出的根本原因是：在下面这段代码中，当missionIds为空列表时，mybatis-plus拼装出来的sql不带任何查询条件导致了全表查询。

![](@attachment/Clipboard_2021-09-17-18-55-47.png)

知道原因后，修复方案也就呼之欲出了。我们知道当传入的missionIds为空列表时，返回的DevopsMissionExt集合肯定是空的，根本不需要到数据库中去查询，所以只需要在构建SQL到数据库中查询之前，判断下missionIds是否为空列表，如果为空列表直接返回即可。

当然这里还存在一个需要注意的点。那就是按照写这段代码的作者原意，当使用mybatis-plus的in方法时，如果传入的参数集合为一个空列表，那么框架为我们组装的SQL语句应该为in()，但实际上我们发现，这样的SQL mysql是没法执行的，如下图所示：

![](@attachment/Clipboard_2021-09-17-18-55-57.png)

于是，我们查看了当前使用的mybatis-plus的版本，如下所示：

```xml
 <dependency>
      <groupId>com.baomidou</groupId>
      <artifactId>mybatis-plus-boot-starter</artifactId>
      <version>3.0.7.1</version>
</dependency>
```

以及mybatis-plus的feature-list，如下图所示：
![](@attachment/Clipboard_2021-09-18-14-00-34 (2).png)

看起来官方也是发现了这种问题，在框架内部为了避免空列表导致的语法错误，但却改变了使用方本来的意图，一旦查询的是一张大表，极大可能会引起业务系统OOM，所以在3.1.0版本官方予以修复，业务方一旦传入空列表，直接报错，让业务方自行提前对参数进行合法性判断。