---
layout: post
title: "Redis HyperLogLog 介绍"
description: ""
category: redis
tags: [redis]
---


## 场景

<span style="color:red"><b>思考：如何开发一个统计模块，实现网站每个网页每天的PV数据和 UV 数据</b></span>

如果统计 PV 那非常好办，给每个网页一个独立的 Redis 计数器就可以了，这个计数器的 key 后缀加上当天的日期。

这样来一个请求，incrby 一次，最终就可以统计出所有的 PV 数据。

但是 UV 不一样，它要去重，同一个用户一天之内的多次访问请求只能计数一次。

这就要求每一个网页请求都需要带上用户的 ID，无论是登陆用户还是未登陆用户都需要一个唯一 ID 来标识。



<hr>
也许你想到了一个简单的方案，那就是为每一个页面一个独立的 set 集合来存储所有当天访问过此页面的用户 ID。当一个请求过来时，

我们使用 sadd 将用户 ID 塞进去就可以了。通过 scard 可以取出这个集合的大小，这个数字就是这个页面的 UV 数据。没错，这是一个非常简单的方案。

但是，如果你的页面访问量非常大，比如一个爆款页面几千万的 UV，你需要一个很大的 set 集合来统计，这就非常浪费空间。

如果这样的页面很多，那所需要的存储空间是惊人的。为这样一个去重功能就耗费这样多的存储空间，值得么？

其实如果统计的数据又不需要太精确，105w 和 106w 这两个数字并没有多大区别，那有没有更好的解决方案呢？

<hr>

Redis 提供了 HyperLogLog 数据结构就是用来解决这种统计问题的。

HyperLogLog 提供不精确的去重计数方案，虽然不精确但是也不是非常不精确，标准误差是 0.81%，这样的精确度已经可以满足上面的 UV 统计需求了。

## 使用方法

HyperLogLog 提供了两个指令 pfadd 和 pfcount，根据字面意义很好理解，一个是增加计数，一个是获取计数。

pfadd 用法和 set 集合的 sadd 是一样的，来一个用户 ID，就将用户 ID 塞进去就是。

pfcount 和 scard 用法是一样的，直接获取计数值。


```python
!redis-cli del uv
!redis-cli pfadd uv user1
!redis-cli pfadd uv user2 user3 user4 user5 user6 user7 user8 user9 user10 
!redis-cli pfcount uv 
```

    (integer) 1
    (integer) 1
    (integer) 1
    (integer) 10



简单试了一下，发现还蛮精确的，一个没多也一个没少。接下来我们使用脚本，往里面输入更多的数据，看看它是否还可以继续精确下去，如果不能精确，差距有多大。



```python
import redis

client = redis.StrictRedis()
for i in range(100):
    client.pfadd("uv", "user%d" % i)
    total = client.pfcount("uv")
    if total != i+1:
        print(total, i+1)
        break
```

    99 100


当我们加入第 100 个元素时，结果开始出现了不一致。接下来我们将数据增加到 10w 个，看看总量差距有多大。


```python
import redis

client = redis.StrictRedis()
for i in range(100000):
    client.pfadd("uv", "user%d" % i)
print(100000, client.pfcount("uv"))
```

    100000 99723


差了 277 个，按百分比是 0.277%，对于上面的 UV 统计需求来说，误差率也不算高。

然后我们把上面的脚本再跑一边，也就相当于将数据重复加入一边，查看输出，可以发现，pfcount 的结果没有任何改变，还是 99723，

说明它确实具备去重功能。

### pfmerge 场景使用


```python
HyperLogLog 除了上面的 pfadd 和 pfcount 之外，还提供了第三个指令 pfmerge，用于将多个 pf 计数值累加在一起形成一个新的 pf 值。

比如在网站中我们有两个内容差不多的页面，运营说需要这两个页面的数据进行合并。其中页面的 UV 访问量也需要合并，

那这个时候 pfmerge 就可以派上用场了。

```


```python
!redis-cli PFADD  nosql  "Redis"  "MongoDB"  "Memcached"
!redis-cli PFADD  RDBMS  "MySQL" "MSSQL" "PostgreSQL"
!redis-cli PFMERGE  databases  nosql  RDBMS
!redis-cli PFCOUNT  databases
```

    (integer) 1
    (integer) 1
    OK
    (integer) 6


## 注意事项

   HyperLogLog 这个数据结构不是免费的，不是说使用这个数据结构要花钱，它需要占据一定 12k 的存储空间，所以它不适合统计单个用户相关的数据。

如果你的用户上亿，可以算算，这个空间成本是非常惊人的。但是相比 set 存储方案，HyperLogLog 所使用的空间那真是可以使用千斤对比四两来形容了。


   不过你也不必过于担心，因为 Redis 对 HyperLogLog 的存储进行了优化，在计数比较小时，它的存储空间采用稀疏矩阵存储，空间占用很小，

仅仅在计数慢慢变大，稀疏矩阵占用空间渐渐超过了阈值时才会一次性转变成稠密矩阵，才会占用 12k 的空间。


## HyperLogLog 实现原理   (感兴趣可以深入研究)