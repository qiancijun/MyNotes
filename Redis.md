---
title: Redis
date: 2020-05-28 19:35:23
tags:
categories:
    - 数据库
mathjax: true
---
# NoSQL概述
* NoSQL(Not Only SQL)：$\color{red}{泛指非关系型数据库}$。
    + 传统的关系数据库在应付web2.0网站，特别是超大规模和高并发的SNS类型的动态网站已经显得力不从心，暴露了很多难以克服的问题，而非关系型数据库则由于其本身的特点得到了非常迅速的发展。NoSQL数据库的产生就是为了解决大规模数据集合多重数据种类带来的挑战，尤其是大数据应用难题，包括超大规模数据的存储。
    + 这些类型的数据存储不需要固定的模式，无需多余操作就可以横向扩展

## 特点
1. 易扩展：
    + NoSQL数据库种类繁多，但是一个共同的特点都是去掉关系数据库的关系型特性。
    + 数据之间无关系，这样就非常容易扩展。也无形之间，在架构的层面上带来了可扩展的能力
2. 大数据高性能：
    + NoSQL数据库都具有非常高的读写性能，尤其在大数据量下，同样表现优秀。这得益于它的无关系性，数据库的结构简单
    + 一般MySQL使用Query Cache，每次表的更新Cache就失败，是一种大粒度的Cache，在针对web2.0的交互频繁的应用，Cache性能不高，而NoSQL的Cache是记录级的，是一种细粒度的Cache，所以NoSQL在这个层面上来说性能高很多了
3. 多样灵活的数据模型
    + NoSQL无需事先为要存储的数据建立字段，随时可以存储自定义的数据格式。而在关系数据库里，增删字段是一件非常麻烦的事情。如果是非常大数据量的表，增加字段简直是一个噩梦。
4. 和传统关系型数据库的对比
* RDBMS：
    - 高度组织化结构化数据
    - 结构化查询语言(SQL)
    - 数据和关系都存储在单独的表中
    - 数据操纵语言，数据定义语言
    - 严格的一致性
    - 基础事务
* NoSQL
    - 代表着不仅仅是SQL
    - 没有声明性查询语言
    - 没有预定义的模式
    - 键值对存储，列存储，文档存储，图形数据库
    - 最终一致性，而非ACID属性
    - 非结构化和不可预知的数据
    - CAP定理
    - 高性能，高可用性和可伸缩性

## NoSQL数据模型
* `BSON`：Binary JSON一种类JSON的一种二进制形式的存储格式。和JSON一样，支持内嵌的文档对象和数组对象。

* 聚合模型：
    + KV键值
    + BSON
        - MongoDB：是一个基于分布式文件存储的数据库，由C++编写，旨在为WEB应用提供可扩展的高性能数据库存储解决方案。是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富
    + 列族
    + 图形
        - 关系图谱

## 在分布式数据库中的CAP

* 传统的ACID
    + A：Atomicity 原子性
    + C：Consistency 一致性
    + I：Isolation 独立性
    + D：Durability 持久性

* CAP
    + C：Consistency 一致性
    + A：Availability 可用性
    + P：Partition tolerance 分区容错性

+ CAP的3进2
    + CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，$\color{red}{最多只能同时较好的满足两个}$
    + 因此，根据CAP原理将NoSQL数据库分成了满足CA原则、满足CP原则和满足AP原则三大类
    + CA：单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大
    + CP：满足一致性，分区容错性的系统，通常性能不是特别高
    + AP：满足可用性，分区容错性，通常可能对一致性要求低一些

* CAP解读
    + 由于当前的网络硬件肯定会出现延迟丢包等问题，所以，$\color{red}{分区容错性}$是必须要实现的。所以只能在$\color{blue}{一致性}$和$\color{blue}{可用性}$之间进行权衡，没有NoSQL系统能同时保证这三点
    + CA：传统Oracle数据库
    + AP：大多数网站架构的选择
    + CP：redis、MongoDB

* 一致性与可用性的抉择
    + 数据库事务一致性需求
        - 很多web实时系统并不要求严格的数据库事务，对读一致性的要求很低，有些场合对写一致性要求并不高。允许实现最终一致性
    + 数据库的写实时性和读实时性需求
        - 对关系数据库来说，插入一条数据之后立刻查询，是肯定可以读出来这条数据的，但是对于很对web应用来说，并不要求这么高的实时性。

* 原理图
![](https://qiancijun.coding.net/p/Image/d/Image/git/raw/master/Mysql/cap.jpg)

## BASE
* BASE就是为了解决关系数据库强一致性引起的问题而引起的可用性降低而提出的解决方案。BASE是对CAP中一致性和可用性权衡的结果，其来源于对大规模互联网系统分布式实践的结论，是基于CAP定理逐步演化而来的，其核心思想是即使无法做到强一致性（Strong consistency），但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性（Eventual consistency）。
    + Basically Available：基本可用
    + Soft state：软状态
    + Eventually consistent：最终一致

### 基本可用
* 分布式系统在出现不可预知故障的时候，允许损失部分可用性——但请注意，这绝不等价于系统不可用。
* 例：
  1. 响应时间上的损失：正常情况下，一个在线搜索引擎需要0.5秒内返回给用户相应的查询结果，但由于出现异常（比如系统部分机房发生断电或断网故障），查询结果的响应时间增加到了1~2秒。
  2. 功能上的损失：正常情况下，在一个电子商务网站上进行购物，消费者几乎能够顺利地完成每一笔订单，但是在一些节日大促购物高峰的时候，由于消费者的购物行为激增，为了保护购物系统的稳定性，部分消费者可能会被引导到一个降级页面。

### 软状态
* 允许系统中的数据存在中间状态，并认为该中间状态的存在不会影响系统的整体可用性，即允许系统在不同节点的数据副本之间进行数据听不的过程存在延时。

### 最终一致性
* 系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。因此，最终一致性的本质是需要系统保证最终数据能够达到一致，而不需要实时保证系统数据的强一致性。最终一致性存在以下五类主要变种。
  1. 因果一致性
    +  如果进程A在更新完某个数据项后通知了进程B，那么进程B之后对该数据项的访问都应该能够获取到进程A更新后的最新值，并且如果进程B要对该数据项进行更新操作的话，务必基于进程A更新后的最新值，即不能发生丢失更新情况。与此同时，与进程A无因果关系的进程C的数据访问则没有这样的限制。
  2. 读己之所写
    + 进程A更新一个数据项之后，它自己总是能够访问到更新过的最新值，而不会看到旧值。也就是说，对于单个数据获取者而言，其读取到的数据一定不会比自己上次写入的值旧。因此，读己之所写也可以看作是一种特殊的因果一致性。
  3. 会话一致性
    + 会话一致性将对系统数据的访问过程框定在了一个会话当中：系统能保证在同一个有效的会话中实现“读己之所写”的一致性，也就是说，执行更新操作之后，客户端能够在同一个会话中始终读取到该数据项的最新值。
  4. 单调读一致性
    + 如果一个进程从系统中读取出一个数据项的某个值后，那么系统对于该进程后续的任何数据访问都不应该返回更旧的值。
  5. 单调写一致性
    + 一个系统需要能够保证来自同一个进程的写操作被顺序地执行。

### 小结
* BASE理论面向的是大型高可用可扩展的分布式系统，和传统事务的ACID特性使相反的，它完全不同于ACID的强一致性模型，而是提出通过牺牲强一致性来获得可用性，并允许数据在一段时间内是不一致的，但最终达到一致状态。但同时，在实际的分布式场景中，不同业务单元和组件对数据一致性的要求是不同的，因此在具体的分布式系统架构设计过程中，ACID特性与BASE理论往往又会结合在一起使用。

## 分布式和集群
* 分布式系统：由多台计算机和通信的软件组件通过计算机网络连接组成。分布式系统是建立在网络之上的软件系统。正是因为软件的特性，所以分布式系统具有高度的内聚性和透明性。
    + 不同的多台服务器上面部署不同的服务模块（工程），他们之间通过Rpc/Rmi之间通信和调用，对外提供服务和组内协作
* 集群：
    + 不同的多台服务器上面部署相同的服务模块，通过分布式调度软件进行统一的调度，对外提供服务和访问

# Redis
* Redis： REmote DIctionary Server（远程字典服务器）
    + 完全开源免费，由C语言编写，遵守BSD协议，是一个高性能的（key/value）分布式内存数据库，基于内存运行并支持持久化的NoSQL数据库，也被称为数据结构服务器 
* Redis与其它三个key/value数据库相比有三个特点
    1. Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用
    2. Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储
    3. redis还支持数据的备份，即master-slave模式的数据备份

## 特点
1. 内存存储和持久化：redis支持异步将内存中的数据写到硬盘上，同时不影响继续服务
2. 取最新N个数据的操作
3. 模拟类似于HttpSession这种需要设定过期时间的功能
4. 发布、订阅消息系统
5. 定时器和计时器

## 五大数据类型
1. String（字符串）
    + String是redis最基本的类型，可以理解为于Memcached一模一样的类型，一个key对应一个value
    + string是二进制安全的，意思是redis的string可以包含任何数据类型
    + 一个redis中字符串value最多可以是512M
2. Hash（哈希，类似Java里的Map）
    + Redis hash是一个键值对集合
    + 是一个string类型的field和value的映射表，特别适合用于存储对象
3. List（列表）
    + redis列表是简单的字符串列表，按照插入顺序排序，可以添加一个元素到列表的头部或者尾部
    + 它的底层实际是个链表
4. Set（集合）
    + redis的set是string类型的无序集合，它是通过HashTable实现的
5. Zset（sorted set有序集合）

## 键（key）
1. `keys *`：查看当前库所有的键值对
2. `exists key的名字`：判断某个key是否存在
3. `move key的名称 库的下标`：移动键值对
4. `expire key 秒钟`：为给定的key设置过期时间，过期后从库中移出
5. `ttl key`：查看还有多少秒过期，-1表示永不过期，-2表示已过期
6. `type key`：查看key是什么类型

## 字符串（String）
* 单值单Value
1. `append`：拼接字符串
2. `strlen`：返回字符串长度
3. `incr/decr/incrby/decrby`：一定要是数字才能进行加减
4. `getrange`：根据范围截取字符串
5. `setrange`：根据下标来替换
6. `setex(set wuth expire)`：设置过期时间和值
7. `setnx(set if not exist)`：不会对已有的值覆盖
8. `mset/mget/msetnx`：设置/获取/设置过期时间 多个键值对

## 列表（List）
* 单值多Value
1. `lpush/rpush/lrange`：l从左边插入，r从右边插入，lrange从0到-1查看全部
2. `lpop/rpop`：l从左边弹出，r从右边弹出
3. `lindex`：按照索引获得元素
4. `llen`：长度
5. `lrem key n value`：删n个value
6. `ltrim key begin end`：截取指定范围的值后再赋值给key
7. `rpoplpush 源 目标列表`
8. `lset key index value`：设置指定索引的值
9. `linsert key before/after value 值`：在指定value前后插入一个值

## 集合（Set）
* 单值多value
1. `sadd/smembers/sismember`：添加/查看/查找
2. `scard`：获取集合里元素的个数
3. `srem key value`：删除集合中的元素
4. `srandmember key n`：随机取出n个数
5. `spop key`：随机出栈
6. `smove key1 key2：在key1里的某个值`：将key1里的某个值赋给key2
7. 差集：sdiff
8. 交集：sinter
9. 并集：sunion

## 哈希（Hash）
* KV模式不变，但V是一个键值对
1. `hset/hget/hmset/hmget/hgetall/hdel`：添加/获取/查看
``` no-highlight 
hset user id 1
hget user id
```
2. `hlen`：获取长度
3. `hexists key 在key里某个值的key`：判断是否存在
4. `hkeys/hvals key`：查看所有的键/值
5. `hincrby/hincrbyfloat`：
6. `hsetnx`

## 有序集合（Zset）
* 在set的基础上，加上一个score值
1. `zadd/zrange`：添加/查看
2. `zrangebyscore key begin end`：范围内分数的查询
    + `zrangebyscore set1 60 90`：60到90之间
    + `zrangebyscore set1 60 (90`：不在60到90之间
    + `zrangebyscore set1 60 90 limit`：开始下标 结束下标：限制个数
3. `zrem key value`值：删除元素
4. `zcard/zcount key score区间/zrank key values值`：获得下标值/zscore key 对应值，获得分数

## 解析配置文件
参数说明
redis.conf 配置项说明如下：
1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
    daemonize no
2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
    pidfile /var/run/redis.pid
3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字
    port 6379
4. 绑定的主机地址
    bind 127.0.0.1
5.当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
    timeout 300
6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
    loglevel verbose
7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null
    logfile stdout
8. 设置数据库的数量，默认数据库为0，可以使用SELECT `<dbid>`命令在连接上指定数据库id
    databases 16
9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
    save `<seconds>` `<changes>`
    Redis默认配置文件中提供了三个条件：
    save 900 1
    save 300 10
    save 60 10000
    分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。

10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
    rdbcompression yes
11. 指定本地数据库文件名，默认值为dump.rdb
    dbfilename dump.rdb
12. 指定本地数据库存放目录
    dir ./
13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
    slaveof `<masterip>` `<masterport>`
14. 当master服务设置了密码保护时，slav服务连接master的密码
    masterauth `<master-password>`
15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH `<password>`命令提供密码，默认关闭
    requirepass foobared
16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息
    maxclients 128
17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区
    maxmemory `<bytes>`
18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no
    appendonly no
19. 指定更新日志文件名，默认为appendonly.aof
      appendfilename appendonly.aof
20. 指定更新日志条件，共有3个可选值： 
    no：表示等操作系统进行数据缓存同步到磁盘（快） 
    always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 
    everysec：表示每秒同步一次（折中，默认值）
    appendfsync everysec

21. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）
      vm-enabled no
22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享
      vm-swap-file /tmp/redis.swap
23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
      vm-max-memory 0
24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值
      vm-page-size 32
25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。
      vm-pages 134217728
26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
      vm-max-threads 4
27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
    glueoutputbuf yes
28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
    hash-max-zipmap-entries 64
    hash-max-zipmap-value 512
29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）
    activerehashing yes
30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
    include /path/to/local.conf

# Redis的持久化

## RDB（Redis Database）
* 在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是讲Snapshot快照，它恢复时将快照文件直接读到内存内。
* Redis会单独创建（Fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，将持久化过程都结束了，再用这个临时文件替换上次持久化的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能，如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。
* Fork：复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等）数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程
* RDB保存的是dump.rdb文件。是整个内存压缩过的Snapshot。默认
    + 1分钟内改了1万次
    + 5分钟改了10次
    + 15分钟改了1次
* 开启RDB
    + 配置文件中默认的配置
    + save命令或者bgsave命令
    + 执行flushall命令，但是里面是空的没有意义

* 优势：
    1. 适合大规模的数据恢复
    2. 对数据完整性和一致性要求不高
* 劣势：
    1. 在一定间隔时间做一次备份，如果redis意外down，就会丢失最后一次快照后的所有修改
    2. Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑


## AOF（Append Only File）
* $\color{red}{以日志的形式来记录每个写操作}$，将redis执行过的所有写指令记录下来（读操作不记录）。只许追加文件但不可以改写文件，redis启动后会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。
* 配置：
    + appendonly：默认是NO关闭
    + appendfilename：aof文件名
    + Appendfsync：
        - Always：同步持久化，每次发生数据变更会被立即记录到磁盘，性能较差但数据完整性比较好
        - Everysec：默认推荐，异步操作，每秒记录，如果一秒内脱机，有数据丢失
        - No
    + No-appendfsync-on-rewrite：重写时是否可以运用Appendfsync，用默认No即可，保证数据安全性
    + Auto-aof-rewrite-min-size：设置重写文件的基准值（企业从3G起步）
    + Auto-aof-rewrite-percentage：设置重写的基准值
### Rewrite
* AOF采用文件追加方式，文件会越来越大，为避免出现此种情况，新增了重写机制，当AOF文件的大小超过所设定的阈值，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集，可以使用命令bgrewriteaof
* 原理：
    + AOF文件持续增长而过大时，会Fork出一条新进程来将文件重写（先写临时文件最后再rename），遍历新进程的内存中数据，每条记录有一条set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件。
* 触发机制：
    + redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发

### 优点
1. 每秒同步：always
2. 每修改同步：everysec
3. 不同步：no

### 缺点
1. 相同的数据集的数据aof文件要远大于rdb文件，恢复速度慢于rdb
2. aof运行效率要慢于rdb，每秒同步策略效率较好，不同步效率和rdb相同

## 总结
1. RDB持久化方式能够再指定的时间间隔能对数据进行快照存储
2. AOF持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以redis协议追加保存每次写的操作到文件末尾
3. Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大
4. 只做缓存：如果希望数据在服务器运行的时候存在，也可以不使用任何持久化方式
5. 同时开启两种持久化方式：先加载AOF

# 事务
* 可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，$\color{red}{按顺序地串行化执行而不会被其它命令插入，不许加塞}$。

## 常用命令
1. discard：取消事务，放弃执行事务块内的所有命令
2. exec：执行所有事务块内的命令
3. multi：标记一个事务块的开始
4. unwatch：取消watch命令对所有key的监视
5. watch key：监视一个（或多个）key，如果在事务执行之前这个key被其他命令所改动，那么事务将被打断

## 特性
1. 部分原子性
2. 悲观锁
    * 悲观锁(Pessimistic Lock), 每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁
2. 乐观锁
    * 乐观锁(Optimistic Lock), 每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量。
3. Watch
    * Watch指令， 类似乐观锁，事务提交时，如果Key的值已被别的客户端改变,比如某个list已被别的客户端push/pop过了， 整个事务队列都不会被执行
    * 通过WATCH命令在事务执行之前监控了多个Keys,倘若在WATCH之后有任何Key的值发生了变化，ExEC命令执行的事务都将被放弃，同时返回Nlmult- bulk应答以通知调用者事务执行失败
4. 单独的隔离操作:事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
5. 没有隔离级别的概念:队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际执行，也就不存在”事务内的查询要看到事务里的更新，在事务外查询不能看到”这个让人万分头痛的问题
6. 不保证原子性: redis同 一个事务中如果有一一条命令执行失败，其后的命令仍然会被执行，没有回滚

# 发布订阅
* 进程间的一种消息通信模式:发送者(pub)发送消息，订阅者(sub)接收消息。

## 命令
1. `PSUBSCRIBE patterm [patterm ...]`：订阅-个或多个符合给定模式的频道。
2. `PUBSUB subcommand [argument [argument .]`：查看订阅与发布系统状态。
3. `PUBLISH channel message`：将信息发送到指定的频道。
4. `PUNSUBSCRIBE [patterm [patterm ..]`：退订所有给定模式的频道。
5. `SUBSCRIBE channel [channel .]`：订阅给定的一个或多个频道的信息。
6. `UNSUBSCRIBE [channel [channel .]`：指退订给定的频道。

# 主从复制
* 主机数据更新后根据配置和策略，自动同步到备机的master/slaver机制，Master以写为主，Slave以读为主

## 使用
1. 配从库不配主库
2. 从库配置：slaveof 主库IP 主库端口
  + 每这-atr开之后。幅需爱重新连接。除非你配置eds.coc性
3. 修改配置文件
  + 拷贝多个redis.conf文件
  + 开启daemonize yes
  + Pid文件名字
  + 指定端口
  + Log文件名字
  + Dump.rdb名字
4. 链式复制
  + 上一个Slave可以是下一个slave的Master, slave同样可以接收其他slaves的连接和同步请求，那么该slave作为了链条中下一个的master,可以有效减轻master的写压力
  + 中途变更转向:会清除之前的数据，重新建立拷贝最新的
  + Slaveof新主库IP新主库端口
5. 取消主从复制
  + slaveof no one：使当前数据库停止与其他数据库的同步，转成主数据库

## 原理
* Slave启动成功连接到master后会发送一个sync命令，Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步
* 全量复制:slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
* 增量复制:Master继续将新的所有收集到的修改命令依次传给slave,完成同步
* 但是只要是重新连接master,一次完全同步(全量复制)将被自动执行

## 哨兵模式
* 在主从复制中，主库宕机，就需要手动配置从库来修改主库ip。而哨兵模式可以自动把一台从服务器提升至主服务器
* 配置：
  + 在Redis安装目录下有一个sentinel.conf文件，copy一份进行修改
  ``` 
  # 禁止保护模式
  protected-mode no
  # 配置监听的主服务器，这里sentinel monitor代表监控，mymaster代表服务器的名称，可以自定义，192.168.11.128代表监控的主服务器，6379代表端口
  # 2代表只有两个或两个以上的哨兵认为主服务器不可用的时候，才会进行failover操作。
  sentinel monitor mymaster 192.168.11.128 6379 2
  # sentinel author-pass定义服务的密码，mymaster是服务名称，123456是Redis服务器密码
  # sentinel auth-pass <master-name> <password>
  sentinel auth-pass mymaster 123456
  ```

## 缺点
* 复制延时：由于所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重。

# 集群
* Redis集群实现了对Redis水平扩容，即启动N个redis节点，将整个数据库分布存储在这N个节点中，每个节点存储总数居的1/N；
* Redis集群通过分区来提供一定程度的可用性，即使集群中有一部分节点失效或者无法进行通讯，集群也可以继续处理命令请求

# Jedis

## 测试联通
* Java程序连接虚拟机Redis
``` java
public static void main(String[] args) {
    Jedis jedis = new Jedis("虚拟机ip", 6379);
    System.out.println(jedis.ping());
}
```
* Linux下`ifconfig`找到ens33下inet为本机ip
* 关闭Linux虚拟机的防火墙
``` Linux
systemctl stop firewall
# 查看状态
firewall-cmd --state
```
* 将redis.conf下的bind修改为本机虚拟机ip

## 事务
``` java
public static void main(String[] args) {
    Jedis jedis = new Jedis("192.168.200.131", 6379);
    // 获取一个事务
    Transaction transaction = jedis.multi();
    transaction.set("k1", "v1");
    transaction.set("k2", "v2");
    transaction.set("k3", "v3");
    // 提交执行
    transaction.exec();
}
```

## JedisPool