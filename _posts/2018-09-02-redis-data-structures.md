---
layout: post
title: "redis 数据结构"
description: ""
category: 
tags: [redis]
---


## Redis 基础数据结构

Redis 有 5 种基础数据结构，分别为：string (字符串)、list (列表)、set (集合)、hash (哈希) 和 zset (有序集合)。


```python
import redis
r = redis.StrictRedis(host='localhost',port=6379,db=0)
```

### string (字符串)

字符串 string 是 Redis 最简单的数据结构。

Redis没有Int、Float、Boolean等数据类型的概念，所有的基本类型在Redis中都以String体现。

Redis 所有的数据结构都是以唯一的 key 字符串作为名称，然后通过这个唯一 key 值来获取相应的 value 数据。不同类型的数据结构的差异就在于 value 的结构不一样。


#### 键值对


```python
r.set('foo','bar')
```




    True




```python
r.get('foo')
```




    b'bar'




```python
r.exists('foo')
```




    True




```python
r.delete('foo')

```




    0




```python
r.exists('foo')
```




    False



#### 批量键值对
可以批量对多个字符串进行读写，节省网络耗时开销。


```python
r.mset({'China':'Beijing','Japan':'Tokyo'})
```




    True




```python
r.mget('China','Japan')
```




    [b'Beijing', b'Tokyo']



#### 过期和 set 命令扩展
可以对 key 设置过期时间，到点自动删除，这个功能常用来控制缓存的失效时间。


```python
!redis-cli -h 127.0.0.1 setnx china shanghai
```

    (integer) 0



```python
!redis-cli -h 127.0.0.1 ttl china
```

    (integer) -1



```python
!redis-cli -h 127.0.0.1 setex china 10 shanghai
!redis-cli -h 127.0.0.1 ttl china 
```

    OK
    (integer) 10



```python
!redis-cli -h 127.0.0.1 ttl china 
```

    (integer) 4



```python
!redis-cli -h 127.0.0.1 set China Beijing EX 20 XX
```

    OK


#### 计数

如果 value 值是一个整数，还可以对它进行自增操作。自增是有范围的，它的范围是 signed long 的最大最小值，超过了这个值，Redis 会报错。


```python
!redis-cli -h 127.0.0.1 set age 20
!redis-cli -h 127.0.0.1 incr age   
!redis-cli -h 127.0.0.1 incrby age 4
!redis-cli -h 127.0.0.1 incrby age -5 
!redis-cli -h 127.0.0.1 decrby age 5 
!redis-cli -h 127.0.0.1 decrby age -5 
!redis-cli -h 127.0.0.1 decr age 
```

    OK
    (integer) 21
    (integer) 25
    (integer) 20
    (integer) 15
    (integer) 20
    (integer) 19



#### 归纳与String相关的常用命令：

- SET：为一个key设置value，可以配合EX/PX参数指定key的有效期，通过NX/XX参数针对key是否存在的情况进行区别操作，时间复杂度O(1)

- GET：获取某个key对应的value，时间复杂度O(1)

- GETSET：为一个key设置value，并返回该key的原value，时间复杂度O(1)

- MSET：为多个key设置value，时间复杂度O(N)

- MSETNX：同MSET，如果指定的key中有任意一个已存在，则不进行任何操作，时间复杂度O(N)

- MGET：获取多个key对应的value，时间复杂度O(N)

Redis的基本数据类型只有String，但Redis可以把String作为整型或浮点型数字来使用，主要体现在INCR、DECR类的命令上：

- INCR：将key对应的value值自增1，并返回自增后的值。只对可以转换为整型的String数据起作用。时间复杂度O(1)

- INCRBY：将key对应的value值自增指定的整型数值，并返回自增后的值。只对可以转换为整型的String数据起作用。时间复杂度O(1)

- DECR/DECRBY：同INCR/INCRBY，自增改为自减。

INCR/DECR系列命令要求操作的value类型为String，并可以转换为64位带符号的整型数字，否则会返回错误。

也就是说，进行INCR/DECR系列命令的value，必须在[-2^63 ~ 2^63 - 1]范围内。


Redis采用单线程模型，天然是线程安全的，这使得INCR/DECR命令可以非常便利的实现高并发场景下的精确控制
- 库存控制
- 自增序列

### list（列表）

Redis将列表数据结构命名为list而不是array，是因为列表的存储结构用的是链表而不是数组，而且链表还是双向链表。

这意味着 list 的插入和删除操作非常快，时间复杂度为 O(1)，但是索引定位很慢，时间复杂度为 O(n)

当列表弹出了最后一个元素之后，该数据结构自动被删除，内存被回收。

Redis 的列表结构常用来做异步队列使用。

将需要延后处理的任务结构体序列化成字符串塞进 Redis 的列表，另一个线程从这个列表中轮询数据进行处理。

> 负下标 链表元素的位置使用自然数0,1,2,....n-1表示，还可以使用负数-1,-2,...-n来表示，-1表示「倒数第一」，-2表示「倒数第二」，那么-n就表示第一个元素，对应的下标为0。

#### 右进左出| 左进右出：队列


```python
!redis-cli -h 127.0.0.1 del books 
!redis-cli -h 127.0.0.1 rpush books php go python
!redis-cli -h 127.0.0.1 llen books 
!redis-cli -h 127.0.0.1 lrange books 0 -1 
```

    (integer) 3
    (integer) 3
    1) "php"
    2) "go"
    3) "python"



```python
!redis-cli -h 127.0.0.1 lpop books
!redis-cli -h 127.0.0.1 lpop books
!redis-cli -h 127.0.0.1 lpop books
!redis-cli -h 127.0.0.1 lpop books
```

    "php"
    "go"
    "python"
    (nil)


#### 右进右出|左进左出：栈


```python
!redis-cli -h 127.0.0.1 del books 
!redis-cli -h 127.0.0.1 rpush books php go python 
!redis-cli -h 127.0.0.1 llen books 
!redis-cli -h 127.0.0.1 lrange books 0 -1 
!redis-cli -h 127.0.0.1 rpop  books 
!redis-cli -h 127.0.0.1 rpop  books 
!redis-cli -h 127.0.0.1 rpop  books 
!redis-cli -h 127.0.0.1 rpop  books 
```

    (integer) 1
    (integer) 3
    (integer) 3
    1) "php"
    2) "go"
    3) "python"
    "python"
    "go"
    "php"
    (nil)


#### 慢操作

> lindex 它需要对链表进行遍历，性能随着参数index增大而变差。

> ltrim 跟的两个参数start_index和end_index定义了一个区间，在这个区间内的值，ltrim 要保留，区间之外统统砍掉。

我们可以通过ltrim来实现一个定长的链表，这一点非常有用。

index 可以为负数，index=-1表示倒数第一个元素，同样index=-2表示倒数第二个元素。



```python
!redis-cli -h 127.0.0.1 rpush lang php go python 
!redis-cli -h 127.0.0.1 lindex lang 1
```

    (integer) 3
    "go"



```python
!redis-cli -h 127.0.0.1 ltrim lang 1 -1 
!redis-cli -h 127.0.0.1 lrange lang 0 -1
```

    OK


#### 归纳与List相关的常用命令

- LPUSH：向指定List的左侧（即头部）插入1个或多个元素，返回插入后的List长度。时间复杂度O(N)，N为插入元素的数量
- RPUSH：同LPUSH，向指定List的右侧（即尾部）插入1或多个元素
- LPOP：从指定List的左侧（即头部）移除一个元素并返回，时间复杂度O(1)
- RPOP：同LPOP，从指定List的右侧（即尾部）移除1个元素并返回
- LPUSHX/RPUSHX：与LPUSH/RPUSH类似，区别在于，LPUSHX/RPUSHX操作的key如果不存在，则不会进行任何操作
- LLEN：返回指定List的长度，时间复杂度O(1)
- LRANGE：返回指定List中指定范围的元素（双端包含，即LRANGE key 0 10会返回11个元素），时间复杂度O(N)。应尽可能控制一次获取的元素数量，一次获取过大范围的List元素会导致延迟，同时对长度不可预知的List，避免使用LRANGE key 0 -1这样的完整遍历操作。

**应谨慎使用的List相关命令**

- LINDEX：返回指定List指定index上的元素，如果index越界，返回nil。index数值是回环的，即-1代表List最后一个位置，-2代表List倒数第二个位置。时间复杂度O(N)
- LSET：将指定List指定index上的元素设置为value，如果index越界则返回错误，时间复杂度O(N)，如果操作的是头/尾部的元素，则时间复杂度为O(1)
- LINSERT：向指定List中指定元素之前/之后插入一个新元素，并返回操作后的List长度。如果指定的元素不存在，返回-1。如果指定key不存在，不会进行任何操作，时间复杂度O(N)

>> Redis还提供了一系列阻塞式的操作命令，如BLPOP/BRPOP等，能够实现类似于BlockingQueue的能力，即在List为空时，阻塞该连接，直到List中有对象可以出队时再返回

### hash (字典)

hash 结构也可以用来存储用户信息，不同于字符串一次性需要全部序列化整个对象，hash 可以对用户结构中的每个字段单独存储。

这样当我们需要获取用户信息时可以进行部分获取。而以整个字符串的形式去保存用户信息的话就只能一次性全部读取，这样就会比较浪费网络流量。

hash 也有缺点，hash 结构的存储消耗要高于单个字符串，到底该使用 hash 还是字符串，需要根据实际情况再三权衡。


Hash的优点包括：

- 可以实现二元查找，如"查找ID为1000的用户的年龄"

- 比起将整个对象序列化后作为String存储的方法，Hash能够有效地减少网络传输的消耗

- 当使用Hash维护一个集合时，提供了比List效率高得多的随机访问命令

**增加元素** 

可以使用hset一次增加一个键值对，也可以使用hmset一次增加多个键值对


```python
!redis-cli -h 127.0.0.1 hset website google google.com
!redis-cli -h 127.0.0.1 hmset website baidu baidu.com zhihu zhihu.com 
```

    (integer) 0
    OK


**获取元素** 

可以通过hget定位具体key对应的value

可以通过hmget获取多个key对应的value

可以使用hgetall获取所有的键值对

可以使用hkeys和hvals分别获取所有的key列表和value列表


```python
!redis-cli -h 127.0.0.1 hget website google 
```

    "google.com"



```python
!redis-cli -h 127.0.0.1 hmget website zhihu baidu
```

    1) "zhihu.com"
    2) "baidu.com"



```python
!redis-cli -h 127.0.0.1 hgetall website 
```

    1) "google"
    2) "google.com"
    3) "baidu"
    4) "baidu.com"
    5) "zhihu"
    6) "zhihu.com"



```python
!redis-cli -h 127.0.0.1 hkeys website 
```

    1) "google"
    2) "baidu"
    3) "zhihu"



```python
!redis-cli -h 127.0.0.1 hvals website 
```

    1) "google.com"
    2) "baidu.com"
    3) "zhihu.com"


**删除元素**

可以使用hdel删除指定key，hdel支持同时删除多个key


```python
!redis-cli -h 127.0.0.1 hdel website google
```


```python
!redis-cli -h 127.0.0.1 hdel website baidu zhihu 
```

    (integer) 2


**判断元素是否存在**

通常我们使用hget获得key对应的value是否为空就知道对应的元素是否存在了，不过如果value的字符串长度特别大，通过这种方式来判断元素存在与否就略显浪费，这时可以使用hexists指令。


```python
!redis-cli -h 127.0.0.1 hexists website  baidu
```

    (integer) 0


**计数器** 

hash结构还可以当成计数器来使用，对于内部的每一个key都可以作为独立的计数器。

如果value值不是整数，调用hincrby指令会出错。


```python
!redis-cli -h 127.0.0.1 hget website googleAge 
!redis-cli -h 127.0.0.1 hincrby website googleAge 20 
!redis-cli -h 127.0.0.1 hget website googleAge 
!redis-cli -h 127.0.0.1 hset website googleName google
!redis-cli -h 127.0.0.1 hincrby website googleName 
```

    "40"
    (integer) 60
    "60"
    (integer) 1
    (error) ERR wrong number of arguments for 'hincrby' command


**归纳Hash相关的常用命令**

- HSET：将key对应的Hash中的field设置为value。如果该Hash不存在，会自动创建一个。时间复杂度O(1)

- HGET：返回指定Hash中field字段的值，时间复杂度O(1)

- HMSET/HMGET：同HSET和HGET，可以批量操作同一个key下的多个field，时间复杂度：O(N)，N为一次操作的field数量

- HSETNX：同HSET，但如field已经存在，HSETNX不会进行任何操作，时间复杂度O(1)

- HEXISTS：判断指定Hash中field是否存在，存在返回1，不存在返回0，时间复杂度O(1)

- HDEL：删除指定Hash中的field（1个或多个），时间复杂度：O(N)，N为操作的field数量

- HINCRBY：同INCRBY命令，对指定Hash中的一个field进行INCRBY，时间复杂度O(1)

应谨慎使用的Hash相关命令：

- HGETALL：返回指定Hash中所有的field-value对。返回结果为数组，数组中field和value交替出现。时间复杂度O(N)

- HKEYS/HVALS：返回指定Hash中所有的field/value，时间复杂度O(N)

上述三个命令都会对Hash进行完整遍历，Hash中的field数量与命令的耗时线性相关，对于尺寸不可预知的Hash，
应严格避免使用上面三个命令，而改为使用HSCAN命令进行游标式的遍历，



### set （集合）

Redis的集合，它内部的键值对是无序的唯一的。

它的内部实现相当于一个特殊的字典，字典中所有的 value 都是一个值NULL。

当集合中最后一个元素移除之后，数据结构自动删除，内存被回收。

**增加元素**

可以一次增加多个元素


```python
!redis-cli -h 127.0.0.1  SADD bbs "discuz.net"
!redis-cli -h 127.0.0.1  SADD bbs "discuz.net"
!redis-cli -h 127.0.0.1  SADD bbs "tianya.cn" "groups.google.com"
```

    (integer) 1
    (integer) 0
    (integer) 2


**读取元素**  

使用smembers列出所有元素

使用scard获取集合长度

使用srandmember获取随机count个元素，如果不提供count参数，默认为1


```python
!redis-cli -h 127.0.0.1  SMEMBERS bbs 
!redis-cli -h 127.0.0.1  SCARD bbs 
!redis-cli -h 127.0.0.1  SRANDMEMBER bbs 
```

    1) "tianya.cn"
    2) "discuz.net"
    3) "groups.google.com"
    (integer) 3
    "groups.google.com"


**删除元素**

使用srem删除一到多个元素

使用spop删除随机一个元素


```python
!redis-cli -h 127.0.0.1 SREM bbs

!redis-cli -h 127.0.0.1 SPOP bbs
```

**判断元素是否存在** 

使用sismember指令，只能接收单个元素


```python
!redis-cli -h 127.0.0.1 SISMEMBER bbs tianya.cn
```

**归纳与Set相关的常用命令**

- SADD：向指定Set中添加1个或多个member，如果指定Set不存在，会自动创建一个。时间复杂度O(N)，N为添加的member个数

- SREM：从指定Set中移除1个或多个member，时间复杂度O(N)，N为移除的member个数

- SRANDMEMBER：从指定Set中随机返回1个或多个member，时间复杂度O(N)，N为返回的member个数

- SPOP：从指定Set中随机移除并返回count个member，时间复杂度O(N)，N为移除的member个数

- SCARD：返回指定Set中的member个数，时间复杂度O(1)

- SISMEMBER：判断指定的value是否存在于指定Set中，时间复杂度O(1)

- SMOVE：将指定member从一个Set移至另一个Set

慎用的Set相关命令：

- SMEMBERS：返回指定Hash中所有的member，时间复杂度O(N)

- SUNION/SUNIONSTORE：计算多个Set的并集并返回/存储至另一个Set中，时间复杂度O(N)，N为参与计算的所有集合的总member数

- SINTER/SINTERSTORE：计算多个Set的交集并返回/存储至另一个Set中，时间复杂度O(N)，N为参与计算的所有集合的总member数

- SDIFF/SDIFFSTORE：计算1个Set与1或多个Set的差集并返回/存储至另一个Set中，时间复杂度O(N)，N为参与计算的所有集合的总member数

上述几个命令涉及的计算量大，应谨慎使用，特别是在参与计算的Set尺寸不可知的情况下，应严格避免使用。可以考虑通过SSCAN命令遍历获取相关Set的全部member

如果需要做并集/交集/差集计算，可以在客户端进行，或在不服务实时查询请求的Slave上进行。


### zset（有序集合）

Redis Sorted Set是有序的、不可重复的String集合。

一方面它是一个 set，保证了内部 value 的唯一性，另一方面它可以给每个 value 赋予一个 score，代表这个 value 的排序权重。它的内部实现用的是一种叫做「跳跃列表」的数据结构。

Sorted Set中的每个元素都需要指派一个分数(score)，Sorted Set会根据score对元素进行升序排序。如果多个member拥有相同的score，则以字典序进行升序排序。

zset 中最后一个 value 被移除后，数据结构自动删除，内存被回收。

Sorted Set非常适合用于实现排名。


**增加元素**

通过zadd指令可以增加一到多个value/score对，score放在前面


```python
!redis-cli -h 127.0.0.1 ZADD page_rank 10 google.com 
!redis-cli -h 127.0.0.1 ZADD page_rank 9 baidu.com  8 zhihu.com
!redis-cli -h 127.0.0.1  ZRANGE page_rank 0 -1 withscores
```

    (integer) 1
    (integer) 1
    1) "zhihu.com"
    2) "8"
    3) "baidu.com"
    4) "9"
    5) "google.com"
    6) "10"


**长度** 

通过指令zcard可以得到zset的元素个数


```python
!redis-cli -h 127.0.0.1 ZCARD page_rank 
```

    (integer) 3


**删除元素** 

通过指令zrem可以删除zset中的元素，可以一次删除多个


```python
!redis-cli -h 127.0.0.1 ZREM page_rank  google.com  baidu.com
!redis-cli -h 127.0.0.1  ZRANGE page_rank 0 -1 withscores
```

    (integer) 2
    1) "zhihu.com"
    2) "8"


**计数器** 

同hash结构一样，zset也可以作为计数器使用。


```python
!redis-cli -h 127.0.0.1 ZINCRBY page_rank 1 zhihu.com 
!redis-cli -h 127.0.0.1  ZRANGE page_rank 0 -1 withscores
```

    "9"
    1) "zhihu.com"
    2) "9"


**获取排名和分数**

通过zscore指令获取指定元素的权重，

通过zrank指令获取指定元素的正向排名，

通过zrevrank指令获取指定元素的反向排名[倒数第一名]。正向是由小到大，负向是由大到小。


```python
!redis-cli -h 127.0.0.1  zscore page_rank zhihu.com

!redis-cli -h 127.0.0.1 zrank page_rank zhihu.com 

!redis-cli -h 127.0.0.1 zrank page_rank baidu.com 

!redis-cli -h 127.0.0.1 zrank page_rank google.com 

!redis-cli -h 127.0.0.1 zrevrank page_rank google.com 
```

    "8"
    (integer) 0
    (integer) 1
    (integer) 2
    (integer) 0


**根据排名范围获取元素列表**

通过zrange指令指定排名范围参数获取对应的元素列表，携带withscores参数可以一并获取元素的权重。

通过zrevrange指令按负向排名获取元素列表[倒数]。正向是由小到大，负向是由大到小。


```python
!redis-cli -h 127.0.0.1 zrange page_rank 0 -1 
```

    1) "zhihu.com"
    2) "baidu.com"
    3) "google.com"



```python
!redis-cli -h 127.0.0.1 zrange page_rank 0 -1  withscores
```

    1) "zhihu.com"
    2) "8"
    3) "baidu.com"
    4) "9"
    5) "google.com"
    6) "10"



```python
!redis-cli -h 127.0.0.1 zrevrange page_rank 0 -1  withscores
```

    1) "google.com"
    2) "10"
    3) "baidu.com"
    4) "9"
    5) "zhihu.com"
    6) "8"


**根据score范围获取列表**

通过zrangebyscore指令指定score范围获取对应的元素列表。

通过zrevrangebyscore指令获取倒排元素列表。正向是由小到大，负向是由大到小。参数-inf表示负无穷，+inf表示正无穷。


```python
!redis-cli -h 127.0.0.1 zrangebyscore page_rank 8 10 
!redis-cli -h 127.0.0.1 zrangebyscore page_rank 8 10  withscores
```

    1) "zhihu.com"
    2) "baidu.com"
    3) "google.com"
    1) "zhihu.com"
    2) "8"
    3) "baidu.com"
    4) "9"
    5) "google.com"
    6) "10"



```python
!redis-cli -h 127.0.0.1 zrangebyscore page_rank -inf +inf withscores
```

    1) "zhihu.com"
    2) "8"
    3) "baidu.com"
    4) "9"
    5) "google.com"
    6) "10"



```python
!redis-cli -h 127.0.0.1 zrangebyscore page_rank +inf -inf withscores
```

    (empty list or set)


**根据范围移除元素列表**

可以通过排名范围，ZREMRANGEBYRANK key start stop

也可以通过score范围来一次性移除多个元素 ZREMRANGEBYSCORE key min max


```python
!redis-cli -h 127.0.0.1 zadd salary 2000 jack  5000 tom  3500 peter
!redis-cli -h 127.0.0.1 ZREMRANGEBYRANK salary 0 1 
!redis-cli -h 127.0.0.1 ZRANGE salary 0 -1 WITHSCORES
```

    (integer) 3
    (integer) 2
    1) "tom"
    2) "5000"



```python
!redis-cli -h 127.0.0.1 zadd salary 2000 jack  5000 tom  3500 peter
!redis-cli -h 127.0.0.1 ZREMRANGEBYSCORE salary 1500 3500
!redis-cli -h 127.0.0.1 ZRANGE salary 0 -1 WITHSCORES
```

    (integer) 2
    (integer) 2
    1) "tom"
    2) "5000"


**归纳Sorted Set的主要命令**


- ZADD：向指定Sorted Set中添加1个或多个member，时间复杂度O(Mlog(N))，M为添加的member数量，N为Sorted Set中的member数量
- ZREM：从指定Sorted Set中删除1个或多个member，时间复杂度O(Mlog(N))，M为删除的member数量，N为Sorted Set中的member数量
- ZCOUNT：返回指定Sorted Set中指定score范围内的member数量，时间复杂度：O(log(N))
- ZCARD：返回指定Sorted Set中的member数量，时间复杂度O(1)
- ZSCORE：返回指定Sorted Set中指定member的score，时间复杂度O(1)
- ZRANK/ZREVRANK：返回指定member在Sorted Set中的排名，ZRANK返回按升序排序的排名，ZREVRANK则返回按降序排序的排名。时间复杂度O(log(N))
- ZINCRBY：同INCRBY，对指定Sorted Set中的指定member的score进行自增，时间复杂度O(log(N))

慎用的Sorted Set相关命令：

- ZRANGE/ZREVRANGE：返回指定Sorted Set中指定排名范围内的所有member，ZRANGE为按score升序排序，ZREVRANGE为按score降序排序，时间复杂度O(log(N)+M)，M为本次返回的member数
- ZRANGEBYSCORE/ZREVRANGEBYSCORE：返回指定Sorted Set中指定score范围内的所有member，返回结果以升序/降序排序，min和max可以指定为-inf和+inf，代表返回所有的member。时间复杂度O(log(N)+M)
- ZREMRANGEBYRANK/ZREMRANGEBYSCORE：移除Sorted Set中指定排名范围/指定score范围内的所有member。时间复杂度O(log(N)+M)


#### 容器型数据结构的通用规则

list/set/hash/zset 这四种数据结构是容器型数据结构，它们共享下面两条通用规则：

- create if not exists

如果容器不存在，那就创建一个，再进行操作。比如 rpush 操作刚开始是没有列表的，Redis 就会自动创建一个，然后再 rpush 进去新元素。

- drop if no elements

如果容器里元素没有了，那么立即删除元素，释放内存。这意味着 lpop 操作到最后一个元素，列表就消失了...


### Bitmap (位图)

Bitmap在Redis中不是一种实际的数据类型，而是一种将String作为Bitmap使用的方法。

可以理解为将String转换为bit数组。使用Bitmap来存储true/false类型的简单数据极为节省空间。

### HyperLogLog


HyperLogLogs是一种主要用于数量统计的数据结构，它和Set类似，维护一个不可重复的String集合，但是HyperLogLogs并不维护具体的member内容，只维护member的个数。

也就是说，HyperLogLogs只能用于计算一个集合中不重复的元素数量，所以它比Set要节省很多内存空间。


### GeoHash

redis 在 3.2 版本以后增加了地理位置 GEO 模块，

意味着我们可以使用 Redis 来实现摩拜单车「附近的 Mobike」、美团和饿了么「附近的餐馆」的功能

