---
layout: post
title: "Redis transaction 介绍"
description: ""
category: redis
tags: [redis]
---



## Redis 事务（transaction）

每个事务的操作都有begin、commit、rollback

begin 指事务的开始

commit 指事务的提交

rollback 指事务的回滚

### Redis事务的基础：MULTI 、EXEX、DISCARD、WATCH
multi 指事务的开始，

exec  指事务的执行，

discard 指事务的丢弃。> multi
OK
> incr books
QUEUED
> incr books
QUEUED
> exec
(integer) 1
(integer) 2
![事务](../images/redis-images/QQ20181021-140707-shiwu.png)

上图显示了以上事务过程完整的交互效果，所有的指令在 exec 之前不执行，而是缓存在服务器的一个事务队列中，服务器一旦收到 exec 指令，

才开执行整个事务队列，执行完毕后一次性返回所有指令的运行结果。

因为 Redis 的单线程特性，它不用担心自己在执行队列的时候被其它指令打搅，可以保证他们能得到的「原子性」执行。

QUEUED 是一个简单字符串，同 OK 是一个形式，它表示指令已经被服务器缓存到队列里了。

### 事务的ACID性质

> 在Redis中，事务总是具有原子性（Atomicity）、一致性（Consistency）和隔离性（Isolation），

> 并且当Redis运行在某种特定的持久化模式下时，事务也具有持久性（Durability）

#### 原子性(Atomicity)

事务具有的原子性指：数据库讲事务中的多个操作当作一个整体来执行，服务器要么就执行事务中的所有操作，要么就一个操作也不执行。

> 对于redis的事务功能来说，事务队列中的命令要么全部都执行，要么就一个都不执行，因此，Redis也是具有原子性的。
> multi
OK
> set msg "hello"
QUEUED
> get msg
QUEUED
> exec
1) OK
2) hello127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set msg "hello"
QUEUED
127.0.0.1:6379> get
(error) ERR wrong number of arguments for 'get' command
127.0.0.1:6379> set age 20
QUEUED
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get msg
(nil)127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set msg "hello"
QUEUED
127.0.0.1:6379> incr msg
QUEUED
127.0.0.1:6379> set age 20
QUEUED
127.0.0.1:6379> exec
1) OK
2) (error) ERR value is not an integer or out of range
3) OK
127.0.0.1:6379> get msg
"hello"
127.0.0.1:6379> get age
"20"
> Redis的事务和传统的关系型数据库事务最大区别在于，<span style="color:red">Redis不支持事务回滚机制（rollback）</span>，

> 即使事务队列中的某个命令在执行期间出现错误，整个事务也会继续执行下去，直到将事务队列中的所有命令都执行完毕为止。

#### 一致性(Consistency）

事务具有原子性指：如果数据库在执行事务之前是一致性的，那么在事务执行之后，无论事务是否执行成功，数据库也应该仍然是一致的。

“一致” 指的是数据符合数据库本身的定义和要求，没有包含非法或者无效的错误数据。

Redis通过错误的检测和设计来保证事务的一致性。

-  事务入队错误

如果一个事务再入队命令的过程中，出现了命令不存在，或者命令的格式不正确等情况，那么Redis将拒绝执行这个事务。

- 执行错误

   - 执行过程中发生的错误都是一些不能在事务入队时被服务器发现的错误，这些错误只会在命令实际执行时被触发。
   - 即使在事务的执行过程中发生错误，服务器也不会中断事务的执行，它会继续执行事务中余下的其他命令，并且已执行的命令不会被出错的命令影响。

- 服务器停机
  
如果Redis服务器在执行事务的过程中停机，那么根据服务器所使用的持久化模式，也是一致的。

#### 隔离性 (Isolation)

事务的隔离性指：即使数据库中有多个事务并发地执行，各个事务之间也不会互相影响，并且在并发状态下执行的事务和串行执行的事务产生的结果完全相同。

> Redis使用的是单线程的方式来执行事务及事务队列中的命令，并且服务器保证在执行事务期间不会对事务进行中断，

> 因此，Redis的事务总是以串行的方式运行的，并且事务也总是具有隔离性。

#### 持久性(Durability)

事务的耐久性指：当一个事务执行完毕时，执行这个事务所得的结果已经被保存到永久性存储介质里面了，

即使服务器在事务执行完毕之后停机，执行事务所得的结果也不会丢失。

> Redis的事务不过是简单地用队列包裹起了一组Redis的命令，Redis并没有为事务提供任何额外的持久化功能，
> 所以Redis事务的持久性由Redis所使用的持久化模式决定。

### discard （放弃事务）

Redis 为事务提供了一个 discard 指令，用于丢弃事务缓存队列中的所有指令，在 exec 执行之前。
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set msg "hello"
QUEUED
127.0.0.1:6379> set age 20
QUEUED
127.0.0.1:6379> DISCARD
OK
127.0.0.1:6379> mget msg age
1) (nil)
2) (nil)
可以看到 discard 之后，队列中的所有指令都没执行，就好像 multi 和 discard 中间的所有指令从未发生过一样。

### watch

WATCH 命令可以为 Redis 事务提供 check-and-set （CAS）行为。 被 WATCH 的键会被监视，并会发觉这些键是否被改动过了。 

如果有至少一个被监视的键在 EXEC 执行之前被修改了， 那么整个事务都会被取消，

EXEC 返回空多条批量回复（null multi-bulk reply）来表示事务已经失败。

Redis 提供了这种 watch 的机制，它就是一种<span style="color:red;">乐观锁</span>。

watch 会在事务开始之前盯住 1 个或多个关键变量，当事务执行时，也就是服务器收到了 exec 指令要顺序执行缓存的事务队列时，

Redis 会检查关键变量自 watch 之后，是否被修改了 (包括当前事务所在的客户端)。

如果关键变量被人动过了，exec 指令就会返回 null 回复告知客户端事务执行失败，这个时候客户端一般会选择重试。


当服务器给 exec 指令返回一个 null 回复时，客户端知道了事务执行是失败的，

这样客户端需要检查一下返回结果是否为 null 来确定事务是否执行失败。

** <span style="color:red">注意事项: Redis 禁止在 multi 和 exec 之间执行 watch 指令，而必须在 multi 之前做好盯住关键变量，否则会出错。</span> **

### Redis不支持回滚（roll back）

如果你有使用关系式数据库的经验， 那么 “Redis 在事务失败时不进行回滚，而是继续执行余下的命令”这种做法可能会让你觉得有点奇怪。

以下是这种做法的优点：

Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：这也就是说，从实用性的角度来说，失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。
因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。

有种观点认为 Redis 处理事务的做法会产生 bug ， 然而需要注意的是， 在通常情况下， 回滚并不能解决编程错误带来的问题。 举个例子， 如果你本来想通过 INCR 命令将键的值加上 1 ， 却不小心加上了 2 ， 又或者对错误类型的键执行了 INCR ， 回滚是没有办法处理这些情况的。

> 鉴于没有任何机制能避免程序员自己造成的错误， 并且这类错误通常不会在生产环境中出现， 所以 Redis 选择了更简单、更快速的无回滚方式来处理事务。
