# Redis基本数据类型 - 列表(list)

## 日常用法

### 初识列表

LPUSH命令添加元素到列表的开头，RPUSH命令添加元素到列表的末尾。LRANGE命令提取列表中指定范围内的元素。

```powershell
> RPUSH mylist A
(integer) 1
> RPUSH mylist B
(integer) 2
> LPUSH mylist "first"
(integer) 3
> LRANGE mylist 0 -1
1) "first"
2) "A"
3) "B"
```

LRANGE命令有2个索引，表示返回指定范围的第一个和最后一个元素。这两个索引都可以是负数，告诉Redis从列表的末尾开始计数；-1表示列表倒数第一个元素，-2表示倒数第二个元素。

RPUSH和LPUSH都是可变参数命令，意味着在一次调用中可以push多个元素：

```powershell
> RPUSH mylist 1 2 3 4 5 "foo bar"
(integer) 9
> LRANGE mylist 0 -1
1) "first"
2) "A"
3) "B"
4) "1"
5) "2"
6) "3"
7) "4"
8) "5"
9) "foo bar"
```

Redis还提供了一项重要的操作 - pop元素。弹出元素的操作包含两个步骤，获取列表中的元素同时将该元素从列表中移除。我们可以从列表的开头或末尾来弹出元素，这与push命令类似：

```powershell
> RPUSH mylist a b c
(integer) 3
> RPOP mylist
"c"
> RPOP mylist
"b"
> RPOP mylist
"a"
```

上面的例子我们往列表中添加3个元素，然后依次弹出这3个元素，在这一系列命令执行之后，列表变为空列表，没有元素可以弹出了。如果我们还想弹出元素，Redis返回NULL

```powershell
> RPOP mylist
(nil)
```



## 运用场景

-   社交网络应用中记录用户最新发布的动态
-   进程间通信，生产者消费者模式的运用 - 生产者生产消息push到列表中，消费者pop这些消息并完成处理。Redis为列表数据类型提供了一些特殊命令来确保可靠和高效的在这种场景下使用列表

### 有界列表

在大部分场景下，我们使用列表只会存储最新的几个元素，无论它们是网络更新、日志还是其他需要记录的数据。

Redis允许我们把列表(list)当做有界集合来使用，它只记录最新的N个元素，使用`LTRIM`命令丢弃掉其他较老的元素。

`LTRIM`命令与`LRANGE`都跟列表范围有关，范围为闭区间(即包含前后边界值)。不同的是`LRANGE`是显示指定范围内的元素，而`LTRIM`是为将该范围的值保留并作为一个新列表返回，范围之外的元素都会被移除。

示例如下：

```powershell
> RPUSH mylist 1 2 3 4 5
(integer) 5
> LTRIM mylist 0 2
OK
> LRANGE mylist 0 -1
1) "1"
2) "2"
3) "3"
```

上面的命令告诉Redis只保留[0,2]这个区间的元素，其他的全部丢弃。这种操作提供了一种非常有用的模式：将往列表中push元素的操作和trim操作结合起来就能提供往列表中添加新元素同时把超过限制的老元素移除的解决方案。

```powershell
LPUSH mylist <some element>
LTRIM mylist 0 999
```

上面的命令组合实现往列表中添加新元素同时只保留最新的1000个元素的功能。再通过`LRANGE`就能获取TOP 1000个元素而不需要记录老元素。

### 阻塞的列表操作

列表还提供了特性：阻塞操作，非常适合用于实现队列。

想象这样一种场景：我们期望在一个进程中往列表中push元素，在另一个进程中运用这些元素做一些工作。这是典型的生产者/消费者模式，我们可以通过下面这些步骤来简单的模拟它的实现：

-   生产者调用`LPUSH`将元素push到列表中
-   消费者调用`RPOP`来获取列表中的元素并处理

当列表为空时，没有元素需要处理，这时`RPOP`会返回NULL。在这种情况下，消费者必须强制等待一段时间然后使用`RPOP`尝试获取元素。这种处理方式称为`polling`，在这个场景下它并不是一个好的解决方案，是因为它有以下问题：

1.  让Redis和客户端处理过多无效的命令，浪费服务器资源
2.  元素的处理操作存在时延，因为当返回NULL时，需要等待一段时间。为了减少这个时延，需要将等待的时间缩短，这样又会导致问题1(会导致Redis和客户端处理很多无效的命令)

为解决这些问题，Redis为`LPOP`和`RPOP`提供了当列表为空时对应的阻塞操作命令`BLPOP`和`BRPOP`。它们返回的条件有两种：要么是有元素添加到列表中，要么是达到用户设置的超时时间。

下面是我们可以在消费线程中使用的`BRPOP`命令示例：

```powershell
> BRPOP tasks 5
1) "tasks"
2) "do_something"
```

这条命令的意思是让tasks列表等待元素，如果5S后列表中还没有元素push就返回。

我们可以将超时时间设置为0，这意味着命令会一直阻塞直到列表中push新的元素，为了同时等待多个列表，你还可以在阻塞命令后指定多个列表，当第一个列表push了新元素我们能得到通知。

在使用`BRPOP`命令过程中有几件事需要注意：

1.  客户端以有序的方式提供服务：当一个客户端阻塞等待列表时，其他客户端往列表中push元素后这个客户端就能得到返回值
2.  相比`RPOP`来说返回值不同：它返回包含两个元素的数组，因为`BRPOP`和`BLPOP`可以同时阻塞等待多个列表，所以它的返回结果需要包含key的名称
3.  如果超时时间到了，返回NULL

示例如下，开启两个终端(模拟)

```powershell
> BRPOP mylist1 mylist2 0
1) "mylist2"
2) "1"
(339.67s)
> BRPOP mylist1 mylist2 0
1) "mylist1"
2) "1"
(7.19s)
> BRPOP mylist1 mylist2 5
(nil)
(5.00s)
```

```powershell
> LPUSH mylist2 1
(integer) 1
> LPUSH mylist1 1
(integer) 1
```

有关列表和阻塞操作还有一些事情我们需要知道。我们建议阅读如下内容：

-   使用[RPOPLPUSH](https://redis.io/commands/rpoplpush)能够构建可靠队列和滚动队列

    -   可靠队列 - 列表提供简单队列的方式是让生产者把消息push到列表中，然后消费者使用RPOP命令获取消息。这种使用方式可能导致消息丢失，试想如果出现网络问题或者消费者从队列中获取了消息但还未来得及处理就崩溃，这个消息就丢失了。但RPOPLPUSH能解决这个问题：消费者在获取消息的同时把消息push到一个正在处理的列表中，当消息处理完成，可以使用LREM命令从正在处理的列表中将消息移除。另外，启动一个线程来扫描正在处理的列表，来检测正在处理的列表中消息存活的时长，根据业务需要把超时处理的这些消息重新push到源队列中。

    -   滚动队列 - 在RPOPLPUSH命令中使用同一个列表作为源列表和目标列表，客户端可以依次访问列表中的所有元素，可以避免LRANGE命令这样的操作一次性把列表的元素从服务端传回客户端。这种场景也适用于下面的情况

        -   有多个客户端来滚动列表：它们获取不同的元素，直到所有的元素都访问一遍，然后重新开始访问
        -   当有其他客户端还在往列表的结尾push新元素时

        ```powershell
        # 构建源列表mylist
        > RPUSH mylist "one" "two" "three"
        (integer) 3
        # 在源列表mylist和目标列表myotherlist上执行RPOPLPUSH
        > RPOPLPUSH mylist myotherlist
        "three"
        # 查看源列表
        > LRANGE mylist 0 -1
        1) "one"
        2) "two"
        # 查看目标列表
        > LRANGE myotherlist 0 -1
        1) "three"
        
        # 不存在的源列表mynelist上执行命令返回NULL
        > RPOPLPUSH mynelist mydestlist
        (nil)
        ```


​    

-   与该命令功能类似的还有一个[BRPOPLPUSH](https://redis.io/commands/brpoplpush)命令

### 自动创建和移除键值

目前为止我们没有在push元素之前先创建一个空列表，或者当列表为空时手动移除该列表。Redis为我们提供了一项能力：当列表为空时它会自动删除键值，当列表不存在时Redis会自动创建一个代表该列表的键值然后向该列表添加元素。

这项能力其实不仅仅是是列表独有的，其他Redis提供的包含多个元素的数据类型都有这项能力 - Streams、Sets、Sorted Set和Hashes都有类似的特性。

总体来说，我们可以总结如下三个规则

1.  当我们向聚合数据类型集合添加元素时，如果目标key不存在，Redis会先创建一个空的聚合类型集合然后再向其中添加元素
2.  当我们从聚合数据类型集合移除元素时，如果值为空，键值会自动移除。Stream数据类型除外。
3.  在空键值上调用类似`LLEN`这样的只读命令，或者移除元素的写入命令，产生的效果与该键值持有一个空聚合数据类型集合的执行效果是一样的

规则1的示例：

```powershell
> del mylist
(integer) 1
> LPUSH mylist 1 2 3
(integer) 3
```

当键值存在时，我们不能在类型上执行不匹配的操作

```powershell
> set foo bar
OK
> LPUSH foo 1 2 3
(error) WRONGTYPE Operation against a key holding the wrong kind of value
> type foo
string
```

规则2的示例：

```powershell
> LPUSH mylist 1 2 3
(integer) 3
> EXISTS mylist
(integer) 1
> LPOP mylist
"3"
> LPOP mylist
"2"
> LPOP mylist
"1"
> EXISTS mylist
(integer) 0
```

当所有的元素被弹出后键值不再存在。

规则3的示例：

```powershell
> DEL mylist
(integer) 0
> LLEN mylist
(integer) 0
> LPOP mylist
(nil)
```

## 实现方式

### 3.0版本

-   压缩列表 - 采用数组的形式来存储
-   双端链表 - 采用链表的形式来存储

默认情况下为了节省内存空间会使用压缩列表来存储，我们可以配置列表占用的空间或元素个数来决定何时从压缩列表转换成双端链表的存储方式。

`list-max-ziplist-size`该配置可以配置成-5 ~ -1或者正数

-   负数
    -   -5: max size: 64 Kb <-- not recommended for normal workloads
    -   -4: max size: 32 Kb  <-- not recommended
    -   -3: max size: 16 Kb  <-- probably not recommended
    -   -2: max size: 8 Kb   <-- good
    -   -1: max size: 4 Kb   <-- good
-   正数
    -   表示列表中存储这么多个数的元素后就会改变

请参考[redis.conf](https://github.com/antirez/redis/blob/unstable/redis.conf) 中的`list-max-ziplist-size`配置项

### 5.0版本

-   快速列表 - quicklist

    quicklist是一个双向链表，每个链表的Node结点又指向了一个Ziplist结构，这样带来的好处是能够避免Ziplist在插入删除操作时引起的全局连锁更新问题。

