## 一、数据结构

### 字典

redis中全局的`key / value`以及数据类型`hash, set`都是通过字典实现的。

它是一种`hash`结构，通过链表解决`hash`冲突。内部有两个`ht(hashtable)`，一个用于存储数据，一个用于扩容

#### 扩容/收缩机制

- 扩容：扩大为不小于元素数量2倍的2的幂
- 收缩：缩小为不小于元素数量的2的幂

扩容时机：当服务器没有执行`bgsave`或`bgrewriteaof`，负载因子大于等于1时扩容；执行时，负载因子大于等于5时扩容

#### 渐进式rehash

如果字典键值对过多，扩容时会消耗时间。因此redis服务器并不是一次性把`ht[0]`数据都`rehash`到`ht[1]`，而是分多次，具体为把`rehash`过程分散到每一次添加、删除、更新和查找过程，这个过程中`rehashidx`不断+1，完成后置-1

由于渐进式rehash过程中同时使用ht[0]和ht[1]，因此执行删除、查找等操作时会对两个都执行。

```c
typedef struct dict {
    dictType *type;
    void *privdata;
    // 两个hashtable
    dictht ht[2];
    // 记录rehash时的进度(索引)，不进行时为-1
    long rehashidx;
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

#### 小结

- 字典被广泛用于实现 Redis 的各种功能， 其中包括数据库和哈希键。
- Redis 中的字典使用哈希表作为底层实现， 每个字典带有两个哈希表， 一个用于平时使用， 另一个仅在进行 rehash 时使用。
- 当字典被用作数据库的底层实现， 或者哈希键的底层实现时， Redis 使用 MurmurHash2 算法来计算键的哈希值。
- 哈希表使用链地址法来解决键冲突， 被分配到同一个索引上的多个键值对会连接成一个单向链表。
- 在对哈希表进行扩展或者收缩操作时， 程序需要将现有哈希表包含的所有键值对 rehash 到新哈希表里面， 并且这个 rehash 过程并不是一次性地完成的， 而是渐进式地完成的。

### 跳跃表

跳表就是链表加多级指针，每一个节点都有一个层数，节点值就是`zset`中的`score`。图中两端灰色的部分分别为链表头和链表尾，都不记录数据。

![image-20201112173553844](https://gitee.com/p8t/picbed/raw/master/imgs/20201112173555.png)

查找元素时从最高层开始查找（图中最高层为`L3`），小于向右，大于向下。查找22过程如下

![image-20201112174318353](https://gitee.com/p8t/picbed/raw/master/imgs/20201112174319.png)

每一个节点的层数是随机生成的，生成算法的伪代码如下

```c
int level = 1
while (random() < P && level < MAXLEVEL) {
    level++
}
return level
// random() : 生成 0-1 之间的随机数
// P : 0.25
// MAXLEVEL : 32
```

因此，一个节点层数是1，2，3...的概率分别为$$0.25^0 * 0.75, 0.25 ^ 1 * 0.75, 0.25^2 * 0.75$$... 且最大层数为32

#### 定义

```c
#define ZSKIPLIST_MAXLEVEL 32
#define ZSKIPLIST_P 0.25

typedef struct zskiplistNode {
    robj *obj;
    // 排序字段
    double score;
    // 后退指针, 只有一个, 因此回退一次只能退一个
    struct zskiplistNode *backward;
    // level
    struct zskiplistLevel {
        // 前进指针, 每层都有一个
        struct zskiplistNode *forward;
        // 跨度, 两个节点之间的距离
        unsigned int span;
    } level[];
} zskiplistNode;

typedef struct zskiplist {
    // 头尾节点
	struct zskiplistNode *header, *tail;
    // 链表长度
    unsigned long length;
    // 最大层数
    int level;
} zskiplist;
```

**`span`**：这个字段用于快速计算`rank`。查询一个数据的`rank`时，先根据`dict`查询`score`，然后在跳表中查询对应的节点，查询过程中走过的`span`之和就是`rank`（倒序，链表长度减去它就是正序）。同理，我们知道利用跳表可以很方便的按`score`区间查询数据；如果想按`rank`区间查询，只需要不停计算`span`即可。

#### zset

- 当数据较少时，`zset`是由一个`ziplist`来实现的。
- 当数据较多时，`zset`是由`dict + skiplist`来实现的。`dict`用来查询数据到分数(`score`)的对应关系，而`skiplist`用来根据分数查询数据（可能是范围查找）。

#### 与平衡树、哈希表的比较

- `skiplist`和各种平衡树（如AVL、红黑树等）的元素是有序排列的，而哈希表不是有序的。因此，在哈希表上只能做单个key的查找，不适宜做范围查找。
- 在做**范围查找**的时候，平衡树比`skiplist`操作要复杂。在平衡树上，我们找到指定范围的小值之后，还需要以中序遍历的顺序继续寻找其它不超过大值的节点。如果不对平衡树进行一定的改造，这里的中序遍历并不容易实现。而在`skiplist`上进行范围查找就非常简单，只需要在找到小值之后，对第1层链表进行若干步的遍历就可以实现。
- 平衡树的插入和删除操作可能引发子树的调整，逻辑复杂，而`skiplist`的插入和删除只需要修改相邻节点的指针，操作简单又快速。
- 从**内存占用**上来说，`skiplist`比平衡树更灵活一些。一般来说，平衡树每个节点包含2个指针（分别指向左右子树），而skiplist每个节点包含的指针数目平均为1/(1-p)，具体取决于参数p的大小。如果像Redis里的实现一样，取p=1/4，那么平均每个节点包含1.33个指针，比平衡树更有优势。
- 查找单个key，`skiplist`和平衡树的时间复杂度都为O(log n)，大体相当；而哈希表在保持较低的哈希值冲突概率的前提下，查找时间复杂度接近O(1)，性能更高一些。
- 从**算法实现**难度上来比较，`skiplist`比平衡树要简单得多。

因此

- `zscore`的查询，是根据数据查`score`，不是由`skiplist`来提供的，而是由`dict`来提供的
- `zrange`的查询，是根据排名查数据，由`skiplist`来提供。
- `zrank`是先在`dict`中由数据查到分数，再拿分数到`skiplist`中去查找，查到后也同时获得了排名。

## 二、持久化

### RDB

相当于快照，把**当前所有数据**都记录下来生成**`dump.rdb`**文件

#### 手动持久化

通过**`save / bgsave`**命令生成**`dump.rdb`**文件，文件位置可通过配置文件进行配置

```bash
# datafile root directory
dir /data
# rdb config
dbfilename dump-6380.rdb	# 文件绝对路径为 /data/dump-6380.rdb
rdbcompression yes
rdbchecksum yes
```

**区别**

`save`命令会阻塞当前`redis`服务器，直到`RDB`过程完成为止

![image-20201108211459086](https://gitee.com/p8t/picbed/raw/master/imgs/20201108211500.png)

`bgsave`会调用`fork`函数生成子进程，进行`RDB`文件的生成

![image-20201108211714944](https://gitee.com/p8t/picbed/raw/master/imgs/20201108211715.png)

#### 自动持久化

通过配置文件的方式实现服务器自动调用`bgsave`指令

```bash
# set触发(无论数据有没有真正改变), get / del 不触发
save 900 1		# 900秒内有1个key变化触发bgsave
save 300 10
save 60 10000
```

#### 优缺点

优点

- RDB是一个紧凑压缩的二进制文件，存储效率较高
- RDB内部存储的是redis在某个**时间点**的数据快照，非常适合用于数据备份，全量复制等场景
- RDB恢复数据的速度要比AOF快很多
- 应用：服务器中每X小时执行bgsave备份，并将RDB文件拷贝到远程机器中，用于灾难恢复。

缺点

- RDB方式无论是执行指令还是利用配置，无法做到实时持久化，具有较大的可能性丢失数据
- bgsave指令每次运行要执行fork操作创建子进程，要牺牲掉一些性能
- Redis的众多版本中未进行RDB文件格式的版本统一，有可能出现各版本服务之间数据格式无法兼容现象

### AOF

RDB存储的弊端

- 存储数据量较大，效率较低
- 基于快照思想，每次读写都是全部数据，当数据量巨大时，效率非常低
- 大数据量下的IO性能较低
- 基于fork创建子进程，内存产生额外消耗
- 宕机带来的数据丢失风险

解决思路

- **不写全数据，仅记录部分数据**
- 降低区分数据是否改变的难度，改记录数据为**记录操作过程**
- 对所有操作均进行记录，排除丢失数据的风险

#### AOF概念

- AOF(append only file)持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中命令达到恢复数据的目的。与RDB相比可以简单描述为改记录数据为记录数据产生的过程
- AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式

#### 写数据过程

![image-20201108215409562](https://gitee.com/p8t/picbed/raw/master/imgs/20201108215410.png)

#### 写数据策略

AOF持久化并不是直接落盘，而是先放在一个缓冲区，对应落盘策略有如下三种

- `always`：每次写入操作均同步到AOF文件中，**数据零误差，性能较低**
- `everysec`：每秒将缓冲区中的指令同步到AOF文件中，**数据准确性较高，性能较高**，在系统突然宕机的情况下丢失1秒内的数据
- `no`：由操作系统控制每次同步到AOF文件的周期，**整体过程不可控**

实际上，由于操作系统的原因本该写入磁盘的数据，并不一定真正写入，它会先保存在内核的缓冲区，等满了或超过时限才真正写入。

#### 开启持久化

```bash
appendonly yes | no	# 默认为 no
appendfsync always | everysec | no # 落盘策略
dir /data
appendfilename appendonly-6380.aof # AOF文件位置
```

#### 重写

由于`aof`是基于记录操作过程的，因此可能会记录大量无用的操作，比如连续`set`同一个`key`，则只有最后一个`key`有效，这样不仅消耗性能还占磁盘。考虑到这种情况，redis提供了`aof`重写功能。

##### 重写作用

- 降低磁盘占用量，提高磁盘利用率
- 提高持久化效率，降低持久化写时间，提高IO性能
- 降低数据恢复用时，提高数据恢复效率

##### 重写规则

- 进程内已超时的数据不再写入文件
- 忽略无效指令，重写时使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令
- 对同一数据的多条写命令合并为一条命令。如`lpush list1 a、lpush list1 b、 lpush list1 c `可以转化为：`lpush list1 a b c`，为防止数据量过大造成客户端缓冲区溢出，对`list、set、hash、zset`等类型，每条指令最多写入64个元素

##### 手动重写

通过`bgrewriteaof`命令

![image-20201108221610032](https://gitee.com/p8t/picbed/raw/master/imgs/20201108221611.png)

##### 自动重写

```bash
# 自动重写触发条件设置
auto-aof-rewrite-min-size size # 缓冲区数据达到size时重写
auto-aof-rewrite-percentage percentage
# 自动重写触发比对参数 (运行指令info Persistence获取具体信息)
aof_current_size	# 缓冲区待写入数据的大小
aof_base_size		# 
```

触发自动重写条件（满足其一即可）

$$aof\_current\_size > auto-aof-rewrite-min-size$$

$$\frac{aof\_current\_size - aof\_base\_size}{aof\_base\_size} >= auto-aof-rewrite-percentage$$

##### 重写流程

对于开启了重写的情况，会有一个额外的aof重写缓存区，用于重写进程使用

![image-20201108223028850](https://gitee.com/p8t/picbed/raw/master/imgs/20201108223029.png)

![image-20201108223040758](https://gitee.com/p8t/picbed/raw/master/imgs/20201108223041.png)

### 选择

如果服务器开启了AOF持久化功能，则使用AOF，否则使用RDB。如果没有RDB文件则不载入

### 对比

![image-20201108223416294](https://gitee.com/p8t/picbed/raw/master/imgs/20201108223417.png)

### 小结

对数据非常敏感，建议使用默认的AOF持久化方案

- AOF持久化策略使用everysecond，每秒钟fsync一次。该策略redis仍可以保持很好的处理性能，当出 现问题时，最多丢失0-1秒内的数据。
- 注意：由于AOF文件存储体积较大，且恢复速度较慢

数据呈现阶段有效性，建议使用RDB持久化方案

- 数据可以良好的做到阶段内无丢失（该阶段是开发者或运维人员手工维护的），且恢复速度较快，阶段 点数据恢复通常采用RDB方案
- 注意：利用RDB实现紧凑的数据持久化会使Redis降的很低

综合比对

- RDB与AOF的选择实际上是在做一种权衡，每种都有利有弊
- 如不能承受数分钟以内的数据丢失，对业务数据非常敏感，选用AOF 
- 如能承受数分钟以内的数据丢失，且追求大数据集的恢复速度，选用RDB
- 灾难恢复选用RDB
- 双保险策略，同时开启 RDB 和 AOF，重启后，Redis优先使用 AOF 来恢复数据，降低丢失数据的量

## 三、事务

### 命令

```bash
multi	# 开启事务
...
exec	# 执行事务

discard	# 回滚
```

![image-20201109085608460](https://gitee.com/p8t/picbed/raw/master/imgs/20201109085610.png)

### 锁

通过`watch`监控某个`key`，如果事务提交的时候发现`key`变化了则`discard`。取消监控用`unwatch`（全部取消）

```bash
# session 1							# session 2
127.0.0.1:6380> set num 1		
OK
127.0.0.1:6380> watch num
OK
127.0.0.1:6380> multi
OK
127.0.0.1:6380> incr num
QUEUED
127.0.0.1:6380> set age 20
QUEUED
									127.0.0.1:6380> incr num
									(integer) 2
127.0.0.1:6380> exec
(nil)
127.0.0.1:6380> get num
"2"
127.0.0.1:6380> get age
(nil)
127.0.0.1:6380> 
```

## 四、分布式锁

### setnx

通过`setnx`实现，这里的`setnx`不单指`setnx key value`这条命令。

由于`set`可以加`nx`参数且有原子性，因此一般用`set key value [EX seconds|PX milliseconds|KEEPTTL] [NX|XX]`

具体实现如下

```bash
127.0.0.1:6380> set LOCK UUID px 200 nx

# 伪代码
redis.set(LOCK, UUID, px, 200, nx)
try {
    ...
} finally {
    if (UUID.equals(redis.get('LOCK')) {
    	redis.del('LOCK');
    }
}
```

- 设置200ms后自动释放是防止因服务器宕机等情况造成死锁
- 释放锁的时候需要先检查是不是自己的锁，具体用锁对应的value来判断。之所以出现准备释放锁却发现不是自己当初申请的锁的情况，是因为设置了锁有效期，可能自己执行时间太长，锁已经自动释放被别人抢去了

实际上释放锁还是有问题：`get `和 `del`不原子，问题如下

```bash
# thread A							thread B
# 锁在A手里，并且A准备释放			     # 抢锁	
get									# 抢锁
# 失去CPU							   # 抢锁
# 超时, 锁自动释放						# 抢到锁

del	# 把B的锁误释放					 # 锁被A误释放
```

因此需要保证`get del`执行的原子性，具体通过嵌入`lua`脚本实现

### redisson

`JUC`的`redis`版本。

redisson解决了业务未执行完，锁过期的问题（`redisson`功能远比这多，不是说只解决了这个问题）。通过看门狗机制，自动延期锁的时效。

`redisson`在加锁成功后，会注册一个定时任务监听这个锁，每隔10秒就去查看这个锁，如果还持有锁，就对过期时间进行续期，重置为默认过期时间。默认过期时间为30秒。那么问题来了，这不就回到原始情况了么：一直续下去和不过期有什么区别？

### redlock

红锁是官方提出的一个分布式锁算法，`redisson`就用到了这种算法

## 五、删除策略

### 数据特征

Redis是一种内存级数据库，所有数据均存放在内存中，内存中的数据可以通过TTL指令获取其状态

- XX ：具有时效性的数据
- -1 ：永久有效的数据
- -2 ：**已经过期的数据** 或 被删除的数据 或 未定义的数据

对于**已经过期的数据**并不是立马删除，有对应的删除策略

### 删除策略

#### 1、定时删除

数据到期后立刻删除

- 优点：节约内存，到时就删除，快速释放掉不必要的内存占用
- 缺点：CPU压力很大，无论CPU此时负载量多高，均占用CPU，会影响redis服务器响应时间和指令吞吐量

#### 2、惰性删除

数据到达过期时间，不做处理。等下次访问该数据时，发现已过期，删除

- 优点：节约CPU性能，发现必须删除的时候才删除
- 缺点：内存压力很大，出现长期占用内存的数据

#### 3、定期删除

- 每秒执行server.hz（默认为10）次serverCron()操作
- 在这个操作中会对所有的库进行轮询访问
- 每次访问会对该库中所有有时效性的key进行随机挑选（每次挑选持续时长`250ms/server.hz`），挑选出W个（通过配置文件配置）
- 对这W个key中过期的进行删除，如果删除的个数超过25%，则重新对该库执行这个过程，否则去下一个库
- 变量`current_db`记录当前轮询到了哪个数据库

### 小结

redis内部使用的是惰性删除和定期删除

## 六、数据逐出（淘汰）算法

删除策略针对的是expire数据。

逐出算法则针对内存不够用的情况，通过删除一些数据（即使是永久性数据）来解决。

### 配置

**最大可用内存**

```bash
maxmemory
```

占用物理内存的比例，默认值为0，表示不限制，即最大吃满内存

**每次选取待删除数据的个数**

```bash
maxmemory-samples
```

不是把选的这些都删了，而是它们执行逐出算法。避免了对全库数据执行逐出算法。这个值太小需要执行多次，太大消耗性能

**删除策略**

```bash
maxmemory-policy
```

对上面选出来的数据执行的逐出算法

### 逐出算法

1、检测易失数据（sever.db[i].expires）

- `volatile-lru`：最近最少使用
- `volatile-lfu`：最近使用次数最少
- `volatile-ttl`：快要过期
- `volatile-random`：随机删

2、检测所有数据（server.db[i].dict）

- `allkeys-lru`：最近最少使用
- `allkeys-lfu`：最近使用次数最少
- `allkeys-random`：随机删

3、放弃数据逐出

- no-enviction：禁止数据逐出（redis4.0默认），会引发OOM

```bash
maxmemory-policy volatile-lru
```

## 七、配置文件

```bash
daemonize yes | no
bind 127.0.0.1
port 6379
requirepass pwd
databases 16
protected-mode yes | no
dir /data
loglevel debug | verbose | notice | warning
logfile xxx.log
maxclients 0 # 支持最大客户端连接数 0表示随便连
timeout 300  # 连接最大闲置时间, 超过关闭连接
```

## 八、高级数据类型

### Bitmaps

用于记录**有没有**的情况

比如一年365天哪些天下雨了，只需要365个bit，下雨标为1，否则标为0即可

对应到redis中的类型为string，它提供了操作二进制的一些接口

### HyperLogLog

用于基数统计，即去重后的元素个数，不会存储数据，只记录个数。**占用空间少，会有误差，0.81%**

### GEO

计算距离（现实距离）

## 九、主从复制

### 作用

- 读写分离：master写、slave读，提高服务器的读写负载能力
- 负载均衡：基于主从结构，配合读写分离，由slave分担master负载，并根据需求的变化，改变slave的数 量，通过多个从节点分担数据读取负载，大大提高Redis服务器并发量与数据吞吐量
- 故障恢复：当master出现问题时，由slave提供服务，实现快速的故障恢复
- 数据冗余：实现数据热备份，是持久化之外的一种数据冗余方式
- 高可用基石：基于主从复制，构建哨兵模式与集群，实现Redis的高可用方案

### 搭建

通过配置文件

```bash
# 在slave配置文件中加上
slaveof master_ip master_port

# 如果master有密码加上
masterauth master_pwd
```

### 过程

建立连接阶段：`slave`连接`master`

数据同步阶段：全部复制，部分复制

命令传播阶段：

#### 1、建立连接阶段

![image-20201110095537681](https://gitee.com/p8t/picbed/raw/master/imgs/20201110095539.png)

1. `slave`对`master`发送建立连接请求
2. `master`收到后响应一下`slave`
3. `slave`收到响应后保存`master`的IP和端口号，创建`socket`用于数据传输
4. `slave`需要周期性向`master`发送`ping`，以防`master`宕机了它还不知道
5. `slave`身份验证
6. `slave`把自己端口号发给`master`，这样`master`就能通过`slave`监听的端口号发送数据

配置完成后，`slave`就只能读了

```bash
127.0.0.1:6382> set name p8t
(error) READONLY You can't write against a read only replica.
```

#### 2、数据同步阶段

![image-20201110110542994](https://gitee.com/p8t/picbed/raw/master/imgs/20201110110544.png)

1. `master`和`slave`刚刚建立连接后首先进行一次**全量复制**
2. `master`生成一个`RDB`文件发送给`slave`，在生成`RDB`文件过程中产生的数据放在一个复制缓冲区中
3. `slave`把`RDB`文件数据写完后告诉`master`，然后`master`再把刚刚缓冲区中的数据以指令的方式发给`slave`
4. `slave`先**重写**，再恢复数据，这一过程为**部分复制**（增量复制）

注意事项

复制缓冲区大小有限，因此如果进行全量复制周期太长，会导致缓冲区前面的数据被覆盖，造成数据丢失。这样就必须**再次进行全量复制**

#### 3、命令传播阶段

命令传播阶段用于数据实时同步，以及处理突发情况

比如出现了断网现象

- 网络闪断闪连：忽略
- 短时间网络中断：部分复制
- 长时间网络中断：全量复制

**部分复制**的三个核心要素

- 服务器的运行 `id`（`run id`）：每一台服务器每次运行的身份识别码，同一个服务器每次运行生成的`id`都不一样。用于`master`识别`slave`
- 主服务器的复制积压缓冲区：由于一个`master`会有多个`slave`，每个slave同步的数据不一样，因此需要一个缓冲区记录已经执行的命令（以AOF的方式记录），这是一个`FIFO`队列，满了最前面的会被覆盖
- 主从服务器的复制偏移量：用于标记不同`slave`已经发送的不同位置

## 十、哨兵

对于主从架构，如果`master`宕机，则需要把它踢掉，然后从`slave`中重新选一个当`master`，哨兵就是干这事的。

哨兵的作用就是主从切换，分为以下三个阶段

- 监控阶段：用于同步各个节点的状态信息。获取其它`sentinel`状态，获取`master`状态，获取`slave`状态

  1. 第一个`sentinel`进来后与`master`交换信息，通过`master`又能获取`slave`信息

  2. 下一个`sentinel`进来后与`master`交换信息，发现有其它`sentinel`连过，就会找到它们交换信息

     ![image-20201110225201730](https://gitee.com/p8t/picbed/raw/master/imgs/20201110225202.png)

- 通知阶段：验证各节点是否正常工作

  1. `sentinel`会间断的向`master/slave`发送信息以确认状态是否正常

  2. 并且会把反馈在`sentinel`之间交换

     ![image-20201110225525509](https://gitee.com/p8t/picbed/raw/master/imgs/20201110225526.png)

- 故障转移阶段

  1. 当`master`宕机后，该消息会在`sentinel`之间传开，如果超过一半（可配置）的`sentinel`确认master`真的宕机了，则开始故障转移。如果是sentinel自己宕机，则该消息传不开。
  2. 然后从`sentinel`之间选出一个代表（它们之间相互通信，一个`sentinel`会把自己的一票投给最先和它通信的`sentinel`，如果有打平则重来）去选出下一任`master`。
  3. 选举规则
     - 不在线的`pass`
     - 响应慢的`pass`
     - 与原master断开时间久（最后一次通信到现在的时间）的`pass`
     - 优先原则
       - 优先级
       - `offset`
       - `runid`（最后看谁`runid`小）
  4. 选完以后通知其它`slave`
  5. 如果原`master`回来了则把它变成`slave`

## 十一、集群

## 十二、缓存问题

### 缓存预热

redis服务器刚启动对外提供服务时，里面没有数据，这就相当于`mysql`裸奔。因此redis需要预先加载部分数据后再对外提供服务。

加载的数据取决于它的热度。至于热度需要平时统计

### 缓存雪崩

**短时间内，大量key过期，导致请求全部压到数据库。**

场景：一个抢购活动，抢购商品都被集中放在缓存中。抢购结束后，缓存中的商品集中过期，但是大量用户仍然在访问导致查询落在数据库上，数据库承受不住崩了，重启数据库依然崩

解决方法

- 过期时间采用固定时间+随机值的形式，错峰过期
- 超热数据使用永久`key`
- 加锁（治标不治本）
- 定期维护，对即将过期的数据进行访问量分析，如果访问量过大可以考虑延时

### 缓存击穿

类似于缓存雪崩：某个`key`过期，然后大量访问该`key`

场景：微博热搜，`redis`中`key`失效，大量请求落入数据库

解决方法

- 设置`key`永不过期
- 加锁

加锁伪代码

```java
String value = redis.get(key)
if (value != null)
    return value;

lock();
value = mysql.get(key);
redis.set(key, value);
unlock();
return value;
```

### 缓存穿透

redis大面积未命中，数据库也查不出记录（大量请求不存在的数据）。

场景：非法攻击，攻击者针对一个数据库**没有的数据**进行大量访问，目的就是弄崩数据库。比如数据库里只有1，2，3，4，5这五条记录，然后有人不停的查询6

解决方法

- 参数校验，拦截一些不可能的参数（比如我的数据库`id`都是正的，那么可以把负的全拦截下来），避免直接查询数据库
- 把这些无效访问的`key`也缓存下来，`value`设为`null`（设置个过期时间），这样不会冲击数据库，相当于在`redis`这拦截下来。但是对于大量不同的无效请求，这个方法不行，因为要缓存的太多了。
- 布隆过滤器

**布隆过滤器**

功能

- 判断一个元素**可能在**集合中
- 判断一个元素**一定不在**集合中

`hashtable`可以高效的实现查询元素是否存在的功能，但布隆过滤器**占用空间更小**。

布隆过滤器的实现需要一个**二进制序列**和**若干个hash函数**。对于一个key，通过这若干个hash函数计算出它在二进制序列上不同位置的若干个点，把这些点标为1。

因此查询的时候看它对应的这些位置是否都为1，只要有一个0，那么肯定不存在这个key。但即使全为1，也只能判断可能存在这个key，因为它对应位置的1有可能被其它key标记了。

## 参考文章

[Redis为什么用跳表而不用平衡树？](https://zhuanlan.zhihu.com/p/23370124)













