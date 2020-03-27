# 基本数据类型 - 有序集合(SortedSet)

有序集合是集合和哈希数据类型的结合。跟集合类似，有序集合也是有唯一的，非重复的字符串元素组成，从整个概念上来说有序集合也是一个集合。

集合中的元素都是无序的，有序集合中的每个元素都关联了一个小数的分值，称作评分。这样有序集合中的元素就是有序的。它的排序遵循如下规则：

-   如果元素A和元素B有不同的评分，如果A.score > B.score 那么A > B
-   如果元素A和元素B的评分相同，如果元素A在字母顺序上比元素B大，则A > B。元素A和元素B不可能相同是因为有序集合保证了元素的唯一性

## 基本用法

### 添加&修改

我们举个例子，在有序集合中添加一批名字，用他们的出生年份作为分值

```powershell
# 使用zadd向有序集合hackers中添加元素
127.0.0.1:6379> zadd hackers 1940 "Alan Kay"
(integer) 1
127.0.0.1:6379> zadd hackers 1957 "Sophie Wilson"
(integer) 1
127.0.0.1:6379> zadd hackers 1953 "Richard Stallman"
(integer) 1
127.0.0.1:6379> zadd hackers 1949 "Anita Borg"
(integer) 1
127.0.0.1:6379> zadd hackers 1965 "Yukihiro Matsumoto"
(integer) 1
127.0.0.1:6379> zadd hackers 1914 "Hedy Lamarr"
(integer) 1
127.0.0.1:6379> zadd hackers 1916 "Claude Shannon"
(integer) 1
127.0.0.1:6379> zadd hackers 1969 "Linus Torvalds"
(integer) 1
127.0.0.1:6379> zadd hackers 1912 "Alan Turing"
(integer) 1

# 添加重复元素会返回失败，但是其评分发生了修改
127.0.0.1:6379> zadd hackers 1922 "Alan Turing"
(integer) 0

# 修改有序集合中元素的评分
127.0.0.1:6379> ZINCRBY hackers 10 "Alan Turing"
"1932"
```

### 查询

```powershell
# 获取有序集合的元素数量
127.0.0.1:6379> ZCARD hackers
(integer) 9

# 通过评分上下界来统计元素的数量
127.0.0.1:6379> ZCOUNT hackers 0 1900
(integer) 0
127.0.0.1:6379> ZCOUNT hackers 1900 1920
(integer) 2

# 如果有序集合元素的评分都相同，可以通过元素的字母表顺序来排序
127.0.0.1:6379> ZADD myzset 0 a 0 b 0 c 0 d 0 e
(integer) 5
127.0.0.1:6379> ZADD myzset 0 f 0 g
(integer) 2
127.0.0.1:6379> ZLEXCOUNT myzset - +
(integer) 7
127.0.0.1:6379> ZLEXCOUNT myzset [b [f
(integer) 5

# ZRANGE范围查询
127.0.0.1:6379> ZADD myzset:range 1 "one"
(integer) 1
127.0.0.1:6379> ZADD myzset:range 2 "two"
(integer) 1
127.0.0.1:6379> ZADD myzset:range 3 "three"
(integer) 1
# start和end 为闭区间 start和end为元素的下标，从0开始，也可以为负数，表示从有序集合的末尾开始计算下标
127.0.0.1:6379> ZRANGE myzset:range 0 -1
1) "one"
2) "two"
3) "three"
# start和end 如果越界，不会报错
127.0.0.1:6379> ZRANGE myzset:range 2 3
1) "three"
127.0.0.1:6379> ZRANGE myzset:range -2 -1
1) "two"
2) "three"

#结果中会同时返回分值
127.0.0.1:6379> ZRANGE myzset:range -2 -1 withscores
1) "two"
2) "2"
3) "three"
4) "3"

# ZREVRANGE 与 ZRANGE 类似，只是返回结果顺序不同
127.0.0.1:6379> ZREVRANGE myzset:range 0 -1
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> ZREVRANGE myzset:range 2 3
1) "one"
127.0.0.1:6379>
127.0.0.1:6379> ZREVRANGE myzset:range -2 -1
1) "two"
2) "one"

# ZRANGEBYLEX根据字母顺序来查询范围区间
127.0.0.1:6379> zadd myzset:range 0 a1 0 b2 0 c3 1 a2 1 d4 1 f5
(integer) 6
127.0.0.1:6379> ZRANGE myzset:range 0 -1
1) "a1"
2) "b2"
3) "c3"
4) "a2"
5) "d4"
6) "f5"
# 按照字母顺序的范围来返回元素 
# [ 为闭区间 ( 为开区间
# - 为负无穷大 + 为正无穷大
127.0.0.1:6379> ZRANGEBYLEX myzset:range [a [a2
1) "a1"
127.0.0.1:6379> ZRANGEBYLEX myzset:range [a [c3
1) "a1"
2) "b2"
3) "c3"
4) "a2"
127.0.0.1:6379> ZRANGEBYLEX myzset:range - +
1) "a1"
2) "b2"
3) "c3"
4) "a2"
5) "d4"
6) "f5"

# ZREVRANGEBYLEX 与ZRANGEBYLEX 类似，不同的是参数顺序和结果返回的顺序
127.0.0.1:6379> ZREVRANGEBYLEX myzset:range + -
1) "f5"
2) "d4"
3) "a2"
4) "c3"
5) "b2"
6) "a1"
127.0.0.1:6379> ZREVRANGEBYLEX myzset:range [c3 [a
1) "a2"
2) "c3"
3) "b2"
4) "a1"

# ZRANGEBYSCORE 通过评分来获取元素 
# +INF -INF 分别代表正无穷 和 负无穷
127.0.0.1:6379> ZADD myzset:range 1 "one" 2 "two" 3 "three" 1 "first"
(integer) 4
127.0.0.1:6379> ZRANGEBYSCORE myzset:range -INF +INF
1) "first"
2) "one"
3) "two"
4) "three"
127.0.0.1:6379> ZRANGEBYSCORE myzset:range 1 2
1) "first"
2) "one"
3) "two"
127.0.0.1:6379> ZRANGEBYSCORE myzset:range (1 2
1) "two"
127.0.0.1:6379> ZRANGEBYSCORE myzset:range (1 (2
(empty list or set)
# ZREVRANGEBYSCORE 与 ZRANGEBYSCORE 类似，区别是参数顺序和返回结合元素的顺序不同
127.0.0.1:6379> ZREVRANGEBYSCORE myzset:range +INF -INF
1) "three"
2) "two"
3) "one"
4) "first"
127.0.0.1:6379>
127.0.0.1:6379> ZREVRANGEBYSCORE myzset:range 2 1
1) "two"
2) "one"
3) "first"
127.0.0.1:6379> ZREVRANGEBYSCORE myzset:range (2 1
1) "one"
2) "first"
127.0.0.1:6379> ZREVRANGEBYSCORE myzset:range (2 (1
(empty list or set)

# ZRANK 获取某个元素在有序集合中的排名 ，按照从小到大的顺序，0表示最小的元素
# 如果元素不存在，返回nil
127.0.0.1:6379> ZRANK myzset:range first
(integer) 0
127.0.0.1:6379> ZRANK myzset:range one
(integer) 1
127.0.0.1:6379> ZRANK myzset:range four
(nil)
# ZREVRANK 与 ZRANK类似，它是按照从大到小的顺序，0表示最大的元素
127.0.0.1:6379> ZREVRANK myzset:range first
(integer) 3
127.0.0.1:6379> ZREVRANK myzset:range three
(integer) 0
127.0.0.1:6379> ZREVRANK myzset:range four
(nil)

# ZSCORE 获取某个元素在有序集合中的评分
127.0.0.1:6379> ZSCORE myzset:range one
"1"
127.0.0.1:6379> ZSCORE myzset:range four
(nil)

# ZSCAN 扫描有序集合,结果中依次包含元素和评分
127.0.0.1:6379> ZSCAN myzset:range 0
1) "0"
2) 1) "first"
   2) "1"
   3) "one"
   4) "1"
   5) "two"
   6) "2"
   7) "three"
   8) "3"
```

### 删除

```powershell
# ZREM 删除有序集合中的某个元素 返回删除的个数
127.0.0.1:6379> ZADD myzset 1 "one" 2 "two" 3 "three"
(integer) 3
127.0.0.1:6379> ZREM myzset two
(integer) 1
127.0.0.1:6379> ZRANGE myzset 0 -1 withscores
1) "one"
2) "1"
3) "three"
4) "3"
127.0.0.1:6379> del myzset
(integer) 1

# ZREMRANGEBYLEX 删除以字母顺序为标准的范围内的元素
127.0.0.1:6379> ZADD myzset 0 aaaa 0 b 0 c 0 d 0 e
(integer) 5
127.0.0.1:6379> ZADD myzset 0 foo 0 zap 0 zip 0 ALPHA 0 alpha
(integer) 5
127.0.0.1:6379> ZRANGE myzset 0 -1
 1) "ALPHA"
 2) "aaaa"
 3) "alpha"
 4) "b"
 5) "c"
 6) "d"
 7) "e"
 8) "foo"
 9) "zap"
10) "zip"
127.0.0.1:6379> ZREMRANGEBYLEX myzset [alpha [omega
(integer) 6
127.0.0.1:6379> ZRANGE myzset 0 -1
1) "ALPHA"
2) "aaaa"
3) "zap"
4) "zip"
127.0.0.1:6379> DEL myzset
(integer) 1

# ZREMRANGEBYRANK、ZREMRANGEBYSCORE与ZREMRANGEBYLEX类似，只是它们分别是根据元素的排名和评分来作为过滤数据的标准

# ZPOPMAX、ZPOPMIN 分别移除并返回最大和最小的元素
127.0.0.1:6379> ZADD myzset 1 "one" 2 "two" 3 "three"
(integer) 3
127.0.0.1:6379> ZPOPMAX myzset
1) "three"
2) "3"
127.0.0.1:6379> ZPOPMIN myzset
1) "one"
2) "1"
127.0.0.1:6379> ZRANGE myzset 0 -1
1) "two"
127.0.0.1:6379> DEL myzset
(integer) 1

#BZPOPMAX和BZPOPMIN分别是ZPOPMAX和ZPOPMIN的阻塞模式，如果当前集合为空时，会一直阻塞直到集合中有元素为止，或者超过过期时间
```

### 集合并集&交集

```powershell
# ZINTERSTORE 把多个有序集合中的元素取交集，然后存入目标集合，元素的评分值默认会累加：依次用交集的结果  元素，在每个集合中的评分分别乘以weights值相加，通过AGGREGATE可以设置为取MAX或MIN
127.0.0.1:6379> ZADD zset1 1 "one" 2 "two"
(integer) 2
127.0.0.1:6379> ZADD zset2 1 "one" 2 "two" 3 "three"
(integer) 3
127.0.0.1:6379> ZINTERSTORE out 2 zset1 zset2 WEIGHTS 2 3
(integer) 2
127.0.0.1:6379> ZRANGE out 0 -1 WITHSCORES
1) "one"
2) "5"
3) "two"
4) "10"
# ZUNIONSTORE 取多个有序集合的并集，然后存入目标集合，元素的评分值默认累加，通过AGGREGATE可以设置取最大值MAX或最小值MIN
127.0.0.1:6379> ZUNIONSTORE out 2 zset1 zset2 WEIGHTS 2 3
(integer) 3
127.0.0.1:6379> ZRANGE out 0 -1 WITHSCORES
1) "one"
2) "5"
3) "three"
4) "9"
5) "two"
6) "10"
```

## 实现方式

有序集合对象默认采用ziplist作为底层编码实现。每个集合元素使用临近的两个压缩列表结点来保存，前面这个结点保存元素值，后面这个结点保存评分。压缩表的元素按照评分从小到大排序。

除此之外，有序集合对象底层还用跳跃表skiplist+hashtable的编码形式来实现。跳跃表从小到大保存集合元素，跳跃表结点的object属性保存元素成员，score属性保存评分。hashtable则创建了从集合元素到评分的映射关系，通过该结构，指定集合元素获取它的评分时间复杂度变为O(1)。

这里跳跃表和字典结构保存的Redis对象，是共享的。因此不会造成空间浪费。

-   压缩表 - ziplist
-   跳跃表 - skiplist

有序集合对象满足以下两个条件时，使用ziplist来实现：

1. 有序集合保存的元素数量小于 `128` 个

2. 有序集合保存的所有元素成员的长度都小于 `64` 字节

不满足以上两个条件的有序集合都采用skiplist作为底层编码。这两个值可以通过Redis配置文件中的`zset-max-ziplist-entries`和`zset-max-ziplist-value` 来修改。

以下代码演示了有序集合对象包含了过多的元素而发生编码转换：

```powershell
127.0.0.1:6379> EVAL "for i=1, 128 do redis.call('ZADD', KEYS[1], i, i) end" 1 numbers
(nil)
127.0.0.1:6379> ZCARD numbers
(integer) 128
127.0.0.1:6379> OBJECT ENCODING numbers
"ziplist"
127.0.0.1:6379> ZADD numbers 3.14 pi
(integer) 1
127.0.0.1:6379> ZCARD numbers
(integer) 129
127.0.0.1:6379> OBJECT ENCODING numbers
"skiplist"
```

以下代码演示了有序集合对象的元素过长而发生编码转换

```powershell
127.0.0.1:6379> ZADD blah 1.0 www
(integer) 1
127.0.0.1:6379> OBJECT ENCODING blah
"ziplist"
127.0.0.1:6379> ZADD blah 2.0 oooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo
(integer) 1
127.0.0.1:6379> OBJECT ENCODING blah
"skiplist"
```

