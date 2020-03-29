

# Redis基本数据类型 - 字符串(String)

## 基本用法

字符串是Redis中最常见的一种基本数据类型。它既可以用作Redis的键类型，也可以用作Redis的值类型。对用字符串作为值类型的操作分类如下：

| 操作类型  | 操作命令    | 命令说明                                             |
| --------- | ----------- | ---------------------------------------------------- |
| 设置&修改 | SET         | 为某个key设置值                                      |
|           | SETNX       | 仅当key不存在时为key设置值                           |
|           | SETEX       | 为指定的key设置值，同时以秒为单位设置key的过期时间   |
|           | PSETEX      | 为指定的key设置值，同时以毫秒为单位设置key的过期时间 |
|           | MSET        | 一次设置多个key的值                                  |
|           | MSETNX      | 只有多个key都不存在时，一次性为多个key设置值         |
|           | GETSET      | 为指定的key设置值同时返回该key的旧值                 |
|           | SETRANGE    | 重写key上指定范围的子串                              |
|           | INCR        | 递增存储到该key上的整数值                            |
|           | INCRBY      | 以指定的偏移量递增存储到该key上的整数值              |
|           | INCRBYFLOAT | 以指定的小数偏移量递增存储到该key上的值              |
|           | DECR        | 递减存储到该key上的整数值                            |
|           | DECRBY      | 以指定的偏移量递减存储到该key上的整数值              |
| 查询      | GET         | 返回存储到该key上的值                                |
|           | GETRANGE    | 返回存储到该key上字符串的子串                        |
|           | MGET        | 一次获取多个key的值                                  |
|           | STRLEN      | 返回存储到某key上字符串值的长度                      |
| 位图      | SETBIT      | 在指定的位置设置值(0或1)                             |
|           | GETBIT      | 返回指定位置的值(0或1)                               |
|           | BITCOUNT    | 计算字符串中设置为1的位的个数                        |
|           | BITOP       | 在多个key上执行按位的运算                            |
|           | BITPOS      | 找到第一个设置为0或者1的位置                         |
|           | BITFIELD    | 在指定位置的二进制位上执行操作                       |

### 设置&修改

#### SET

SET命令设置单个字符串值，如果该key已经存在，无论它之前是什么类型，直接覆盖它。一旦覆盖成功，该key之前的TTL属性都会丢弃。

##### 选项

SET命令提供了多个选项

-   EX 秒数 - 指定过期时间，以秒为单位
-   PX 毫秒 - 指定过期时间，以毫秒为单位
-   NX - 仅仅只有当**该key不存在时**才会设置值
-   XX - 仅仅只有当**key存在时**才会设置值
-   KEEPTTL - 获取该key的TTL

由于SET命令的选项可以替代SETNX、SETEX、PSETEX命令的功能，未来的版本可能会移除这三个命令。

##### 返回值

如果命令执行成功直接返回**OK**字符串。

```powershell
# 设置单个值
127.0.0.1:6379> set mykey myvalue
OK
127.0.0.1:6379> get mykey
"myvalue"
# 设置otherkey 60秒后过期
127.0.0.1:6379> set otherkey value ex 60
OK
# 当mykey不存在时设置5秒后过期
127.0.0.1:6379> SET mykey myval EX 5 NX
OK
# 当不存在的条件不成立时直接返回NIL
127.0.0.1:6379> SET mykey myval EX 5 NX
(nil)
```

#### SETNX

当key不存在时为key设置字符串值，它的功能与SET是一样的。当该key已存在，什么也不会执行。SETNX是"**SET** if **N**ot e**X**ists"的简称。

##### 返回值

返回值为一个数字

-   如果设置成功返回1
-   如果设置失败返回0

```powershell
# 仅当key不存在时设置值，mykey已存在则设置失败
127.0.0.1:6379> SETNX mykey myvalue
(integer) 0
127.0.0.1:6379> DEL mykey
(integer) 1
# mykey不存在时设置成功
127.0.0.1:6379> SETNX mykey myvalue
(integer) 1
```

#### SETEX

该命令为key设置字符串值，同时设置该key在指定的秒数后过期。该命令与下面的命令是等价的

```powershell
SET mykey value
EXPIRE mykey seconds
```

`SETEX` 命令是原子性的，它们与用 MULTI/EXEC块来执行前面两个命令是等价的。对于把Redis作为缓存是常见的一种用法。当传入的秒数不合法会直接返回错误

##### 返回值

返回字符串

```powershell
# 为mykey设置"hello"字符串,10秒后过期
127.0.0.1:6379> SETEX mykey 10 "hello"
OK
127.0.0.1:6379> TTL mykey
(integer) 7
127.0.0.1:6379> GET mykey
"hello"
# 过期后查询不到mykey的值
127.0.0.1:6379> GET mykey
(nil)
# 设置的秒数必须为正数,否则直接报错
127.0.0.1:6379> SETEX mykey -10 "hello"
(error) ERR invalid expire time in setex
127.0.0.1:6379>
127.0.0.1:6379> SETEX mykey 0 "hello"
(error) ERR invalid expire time in setex
```

#### PSETEX

与**SETEX**类似，只是过期时间传入的是毫秒。

#### MSET

与SET功能类似，不同的是可以同时为多个key设置字符串值。MSET命令是原子操作，所有的key会一次性设置完，这就导致客户端无法看到有哪些key的字符串值发生了改变，哪些没有发生改变。

##### 返回值

总是返回**OK** 因为它不会失败。

```powershell
# 一次性设置好key1和key2
127.0.0.1:6379> MSET key1 "hello" key2 "world"
OK
127.0.0.1:6379> get key1
"hello"
127.0.0.1:6379> get key2
"world"
```

#### MSETNX

仅当指定的所有key都不存在，才会执行设置操作，只要有一个key已经存在，都不会执行任何操作。

因为它的这个语义，我们可以用它来为一个独立对象的不同字段设置不同的键值，这样就能确保要么所有的字段都被设置，要么一个都不需要设置。

MSETNX命令是原子性的，所有的keys会一次性设置完成。同样对客户端来说没有办法看到哪些key被更新了，哪些key没有变更。

##### 返回值

返回一个整数

-   如果所有的key被设置会返回1
-   如果没有1个key被设置会返回0(意味着至少有一个key已经存在)

```powershell
# key1和key2都不存在，所以可以同时设置成功，返回1
127.0.0.1:6379> MSETNX key1 "hello" key2 "there"
(integer) 1
# key2已经存在，所以无法设置成功，返回0
127.0.0.1:6379> MSETNX key2 "new" key3 "world"
(integer) 0
127.0.0.1:6379> MGET key1 key2 key3
1) "hello"
2) "there"
3) (nil)
```

#### GETSET

自动为key设置字符串值，同时返回该key之前存储的字符串值。如果该key存在但存储的值类型不是字符串，会报错。

##### 返回值

返回存储在该key的字符串值，如果该key不存在，返回NIL

```powershell
# 为mykey设置一个值类型为list的值列表
127.0.0.1:6379> LPUSH mykey 1 2 3
(integer) 3
# 因为mykey的类型不是字符串，所以执行命令报错
127.0.0.1:6379> GETSET mykey 1
(error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> DEL mykey
(integer) 1
127.0.0.1:6379> SET mykey "Hello"
OK
127.0.0.1:6379> GETSET mykey "World"
"Hello"
127.0.0.1:6379> GET mykey
"World"
```

#### SETRANGE

从指定的offset开始，用新字符串覆写存储在该key上的字符串值。如果提供的offset比原字符串的长度大，会在原值后面用0字节来填充，然后再追加新字符串。不存在的key会被认为是一个空字符串，该命令会一直填充0字节让它足够大来满足在指定的位置设置新字符串。因为Redis字符串的最大长度为512M，所以最大的offset是  2 ^29^ - 1(536870911)  。如果存储的字符串超过了这个长度，建议使用多个key来存储。

**注意：** 如果key不存在，或者key上保存着一个较小的字符串，在设置最后一个字节时，由于Redis需要申请临时内存，所以它会导致服务器阻塞一段时间。

##### 返回值

整数值：被该命令修改后的字符串的长度

```powershell
127.0.0.1:6379> SET key1 "Hello world"
OK
127.0.0.1:6379> SETRANGE key1 6 "redis"
(integer) 11
127.0.0.1:6379> GET key1
"Hello redis"

# key2不存在，会在追加"redis"之前用\x00填充6个字节
127.0.0.1:6379> SETRANGE key2 6 "redis"
(integer) 11
127.0.0.1:6379> get key2
"\x00\x00\x00\x00\x00\x00redis"
```

#### INCR

存储在key上的数值自增1。如果key不存在，在执行自增操作之前会先把该key设置为0。如果key存储的字符串无法转换成整数值，执行该命令会报错。该操作的数值范围为64位有符号的整数。

**注意：** 由于Redis没有整数类型，所以它是一个字符串操作。存储在该key上的字符串会被解释为10进制的64位有符号整数，然后在执行操作。

Redis将整数存储在其整数形式中，因此对于实际包含整数的字符串值，不需要存储整数的字符串形式。

##### 返回值

返回递增后的结果

```powershell
127.0.0.1:6379> SET mykey 10
OK
127.0.0.1:6379> INCR mykey
(integer) 11
127.0.0.1:6379> GET mykey
"11"
127.0.0.1:6379> SET mykey "hello"
OK
# 在一个无法转换为10进制的整数字符串上执行INCR操作，会报错
127.0.0.1:6379> INCR mykey
(error) ERR value is not an integer or out of range
```

##### 使用场景

-   计数器 -使用Redis自增操作可以用作计数器。比如以个操作执行过后向Redis发送一个INCR命令。比如对于Web应用来说我们想知道某个用户一天访问了多少次网页，为完成这个目标我们只需要使用该用户ID，拼接上当前日期作为key，然后在用户每次访问网页时在该key上执行INCR操作即可。

-   限速器 - 限速器是一个特殊的计数器，它主要用来限制某个具体操作的频率。我们提供两种使用INCR实现限速器的方式，我们假设要解决的问题是限制一个IP一秒内不能超过10次API调用。

    -   限速器实现方式1 - 如下是最简单直接的实现方式

    ```powershell
    FUNCTION LIMIT_API_CALL(ip)
    ts = CURRENT_UNIX_TIME()
    keyname = ip+":"+ts
    current = GET(keyname)
    IF current != NULL AND current > 10 THEN
        ERROR "too many requests per second"
    ELSE
        MULTI
            INCR(keyname,1)
            EXPIRE(keyname,10)
        EXEC
        PERFORM_API_CALL()
    END
    ```

    首先我们为每个ip在每一秒设置一个计数器，计数器每次自增1然后设置过期时间为10秒，这样只要当前的时间发生了切换之前时间的key会由Redis自动清理掉。这里注意MULTI和EXEC的使用是为了保证自增和设置过期操作是一个原子操作。

    -   限速器实现方式2 使用单计数器，但是要在不产生竞态条件的情况下实现它还是比较复杂。我们将基于这个思路研究不同的变体。

        ```powershell
        FUNCTION LIMIT_API_CALL(ip):
        current = GET(ip)
        IF current != NULL AND current > 10 THEN
            ERROR "too many requests per second"
        ELSE
            value = INCR(ip)
            IF value == 1 THEN
                EXPIRE(ip,1)
            END
            PERFORM_API_CALL()
        END
        ```

        **上面这段代码存在竞态条件。** 如果客户端因为某些原因执行了INCR命令，但没有执行EXPIRE命令，除非这个IP再次发生请求，否则它会发生内存泄漏。

        我们可以通过Lua脚本来执行INCR命令和EXPIRE命令，然后通过EVAL命令执行Lua脚本

        ```lua
        local current
        current = redis.call("incr",KEYS[1])
        if tonumber(current) == 1 then
            redis.call("expire",KEYS[1],1)
        end
        ```

        另外还有一种不使用Lua脚本的方式来修复这个问题，就是使用Redis列表来代替计数器。这个实现使用了许多高级特性相比较而言更复杂，但它有个优势就是能记录当前正在进行API调用的IP地址，这可能非常有用，也可能无用，取决于应用程序。

        ```powershell
        FUNCTION LIMIT_API_CALL(ip)
        current = LLEN(ip)
        IF current > 10 THEN
            ERROR "too many requests per second"
        ELSE
            IF EXISTS(ip) == FALSE
                MULTI
                    RPUSH(ip,ip)
                    EXPIRE(ip,1)
                EXEC
            ELSE
                RPUSHX(ip,ip)
            END
            PERFORM_API_CALL()
        END
        ```

        RPUSHX 命令表示只有当key存在，才会执行push操作。

        注意这里也存在竞态条件，但这并不是问题：当EXISTS返回 FALSE，在执行MULTI/EXEC代码块之前其他的客户端已经创建了该key，这时再执行MULTI/EXEC代码块就会重置该key的过期时间，相当于竞态条件漏统计了这次API调用，限速依然正常工作。

#### INCRBY

与INCR类似，不同的是可以指定递增的步长。

#### INCRBYFLOAT

与INCRBY类似，不同的是以浮点数递增。

#### DECR

与INCR相反，DECR是递减数值。操作数值的范围为64位有符号整数。

##### 返回值

key递减之后的值

```powershell
127.0.0.1:6379> SET mykey 10
OK
127.0.0.1:6379> DECR mykey
(integer) 9
127.0.0.1:6379> SET mykey 234293482390480948029348230948
OK
# 数值越界，返回错误
127.0.0.1:6379> decr mykey
(error) ERR value is not an integer or out of range
```

#### DECRBY

与DECR类似，不同的是可以指定递减步长。

### 位图

#### SETBIT

设置或清除指定位置上的bit。到底是设置还是清除取决于参数值是1还是0，传1就是设置，传0就是清除。

当key不存在时，会自动创建一个新key。字符串会自动扩容以提供offset来设置bit。offset参数的取值范围为[0,2^32^) (位图会限制在512M)。当字符串值扩容时，增加的位默认设置为0。

**注意：** 当设置的key不存在，或者该key存储的字符串较小，Redis扩容申请临时内存会导致服务器阻塞一段时间。

##### 返回值

存储在指定offset上原来的bit值

```powershell
127.0.0.1:6379> SETBIT mykey 7 1
(integer) 0
127.0.0.1:6379> SETBIT mykey 7 0
(integer) 1
127.0.0.1:6379> GET mykey
"\x00"
```

##### 模式

-   设置多个bit - 为了优化设置多个bit 时需要多次调用SETBIT命令的操作，我们可以使用BITFIELD命令。

```powershell
# 为了设置多个bit,需要多次调用SETBIT
127.0.0.1:6379> SETBIT bitmapsarestrings 2 1
(integer) 0
127.0.0.1:6379> SETBIT bitmapsarestrings 3 1
(integer) 0
127.0.0.1:6379> SETBIT bitmapsarestrings 5 1
(integer) 0
127.0.0.1:6379> SETBIT bitmapsarestrings 10 1
(integer) 0
127.0.0.1:6379> SETBIT bitmapsarestrings 11 1
(integer) 0
127.0.0.1:6379> SETBIT bitmapsarestrings 14 1
(integer) 0
127.0.0.1:6379> GET bitmapsarestrings
"42"
# 可以使用BITFIELD命令进行优化
127.0.0.1:6379> BITFIELD bitsinabitmap SET u1 2 1 SET u1 3 1 SET u1 5 1 SET u1 10 1 SET u1 11 1 SET u1 14 1
1) (integer) 0
2) (integer) 0
3) (integer) 0
4) (integer) 0
5) (integer) 0
6) (integer) 0
127.0.0.1:6379> GET bitsinabitmap
"42"
```

#### GETBIT

返回存储在位图上指定offset上的bit值。当offset超过字符串的长度，为通过0字节来填充。

##### 返回值

存储在该offset上的bit值

```powershell
# 在第7位上设置bit值
127.0.0.1:6379> SETBIT mykey 7 1
(integer) 0
127.0.0.1:6379> GETBIT mykey 0
(integer) 0
127.0.0.1:6379> GETBIT mykey 7
(integer) 1
# 超过字符串长度，用0填充，所以返回0
127.0.0.1:6379> GETBIT mykey 10
(integer) 0
```

#### BITCOUNT

统计字符串以二进制形式表示时设置为1的位的个数。

##### 返回值

整数值：bit为1的个数

```powershell
127.0.0.1:6379> SETBIT mykey 10 1
(integer) 0
127.0.0.1:6379> GETBIT mykey 10
(integer) 1
127.0.0.1:6379> GETBIT mykey 11
(integer) 0
127.0.0.1:6379> GETBIT mykey 0
(integer) 0
127.0.0.1:6379> SETBIT mykey 9 1
(integer) 0
127.0.0.1:6379> SETBIT mykey 8 1
(integer) 0
127.0.0.1:6379> GET mykey
"\x00\xe0"
# "\x00\xe0" ==> "0000 0000 1110"
127.0.0.1:6379> BITCOUNT mykey
(integer) 3
# BITCOUNT后跟的start、end是字节索引(非位索引)，且为闭区间[start,end]
127.0.0.1:6379> BITCOUNT mykey 0 0
(integer) 0
127.0.0.1:6379> BITCOUNT mykey 1 0
(integer) 0
127.0.0.1:6379> BITCOUNT mykey 0 1
(integer) 3
127.0.0.1:6379> BITCOUNT mykey 1 1
(integer) 3

# meow ==> "01101101 01100101 01101111 01110111"
127.0.0.1:6379> SET mykey meow
OK
127.0.0.1:6379> BITCOUNT mykey
(integer) 21
```

#### 模式

-   使用位图做实时统计 - 位图是一种高效利用空间来存储某些信息的方法。具体实现方式请参考[Fast easy realtime metrics using Redis bitmaps](http://blog.getspool.com/2011/11/29/fast-easy-realtime-metrics-using-redis-bitmaps)

#### BITOP

在多个存储了字符串的key上执行位运算并把结果存储到目标key中。BITOP 命令支持4种位操作：**AND**，**OR**，**XOR**和**NOT**。 下面是这4种操作命令的正确使用方式

-   BITOP AND destkey srckey1 srckey2 srckey3 ... srckeyN
-   BITOP OR destkey srckey1 srckey2 srckey3 ... srckeyN
-   BITOP XOR destkey srckey1 srckey2 srckey3 ... srckeyN
-   BITOP NOT destkey srckey

如果多个key存储的字符串长度不一，较短的字符串会在后面填充0直到与较长字符串的长度相等，以让它们能完成位运算。

##### 返回值

返回存储到目标key上字符串的长度，也就是输入key中最长的那个字符串的长度。

```powershell
127.0.0.1:6379> SET key1 "foobar"
OK
127.0.0.1:6379> SET key2 "abcdef"
OK
# "foobar" ==> "01100110 01101111 01101111 01100010 01100001 01110010"
# "abcdef" ==> "01100001 01100010 01100011 01100100 01100101 01100110"
# "`bc`ab" ==> "01100000 01100010 01100011 01100000 01100001 01100010"
127.0.0.1:6379> BITOP AND dest key1 key2
(integer) 6
127.0.0.1:6379> GET dest
"`bc`ab"
```

#### BITPOS

返回字符串中第一个设置为0或1的位置。

```powershell
127.0.0.1:6379> SET mykey "\xff\xf0\x00"
OK
# "\xff\xf0\x00" ==> "11111111 11110000 00000000"
127.0.0.1:6379> BITPOS mykey 0
(integer) 12
127.0.0.1:6379> SET mykey "\x00\xff\xf0"
OK
# “\x00\xff\xf0” ==> "00000000 11111111 11110000"
# 从第0个字节开始查找第一个设置为1的位置
127.0.0.1:6379> BITPOS mykey 1 0
(integer) 8
# 从第2个字节开始查找第一个设置为1的位置
127.0.0.1:6379> BITPOS mykey 1 2
(integer) 16
127.0.0.1:6379> set mykey "\x00\x00\x00"
OK
# "\x00\x00\x00" ==> "00000000 00000000 00000000"
127.0.0.1:6379> BITPOS mykey 1
(integer) -1
```

#### BITFIELD

支持三种基于位的操作

-   **GET** <type> <offset> - 返回指定位的字段 
-   **SET** <type> <offset> <value> - 设置指定位字段的值并返回旧值
-   **INCRBY** <type> <offset> <increment> - 递增或递减指定位的字段并返回新值
    -   **OVERFLOW** [WRAP|SAT|FAIL] 主要是配合INCRBY来提供溢出方案

对于我们操作的整数类型，分为有符号的整数和无符号的整数，有符号的整数以i作为前缀，无符号的整数以u作为前缀。比如u8表示一个8位的无符号整数，i16表示一个16位的有符号整数。最大可以支持64位有符号的整数，63位无符号的整数。

指定偏移量有两种方式。如果偏移量没有添加任何前缀，它仅仅只是作为从0开始的偏移量。如果偏移量以#作为前缀，偏移量需要乘以整数类型值的宽度，比如：

```powershell
BITFIELD mystring SET i8 #0 100 i8 #1 200
```

你将在偏移量0处设置8位有符号整数100，在偏移量8处设置8位有符号整数200。

##### 溢出控制

Redis提供了三种溢出控制方案

-   **WRAP** ：从数值范围的最小值到最大值之间滚动。比如8位有符号整数设置为127，递增1之后为-128
-   **SAT**：使用饱和算法，意思是如果从下限越界将一直返回该范围内的最小值，反之如果上限越界将一直返回该范围内的最大值。比如一个8位有符号整数120，自增10，它的结果会返回127。
-   **FAIL**：这种方案如果溢出了不会采取任何行动。返回值为NULL

```powershell
# 默认的溢出策略是滚动
127.0.0.1:6379> BITFIELD mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1
1) (integer) 1
2) (integer) 1
127.0.0.1:6379> BITFIELD mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1
1) (integer) 2
2) (integer) 2
127.0.0.1:6379> BITFIELD mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1
1) (integer) 3
2) (integer) 3
# SAT策略在溢出后一直返回最大值
127.0.0.1:6379> BITFIELD mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1
1) (integer) 0
2) (integer) 3
# FAIL策略在移除后直接返回NIL
127.0.0.1:6379> BITFIELD mykey incrby u2 100 1 OVERFLOW FAIL incrby u2 102 1
1) (integer) 1
2) (nil)
```

## 实现方式

Redis是通过c语言实现的，它并没有直接使用C语言传统的字符串表示，而是基于它构建了一种称之为简单动态字符串(Simple Dynamic String)的抽象类型，并把它作为Redis的默认字符串表示。

具体的底层实现请参考[Redis设计与实现](http://redisbook.com/preview/sds/different_between_sds_and_c_string.html)

## 应用场景

-   Redis字符串可以缓存业务对象
-   Redis字符串可以实现分布式锁
-   Redis字符串可以实现简单的限速器
-   Redis字符串位图可以高效实现活跃用户统计