#缓存使用介绍和redis介绍

读了一篇缓存使用的文章，搬过来介绍一下，顺便总结一下自己在项目中的redis应用

**缓存介绍：**<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;降低应用程序对物理数据源访问的频次，从而提高应用程序的运行性能。缓存内的数据是对物理数据源中的数据的复制，应用程序在运行时从缓存读写数据，用特定的事件或事件来同步。

**使用场景：**<br />

* 很少被修改的数据(读多写少)
* 常量数据
* 不会并发使用的数据
* 非共享的数据<br />
* ......

**分类**

1. 本地缓存(localcache)
2. memcache
3. redis

本地缓存：适用数据量少，修改不频繁的场景，如列表数据、黑白名单、配置信息等。因为这些数据量很少，可以在本地内存中。缺点是对于分布式部署的多台机器信息更新时可能会出现数据不一致情况。<br />

memcache：适合大量高并发数据读写，提供了cas命令，可以保证多个并发访问操作同一份数据的一致性问题；因为是全内存的数据缓冲系统并且只支持简单的key-value存储，所以在存储大数据优于redis。缺点是不支持持久化，数据结构单一。

redis：不仅仅支持简单的k/v类型的数据，同时还提供list，set，zset，hash等数据结构的存储；支持数据备份和持久化。与memcache区别是不是所有数据都会存在内存中[^1]。缺点是高并发大数据会出现阻塞问题。

[^1]: redis的持久化会根据一个阀值来操作，如果选用RDB持久化会根据 etc/redis.conf配置文件中的save * *值来计算怎么执行BGSAVE(bgsave以子进程执行，不会阻塞其它命令执行；save执行过程中会阻塞) ，如果是AOF持久化会根据配置文件appendfsync的值来计算执行（aof是将缓冲区的命令存入aof文件中）。

**先操作数据库和先操作缓存问题**

当更新数据时候需要对缓存操作，那么又有两种情况

1. 先写数据库，在淘汰缓存(或者更新缓存)
2. 先淘汰缓存，再写数据库

第一种情况：第一步先写数据库，成功后淘汰缓存，如果缓存操作失败，cache中就是老数据，数据不一致。
第二种情况：先淘汰缓存，成功后写库，写库失败会引起一次cache miss，并且库里的数据是错误的。

这里通常建议是第二种，保证取得数据是正确的。可以根据实际需求选择。<br />

这里还要讨论一下更新缓存和淘汰缓存<br />
淘汰缓存：是先清除数据，当有请求的时候再更新，会引起一次cache miss。<br />
更新缓存：是在数据入库的时候还会把数据写入缓存。

在选择方案的时候要考虑更新缓存的代价，可以根据需求倾向于哪个方案。

**最后总结一下区别**

1.	memcache中所有数据存储在内存中;redis不是所有数据都在内存中。
2.	memcache仅支持简单的k/v类型的数据;redis不仅仅支持简单的k/v类型的数据还支持list，set，zset，hash等结构。
3.	memcache是多线程，非阻塞IO复用的网络模型；redis使用单线程的IO复用模型。
4.	memcached提供了cas命令，可以保证多个并发访问操作同一份数据的一致性问题； redis没有提供cas命令，不过redis提供了事务的功能，可以保证一串命令的原子性，中间不会被任何操作打断。
5.	memcache挂掉后数据无法恢复；redis可以利用定期备份恢复数据，保证的数据安全和完整。

redis更多场景是作为memcache的替代者来使用;当需要除key/value之外的更多数据类型支持时，使用redis更合适;当存储的数据不能被剔除时，对数据持久化和数据同步要求的使用redis更合适。


**redis在项目中的应用**
先介绍一下redis的五种结构类型

* string ：普通的key/value类型都是此类型结构存储，常用命令：set、get、mget、incr、decr等
* hash ：当value是一个hashMap结构时会使用该结构，map中的结构是field=>value，可以通过key+field来对响应的数据进行操作。常用命令有hset、hget、hgetall。
* list ：list列表底层采用双端链表结构，这样在操作遍历查找时候非常方便。常用命令：lpush、rpush、lpop、rpop、lrange。
* set ：set集合，可以理解集合是排重的列表，在集合里面不会保存重复数据。常用命令：sadd、spop、smembers、sunion
* sorted set ：有序集合，与set的区别是有序集合根据score参数来进行排序的。常用命令：zadd、zrange、zrem、zcard。

这里介绍一下有序集合的使用：标签系统，现在有近两百万的粉丝，每个粉丝都会有一些标签，最近统计了一下mysql中存了大约2800w条标签。其实单表280w统计信息时候就已经要长时间了，当表扩大10倍mysql中更难处理了；并且需要两个维度，一时根据粉丝id获取标签，另一个就是根据标签id获取粉丝id、对粉丝id的统计处理。这里就利用到了有序集合。

```

zadd(tag_id, score, uid)；	//	为标签添加用户id

zadd(uid, score, tag_id)；	//	为用户打标签
```

用zcard统计标签使用量，用zrevrange返回用户标签id信息。这里单品牌3000w条标签数据使用正常。还有关联上一条命令就是产品里面有多个品牌，对每个品牌打标签时候使用hIncrBy分别统计单品牌总标签数量。

队列使用：群发、任务脚本、后台信息同步等等。
通过push和pop往队列里面插入和消费，对消费失败的再次加入队列或者记录一下。对非实时的信息通过队列来处理缓解服务器压力。队列使用很常见，很多地方都会用到。


