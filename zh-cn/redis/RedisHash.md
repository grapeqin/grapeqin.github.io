# Redis基本数据类型-哈希(Hash)

## 基本用法

### 增加

Redis Hash与我们期望的hash结构是一样的，存储键值对：

```powershell
#HMSET 一次可以设置多个字段
127.0.0.1:6379> HMSET user:1000 username antirez birthyear 1977 verified 1
OK
127.0.0.1:6379> HGET user:1000 username
"antirez"
127.0.0.1:6379> HGETALL user:1000
1) "username"
2) "antirez"
3) "birthyear"
4) "1977"
5) "verified"
6) "1"

# 一次设置一个字段
127.0.0.1:6379> HSET user:1000 gender m
(integer) 1

# 设置字段，只有键值不存在时才能设置成功
127.0.0.1:6379> HSETNX user:1000 gender f
(integer) 0

127.0.0.1:6379> HGETALL user:1000
1) "username"
2) "antirez"
3) "birthyear"
4) "1997"
5) "verified"
6) "1"
7) "gender"
8) "m"
```

`HMSET` 为hash设置多个字段，`HGET`获取单个字段，`HMGET`与`HGET`类似，都是获取字段，但它返回的是数组结构。

```powershell
127.0.0.1:6379> HMGET user:1000 username birthyear no-such-field
1) "antirez"
2) "1977"
3) (nil)
```

### 删除

```powershell
# 删除单个字段
127.0.0.1:6379> HDEL user:1000 gender
(integer) 1
```

### 修改

Redis也提供了一些能在单个字段上执行的命令，例如：

```powershell
127.0.0.1:6379> HINCRBY user:1000 birthyear 10
(integer) 1987
127.0.0.1:6379> HINCRBY user:1000 birthyear 10
(integer) 1997
```

### 查询

Redis还提供了判断字段是否存在，获取所有字段，获取所有字段值，以及获取集合长度的命令：

```powershell
# 判断字段是否存在
127.0.0.1:6379> HEXISTS user:1000 username
(integer) 1
127.0.0.1:6379> HEXISTS user:1000 no-such-field
(integer) 0

# 获取hash结构的长度
127.0.0.1:6379> HLEN user:1000
(integer) 3

# 获取字段值的长度
127.0.0.1:6379> HSTRLEN user:1000 username
(integer) 7

# 获取所有的字段
127.0.0.1:6379> HKEYS user:1000
1) "username"
2) "birthyear"
3) "verified"

# 获取所有的字段值
127.0.0.1:6379> HVALS user:1000
1) "antirez"
2) "1997"
3) "1"

# 迭代hash
127.0.0.1:6379> HSCAN user:1000 0
1) "0"
2) 1) "username"
   2) "antirez"
   3) "birthyear"
   4) "1997"
   5) "verified"
   6) "1"
```

## 运用场景

Hash结构可以很方便的来描述对象，我们放入hash里面的字段理论上没有任何限制(只要内存足够)，因此在我们的应用中可以有许多地方来使用hash。

## 实现方式

-   压缩列表 - ziplist
-   哈希表 - hashtable

当哈希对象同时满足以下两个条件时，哈希对象使用 ziplist：

-   哈希对象保存的所有键值对的键和值的长度都小于64字节
-   哈希对象保存的键值对的数量小于512个

不能满足这两个条件的哈希对象都要使用 hashtable。这两个条件可以通过配置文件中的`hash-max-ziplist-entries` 和`hash-max-ziplist-value` 这两个参数进行修改。

以下展示了哈希对象因为键值对的长度太长导致的编码转换：

```powershell
127.0.0.1:6379> HSET book name "Mastering Docker in 30 days"
(integer) 1
127.0.0.1:6379> OBJECT encoding book
"ziplist"

# 向哈希对象中添加一个新的键值对，键的长度大于64字节，发生编码转换
127.0.0.1:6379> HSET book long_long_long_long_long_long_long_long_long_long_long_description "content"
(integer) 1
127.0.0.1:6379> OBJECT encoding book
"hashtable"

127.0.0.1:6379> HSET blah greeting "hello world"
(integer) 1
127.0.0.1:6379> OBJECT Encoding blah
"ziplist"
# 向哈希对象中添加新的键值对，值的长度大于64字节，发生编码转换
127.0.0.1:6379> HSET blah story "many string ... many string ... many string ... many string ... many"
(integer) 1
127.0.0.1:6379> OBJECT encoding blah
"hashtable"
```

如下代码展示哈希对象包含了过多的键值对导致的编码转换：

```powershell
# 向哈希对象中添加512个对象
127.0.0.1:6379> EVAL "for i=1, 512 do redis.call('HSET', KEYS[1], i, i) end" 1 "numbers"
(nil)
127.0.0.1:6379> HLEN numbers
(integer) 512
127.0.0.1:6379> OBJECT ENCODING numbers
"ziplist"
# 向哈希对象中添加一个新键值对
127.0.0.1:6379> HMSET numbers "key" "value"
OK
127.0.0.1:6379> HLEN numbers
(integer) 513
# 超过键值对长度的阈值，发生编码转换
127.0.0.1:6379> OBJECT ENCODING numbers
"hashtable"
```

