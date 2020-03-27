# Redis基本数据类型-集合(set)

## 基本用法

### 增删查

Redis Set对象用来存储无序的字符串集合。通过`SADD`可以向Set对象添加元素，通过`SISMEMBER`判断某个元素是否位于集合中，通过`SMEMBER`获取整个集合的元素

```powershell

# SADD 添加元素到集合中
127.0.0.1:6379> SADD myset 1 2 3
(integer) 3
# 判断元素是否存在于集合中
127.0.0.1:6379> SISMEMBER myset 1
(integer) 1
127.0.0.1:6379> SISMEMBER myset 4
(integer) 0
# 返回集合的所有元素(无序)
127.0.0.1:6379> SMEMBERS myset
1) "1"
2) "2"
3) "3"

# 随机获取一个成员
127.0.0.1:6379> SRANDMEMBER myset
"1"
127.0.0.1:6379> SRANDMEMBER myset
"3"
127.0.0.1:6379> SRANDMEMBER myset
"3"
127.0.0.1:6379> SRANDMEMBER myset
"2"

# 移除并返回一个成员
127.0.0.1:6379> SPOP myset
"1"
127.0.0.1:6379> SMEMBERS myset
1) "2"
2) "3"

# 删除指定的成员，返回删除成功或失败的结果
127.0.0.1:6379> SREM myset 1
(integer) 0
127.0.0.1:6379> SREM myset 2
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "3"

127.0.0.1:6379> SADD myset 1 2
(integer) 2
127.0.0.1:6379> SMEMBERS myset
1) "1"
2) "2"
3) "3"
# 扫描set对象
127.0.0.1:6379> SSCAN myset 0
1) "0"
2) 1) "1"
   2) "2"
   3) "3"

# 返回集合的长度
127.0.0.1:6379> SCARD myset
(integer) 3
```

### 交并差

Set集合也提供判断集合关系的命令，比如集合的交集、并集和差集。

```powershell
127.0.0.1:6379> SADD myset1 1 2 3
(integer) 3
127.0.0.1:6379>
127.0.0.1:6379> SADD myset2 2 3 4
(integer) 3
# 取两个集合myset1和myset2的交集
127.0.0.1:6379> SINTER myset1 myset2
1) "2"
2) "3"
# 把myset1和myset2的交集结果存到myset3中
127.0.0.1:6379> SINTERSTORE myset3 myset1 myset2
(integer) 2
127.0.0.1:6379> SMEMBERS myset3
1) "2"
2) "3"

#取两个集合myset1和myset2的并集
127.0.0.1:6379> SUNION myset1 myset2
1) "1"
2) "2"
3) "3"
4) "4"

# 把myset1和myset2的并集结果存到myset4中
127.0.0.1:6379> SUNIONSTORE myset4 myset1 myset2
(integer) 4
127.0.0.1:6379> SMEMBERS myset4
1) "1"
2) "2"
3) "3"
4) "4"

#取两个集合myset1和myset2的差集
127.0.0.1:6379> SDIFF myset1 myset2
1) "1"

#把myset1和myset2的差集结果存到myset5中
127.0.0.1:6379> SDIFFSTORE myset5 myset1 myset2
(integer) 1
127.0.0.1:6379> SMEMBERS myset5
1) "1"

#把myset5中的元素1移动到myset3中
127.0.0.1:6379> SMOVE myset5 myset3 1
(integer) 1
127.0.0.1:6379> SMEMBERS myset5
(empty list or set)
127.0.0.1:6379> SMEMBERS myset3
1) "1"
2) "2"
3) "3"
```

## 实现方式

-   整数集合 - intset
-   哈希表 - hashtable

集合对象的编码可以是intset和hashtable。如果集合对象符合以下条件，底层采用intset来存储：

1.  集合中存储的都是整数元素
2.  集合中的元素数量不超过512个

如果不能满足以上两个条件，底层会采用hashtable来存储。在Redis配置文件中可以通过`set-max-intset-entries` 来调整第二个条件的阈值。

以下示例用来演示我们在一个整数集合中添加一个字符串后发生编码转换的过程

```powershell
# 整数集合
127.0.0.1:6379> SADD numbers 1 2 3
(integer) 3
127.0.0.1:6379> OBJECT encoding numbers
"intset"
# 添加一个字符串元素
127.0.0.1:6379> SADD numbers "four"
(integer) 1
# 发生了编码转换
127.0.0.1:6379> OBJECT encoding numbers
"hashtable"
```

此外，我们构建一个包含512个元素的集合，然后再添加一个新元素，观察编码转换的过程

```powershell
# 向集合integers中添加512个元素
127.0.0.1:6379> EVAL "for i=1, 512 do redis.call('SADD', KEYS[1], i) end" 1 integers
(nil)
# 查看集合对象的编码
127.0.0.1:6379> OBJECT encoding integers
"intset"
# 向集合对象中添加一个新元素
127.0.0.1:6379> SADD integers 10086
(integer) 1
# 查看集合对象的编码，已经发生了转换
127.0.0.1:6379> OBJECT encoding integers
"hashtable"

# 移除添加的元素，编码不会转换
127.0.0.1:6379> SREM integers 10086
(integer) 1
127.0.0.1:6379> OBJECT encoding integers
"hashtable"
127.0.0.1:6379> SREM integers 512
(integer) 1
127.0.0.1:6379> OBJECT encoding integers
```