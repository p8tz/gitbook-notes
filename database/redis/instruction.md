| 数据类型 | 可以存储的值           | 操作                                                         |
| -------- | ---------------------- | ------------------------------------------------------------ |
| STRING   | 字符串、整数或者浮点数 | 对整个字符串或者字符串的其中一部分执行操作 对整数和浮点数执行自增或者自减操作 |
| LIST     | 列表                   | 从两端压入或者弹出元素 对单个或者多个元素进行修剪， 只保留一个范围内的元素 |
| SET      | 无序集合               | 添加、获取、移除单个元素 检查一个元素是否存在于集合中 计算交集、并集、差集 从集合里面随机获取元素 |
| HASH     | 包含键值对的无序散列表 | 添加、获取、移除单个键值对 获取所有键值对 检查某个键是否存在 |
| ZSET     | 有序集合               | 添加、获取、删除元素 根据分值范围或者成员来获取元素 计算一个键的排名 |

## string

**`set, get, del`**

```sql
127.0.0.1:6379> set name jjz
OK
127.0.0.1:6379> get name
"jjz"
127.0.0.1:6379> del name
(integer) 1
127.0.0.1:6379> get name
(nil)
127.0.0.1:6379> del name
(integer) 0
```

**`mset, mget`**

```sql
127.0.0.1:6379> mset a 1 b 2 c 3
OK
127.0.0.1:6379> mget a b c
1) "1"
2) "2"
3) "3"
127.0.0.1:6379>
```

**`strlen, append`**

```sql
127.0.0.1:6379> set k 123
OK
127.0.0.1:6379> strlen k
(integer) 3
127.0.0.1:6379> append k 456
(integer) 6
127.0.0.1:6379> get k
"123456"
```

**`expire, pexpire`**

```sql
# 初始化时设置
127.0.0.1:6379> set x 1 ex 5  # 5秒后过期, px单位为ms
OK
127.0.0.1:6379> get x
"1"
127.0.0.1:6379> get x
(nil)

# 分开设置
127.0.0.1:6379> set x 1
OK
127.0.0.1:6379> expire x 5
(integer) 1
127.0.0.1:6379> get x
"1"
127.0.0.1:6379> get x
(nil)
```

**`incr, incrby, decr, decyby, incrbyfloat`**

```sql
127.0.0.1:6379> set num 1
OK
127.0.0.1:6379> incr num
(integer) 2
127.0.0.1:6379> get num
"2"
127.0.0.1:6379> incrby num 5
(integer) 7
127.0.0.1:6379> decr num
(integer) 6
127.0.0.1:6379> incrbyfloat num 1.5
"7.5"
```

**`setnx`**

```sql
127.0.0.1:6379> set x 1
OK
127.0.0.1:6379> setnx x 2
(integer) 0
127.0.0.1:6379> get x
"1"
```



![image-20201107150940616](https://gitee.com/p8t/picbed/raw/master/imgs/20201107160825.png)

## hash

**`hset, hget, hgetall, hdel, hmset... heln, hexists`**

```sql
127.0.0.1:6379> hset person name jjz
(integer) 1
127.0.0.1:6379> hset person age 20
(integer) 1
127.0.0.1:6379> hget person name
"jjz"
127.0.0.1:6379> hgetall person
1) "name"
2) "jjz"
3) "age"
4) "20"
127.0.0.1:6379> hdel person age
(integer) 1
127.0.0.1:6379> hget person age
(nil)
127.0.0.1:6379>
127.0.0.1:6379> hlen person
(integer) 1
127.0.0.1:6379> hexists person age
(integer) 0
```

**`hkeys, hvals`**

```sql
127.0.0.1:6379> hmset user name jjz age 20
OK
127.0.0.1:6379> hkeys user
1) "name"
2) "age"
127.0.0.1:6379> hvals user
1) "jjz"
2) "20"
```

**`hincrby, hincrbyfloat`**

## list

**`lpush, rpush, lpop, rpop, lindex, lrange, llen`**

```sql
127.0.0.1:6379> rpush list1 tencent alibaba bytedance
(integer) 3
127.0.0.1:6379> llen list1
(integer) 3
127.0.0.1:6379> lrange list1 0 2
1) "tencent"
2) "alibaba"
3) "bytedance"
127.0.0.1:6379> lrange list1 0 -1
1) "tencent"
2) "alibaba"
3) "bytedance"
127.0.0.1:6379> lindex list1 0
"tencent"
127.0.0.1:6379> lpop list1
"tencent"
127.0.0.1:6379> lindex list1 0
"alibaba"
```

**`blpop, brpop`**

```sql
session 1:						session 2:
127.0.0.1:6379> rpush l1 a      
(integer) 1
127.0.0.1:6379> rpop l1
"a"
127.0.0.1:6379> lpop l1
(nil)
								127.0.0.1:6379> blpop l1 10
								.
127.0.0.1:6379> rpush l1 x		.
(integer) 1						.
								1) "l1"
                                2) "x"
                                (9.43s)
```

**`lrem : lrem key count value`**

```sql
127.0.0.1:6379> rpush 001 a b c d e d f
(integer) 7
127.0.0.1:6379> lrem 001 1 b
(integer) 1
127.0.0.1:6379> lrange 001 0 -1
1) "a"
2) "c"
3) "d"
4) "e"
5) "d"
6) "f"
127.0.0.1:6379> lrem 001 2 d
(integer) 2
127.0.0.1:6379> lrange 001 0 -1
1) "a"
2) "c"
3) "e"
4) "f"
```

## set

**`sadd, smembers, srem, scard, sismember`**

```sql
127.0.0.1:6379> sadd companies tencent alibaba bytedance
(integer) 3
127.0.0.1:6379> smembers companies
1) "tencent"
2) "bytedance"
3) "alibaba"
127.0.0.1:6379> srem companies bytedance
(integer) 1
127.0.0.1:6379> smembers companies
1) "tencent"
2) "alibaba"
127.0.0.1:6379> scard companies
(integer) 2
127.0.0.1:6379> sismember companies tencent
(integer) 1
```

**`srandmember, spop`**

```sql
127.0.0.1:6379> sadd companies tencent alibaba bytedance
(integer) 3
127.0.0.1:6379> srandmember companies 1
1) "alibaba"
127.0.0.1:6379> scard companies
(integer) 3
127.0.0.1:6379> spop companies
"tencent"
127.0.0.1:6379> scard companies
(integer) 2
```

## zset

**`zadd, zrange, zrevrange, zrem, zcard, zcount`**

```sql
127.0.0.1:6379> zadd scores 75 a 50 b 24 c 98 d 35 e
(integer) 5
127.0.0.1:6379> zrange scores 0 -1
1) "c"
2) "e"
3) "b"
4) "a"
5) "d"
127.0.0.1:6379> zrange scores 0 -1 withscores
 1) "c"
 2) "24"
 3) "e"
 4) "35"
 5) "b"
 6) "50"
 7) "a"
 8) "75"
 9) "d"
10) "98"
127.0.0.1:6379> zrevrange scores 0 -1
1) "d"
2) "a"
3) "b"
4) "e"
5) "c"
127.0.0.1:6379> zrem scores c
(integer) 1
127.0.0.1:6379> zcard scores
(integer) 4
```

**`zrangebyscore, zrevrangebyscore, zremrangebyrank, zremrangebyscore`**

```sql
127.0.0.1:6379> zadd scores 75 a 50 b 24 c 98 d 35 e
(integer) 5
127.0.0.1:6379> zrangebyscore scores 20 50 withscores
1) "c"
2) "24"
3) "e"
4) "35"
5) "b"
6) "50"
127.0.0.1:6379> zrangebyscore scores 20 50 withscores limit 0 2
1) "c"
2) "24"
3) "e"
4) "35"
127.0.0.1:6379> zremrangebyscore scores 20 50 # 按score区间删除
(integer) 3
127.0.0.1:6379> zcard scores
(integer) 2
127.0.0.1:6379> zremrangebyrank scores 0 1 # 按排序后索引删除
(integer) 2
127.0.0.1:6379> zcard scores
(integer) 0
```

**`zrank, zrevrank, zscore, zincrby`**

```sql
127.0.0.1:6379> zadd sc 50 a
(integer) 1
127.0.0.1:6379> zrank sc a		# 查询排名
(integer) 0
127.0.0.1:6379> zscore sc a     # 查询score
"50"
127.0.0.1:6379> zincrby sc 10 a # 改变score
"60"
```

## 通用命令

### 与key相关

**`del, exists, type`**

**`expire, pexpire, expireat, ttl, pttl, persist`**

```sql
127.0.0.1:6380> set name JJZ ex 5
OK
127.0.0.1:6380> ttl name  
(integer) 2				 # 返回有效时间
127.0.0.1:6380> ttl name
(integer) -2			 # 过期或不存在返回-2
127.0.0.1:6380> set name JJZ
OK
127.0.0.1:6380> ttl name
(integer) -1			 # 永久有效返回-1
```

**`keys pattern`**

```sql
127.0.0.1:6380> set abc 1
127.0.0.1:6380> set acb 1
127.0.0.1:6380> set aedf 1
127.0.0.1:6380> set esfd 1
127.0.0.1:6380> set saed 1
127.0.0.1:6380> keys a*			# * 匹配任意字符
1) "abc"
2) "acb"
3) "aedf"
127.0.0.1:6380> keys a[ed]df	# [] 匹配其中任意一个
1) "aedf"
127.0.0.1:6380> keys sae?		# ? 匹配任意一个
1) "saed"
```

**`rename, renamenx, sort`**

```sql
127.0.0.1:6380> set name1 1
127.0.0.1:6380> set name2 2
127.0.0.1:6380> rename name1 name2
OK
127.0.0.1:6380> get name1
(nil)
127.0.0.1:6380> get name2	# 覆盖目标名
"1"
127.0.0.1:6380> flushdb
OK
127.0.0.1:6380> set name1 1
OK
127.0.0.1:6380> set name2 2
OK
127.0.0.1:6380> renamenx name1 name2 # 目标名已存在则不改
(integer) 0
127.0.0.1:6380> keys *
1) "name1"
2) "name2"

127.0.0.1:6380> rpush arr 4 3 5 1 2
(integer) 5
127.0.0.1:6380> lrange arr 0 -1
1) "4"
2) "3"
3) "5"
4) "1"
5) "2"
127.0.0.1:6380> sort arr
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
127.0.0.1:6380> lrange arr 0 -1	# 原list不变
1) "4"
2) "3"
3) "5"
4) "1"
5) "2"
```

### 与DB相关

**`select, ping`**

```sql
127.0.0.1:6380> select 1
OK
127.0.0.1:6380[1]> select 15
OK
127.0.0.1:6380[15]> select 16
(error) ERR DB index is out of range
127.0.0.1:6380[15]> select 0
OK
127.0.0.1:6380> 
```

**`move`**

```sql
127.0.0.1:6380> set name JJZ
OK
127.0.0.1:6380> select 1
OK
127.0.0.1:6380[1]> get name
(nil)
127.0.0.1:6380[1]> select 0
OK
127.0.0.1:6380> move name 1
(integer) 1
127.0.0.1:6380> get name
(nil)
127.0.0.1:6380> select 1
OK
127.0.0.1:6380[1]> get name
"JJZ"
-----------------------------------
127.0.0.1:6380[1]> set name db1
OK
127.0.0.1:6380[1]> select 0
OK
127.0.0.1:6380> set name db0
OK
127.0.0.1:6380> move name 1
(integer) 0		# 移动失败
```

**`flushdb, flushall, dbsize`**