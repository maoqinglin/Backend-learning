# 一、NoSQL入门简介

## 1、3V+3高

大数据时代的3V：海量（Volume）， 多样（Variety），实时（Velocity）

互联网需求的3高：高并发（），高可扩展（），高性能（）



## 2、数据如何存放

以女装、女包为例

1、商品基本信息：名称、价格，出厂日期，生产厂商等**（冷数据）**

关系型数据库：MySQL/Oracle

2、商品描述、详情、评价信息（多文字类）

多文字信息描述类，IO读写性能变差：文档数据库 MongDB 中

3、商品的图片

分布式的文件系统中：淘宝自己的TFS、Google的GFS、Hadoop的HDFS

4、商品的关键字

搜索引擎，淘宝内用：ISearch

**5、商品的波段性的热点高频信息（热数据）**

内存数据库：Tair、Redis、 Memcache

6、商品的交易、价格计算、积分累计

支付接口，支付宝



总结大型互联网应用的难点和解决方案：

难点：数据类型多样性，数据源多样性和变化重构，数据源改造而数据服务平台不需要大面积重构

解决办法：

**EAI和统一数据平台服务层**

UDSL：映射、API、热点缓存

 

## 3、NoSQL数据模型简介

以一个电商客户、订单、订购、地址模型来对比关系型数据库和非关系型数据库

1）、关系型数据库：ER图

![](.\Images\redis\电商订单ER图.png)

2）、NoSQL如何设计：**聚合模型**

​	1.K - V键值

​	2.BSON：Binary JSON 内嵌的文档对象和数组对象

​	3.列族

​	4.图形

3）、问题和难点

为什么上述的情况可以用聚合模型来处理：

​	1.高并发的操作是不太建议由关联查询的，

​	2.互联网公司用冗余数据来避免关联查询

​	**3.分布式事务是支持不了太多的并发的** 



## 4、NoSQL数据库的四大分类

​	1.K - V键值：新浪（BerkeleyDB+redis），美团（redis+tair），阿里、百度：memcache+redis

​	2.文档数据库（bson格式比较多）：CouchDB、MongoDB（ 介于关系型与非关系型数据库）

​	3.列存储数据库：HBase，分布式数据库

​	4.图关系数据库：放的是关系比如：朋友圈社交网络、广告推荐系统、。专注于构建关系图谱

​		Neo4J，InfoGrid



## 5、分布式数据库中CAP+BASE

### 1）、CAP

**强一致性（Consistency），可用性（Availability），分区容错性（Partition tolerance）**



CAP理论的核心：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求

**最多只能同时较好的满足两个。**

因此，根据CAP原理将 NoSQL数据库分成类满足 CA 原则、满足 CP 原则和满足 AP原则三大类：

CA：单点集群，满足一致性，可用性的系统，通常性能不是特别高。

CP：满足一致性，分区容忍性的系统，通常性能不是特别高

**AP：满足可用性，分区容忍性的系统，通常可能对一致性要求低一些**	

![](G:\03_Markdown\Images\redis\经典CAP.png)

CAP理论就是说在分布式存储系统中，最多只能实现上面的两点。而由于当前的网络硬件肯定会出现延迟丢包等问题，所以

分区容忍性是我们必须要实现的

CA：传统Oracle

CP：Redis、MongoDB

**AP：大多数网站架构的选择**



### 2）、传统的ACID

原子性（ Atomicity）、一致性（Consistency）、独立性（Isolation）、持久性（Durability）



### 3）、BASE

BASE就是为了解决关系数据库强一致性引起的问题而引起的可用性降低而提出的解决方案

BASE其实是下面三个术语的缩写：

​	基本可用（Basically Available）

​	软状态（Soft state）

​	**最终一致（Eventually consistent）**

它的思想是通过让系统**放松对某一时刻数据一致性**的要求来**换取整体伸缩性和性能上改观**。缘由就在于大型系统往往由于地域分布和极高性能的要求，不可能采用分布式事务来完成这些指标，要想获得这些指标，我们必须采用另外一种方式来完成，这里BASE就是解决这个问题的办法



### 4）、分布式和集群

分布式：不同的多台服务器上部署不同的服务模块（工程），它们之间通过 RPC/RMI 通信和调用，对外提供服务和组内协作

集群：不同多台服务器上面部署相同的服务模块，通过分布式调度软件进行统一的调度，对外提供服务和访问



# 二、Redis入门

## 1、是什么

Redis：Remote Dictionary Server（远程字典服务器）

高性能的（key/value）**分布式内存数据库**，基于内存运行，**并支持持久化**的NoSQL数据库

三个特点：

​	1.Redis 支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用

​	2.Redis不仅仅支持简单的 key-value 类型的数据，同时还提供 list、set、zset，hash等数据结构

​	**3.Redis支持数据的备份，即 master-slave模式的数据备份**



## 2、能干嘛

1）内存存储和持久化：Redis支持异步将内存中的数据写到硬盘上，同时不影响继续服务

2）取最新N个数据的操作，如：可以将最新的 10 条评论的 ID 放在 Redis 的 list 集合里面

3）模拟类似于 HttpSession 这种需要设定过期时间的功能

4）发布、订阅消息系统

5）定时器、计数器



# 三、Redis使用

## 1、安装

```bash
tar -zxvf redis-5.0.5.tar.gz

make

# 修改自定义的配置文件（不改动出厂的默认配置）
mkdir /myredis
cp redis.conf /myredis/
vi /myredis/redis.conf 
# 修改如下，并保存；	(shift+$ 到行尾)
daemonize yes


# 启动自定义配置
cd /usr/local/bin/
redis-server /myredis/redis.conf 

# 查看进程
[root@node-1 bin]# ps -ef|grep redis
root      6474     1  0 00:22 ?        00:00:00 redis-server 127.0.0.1:6379
root      6479  6443  0 00:28 pts/1    00:00:00 grep --color=auto redis

# 启动 redis 客户端，不需要使用 -p，因为服务端已经占用了 6379 端口
[root@node-1 bin]# redis-cli

# 测试连通
127.0.0.1:6379>  ping
PONG

[root@node-1 bin]# ps -ef|grep redis
root      6491     1  0 00:34 ?        00:00:00 redis-server 127.0.0.1:6379
root      6497  2170  0 00:35 pts/0    00:00:00 redis-cli
root      6499  6443  0 00:35 pts/1    00:00:00 grep --color=auto redis


# 退出客户端
exit
```



## 2、重要知识

### 1）单进程

单进程模型来处理客户端的请求。对读写等事件的响应是通过 epoll 函数的包装做到的。Redis的实际处理速度完全依靠主进程的执行效率

Epoll 是 Linux 内核为处理大批量文件描述符而做了改进的 epoll，是 Linux下多路复用接口 select/poll的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率

### 2）常用命令

默认 16 个数据库，类似数组下标从零开始，初始默认使用 0 号库

```bash
# 查看当前数据库的 key 的数量
127.0.0.1:6379> DBSIZE 
(integer) 5

# 获取所有 key
127.0.0.1:6379> keys *
1) "key:__rand_int__"
2) "mylist"
3) "name"
4) "counter:__rand_int__"
5) "myset:__rand_int__"

# 模糊查询 m开头的key
127.0.0.1:6379> keys m*
1) "mylist"
2) "myset:__rand_int__"

127.0.0.1:6379> keys mylis?
1) "mylist"

# 切换数据库
127.0.0.1:6379[1]> SELECT 1

# 清空当前库
127.0.0.1:6379[1]> FLUSHDB 
OK
127.0.0.1:6379[1]> DBSIZE
(integer) 0

127.0.0.1:6379[1]> SELECT 0
OK
127.0.0.1:6379> DBSIZE
(integer) 5

# 清空所有库
127.0.0.1:6379> FLUSHALL
OK
127.0.0.1:6379> DBSIZE
(integer) 0
```

统一密码管理，16个库都是同样密码，要么都 ok要么一个也连接不上

Redis索引都是从 0 开始



# 四、Redis数据类型

## 1、key

1）判断 key 是否存在，移动 key 到指定的库

```bash
keys *
# exists key : 判断某个 key 是否存在
127.0.0.1:6379> keys *
1) "k2"
2) "k3"
3) "k1"
127.0.0.1:6379> EXISTS k3
(integer) 1
127.0.0.1:6379> EXISTS k4
(integer) 0

# move key db：移动 key 到指定的 db，当前db就没有此key了
127.0.0.1:6379> move k3 1
(integer) 1
127.0.0.1:6379> keys *
1) "k2"
2) "k1"

127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> keys *
1) "k3"
```



2）、设置过期时间，剩余时间 ttl （time to leave）

```bash
# expire key 秒钟：为给定的 key 设置过期时间
127.0.0.1:6379> EXPIRE k2 10
(integer) 1

# ttl key 查看还有多少秒过期，-1表示永不过期，-2表示已过期
127.0.0.1:6379> ttl k2
(integer) 5
127.0.0.1:6379> get k2
(nil)
127.0.0.1:6379> keys *
1) "k1"

127.0.0.1:6379> ttl k1
(integer) -1
```



3）查看 key 对应的 value 的类型

```bash
# type key：查看 key 的类型
127.0.0.1:6379> LPUSH mylist 1 2 3 4 5
(integer) 5
127.0.0.1:6379> KEYS *
1) "mylist"
2) "k1"
127.0.0.1:6379> get mylist
(error) WRONGTYPE Operation against a key holding the wrong kind of value

127.0.0.1:6379> LRANGE mylist 0 -1
1) "5"
2) "4"
3) "3"
4) "2"
5) "1"
127.0.0.1:6379> type mylist
list
```



## 2、string

string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象

string类型是Redis最基本的数据类型，一个Redis中字符串value最多可以是 512M

1）set、get、del、append、strlen

```bash
# SET key value
127.0.0.1:6379> set k1 v1
OK
# GET key
127.0.0.1:6379> get k1
"v1"

# APPEND key value	
127.0.0.1:6379> append k1 name
(integer) 6
127.0.0.1:6379> get k1
"v1name"

# STRLEN key
127.0.0.1:6379> strlen k1
(integer) 6
```

2）Incr、decr、incrby、decrby：必须是数字才能进行加减

```bash
127.0.0.1:6379> set k2 2
OK
127.0.0.1:6379> set k3 v3
OK
# INCR key
127.0.0.1:6379> incr k2		
(integer) 3
127.0.0.1:6379> INCR k2
(integer) 4

# 步进，INCRBY key increment
127.0.0.1:6379> INCRBY k2 2
(integer) 6
127.0.0.1:6379> INCRBY k2 2
(integer) 8
```

```bash
# DECR key
127.0.0.1:6379> DECR k2
(integer) 7
127.0.0.1:6379> DECR k2
(integer) 6

# DECRBY key decrement
127.0.0.1:6379> DECRBY k2 3
(integer) 3
127.0.0.1:6379> DECRBY k2 3
(integer) 0
```

```bash
127.0.0.1:6379> get k3
"v3"
127.0.0.1:6379> INCR k3
(error) ERR value is not an integer or out of range
```



3）getrange：获取指定区间范围内的值，类似 between....and

```bash
127.0.0.1:6379> get k1
"v1name"

# GETRANGE key start end
127.0.0.1:6379> GETRANGE k1 0 -1
"v1name"
127.0.0.1:6379> GETRANGE k1 0 3
"v1na"

# SETRANGE key offset value
127.0.0.1:6379> SETRANGE k1 3 abc
(integer) 6
127.0.0.1:6379> get k1
"v1nabc"

# offset 必须大于0
127.0.0.1:6379> SETRANGE k1 -1 xiao
(error) ERR offset is out of range

# SETRANGE key offset value；value为几位替换几位
127.0.0.1:6379> get k1
"abc"
127.0.0.1:6379> SETRANGE k1 2 456
(integer) 5
127.0.0.1:6379> get k1
"ab456"
```



4）setex（set with expire）键秒值，setnx（set if not exist）

```bash
# SETEX key seconds value
127.0.0.1:6379> setex k4 10 v4
OK
# TTL key
127.0.0.1:6379> ttl k4
(integer) 7
127.0.0.1:6379> get k4
"v4"

127.0.0.1:6379> ttl k4
(integer) -2
127.0.0.1:6379> get k4
(nil)
```

```bash
# SETNX key value
127.0.0.1:6379> SETNX k1 aaaa
(integer) 0
127.0.0.1:6379> get k1
"ab456"
127.0.0.1:6379> SETNX k22 222
(integer) 1

```



**5）mset、mget、msetnx**

```bash
# MSET key value [key value ...]
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3
OK
127.0.0.1:6379> keys *
1) "k2"
2) "k3"
3) "k22"
4) "k1"

# MGET key [key ...]
127.0.0.1:6379> mget k1 k2 k3
1) "v1"
2) "v2"
3) "v3"

# 部分存在，不会覆盖成功
127.0.0.1:6379> KEYS *
1) "k2"
2) "k3"
3) "k22"
4) "k1"

# MSETNX key value [key value ...]
127.0.0.1:6379> MSETNX k3 v3 k4 v4
(integer) 0
```



## 3、List

Redis 列表是简单的字符串列表，按照顺序排序。一可以添加一个元素到列表的头部（左边）或者尾部（右边）。它的底层实际是个链表

1）lpush、rpush、lrange

```bash
# LPUSH key value [value ...]
127.0.0.1:6379> LPUSH list 1 2 3 4 5
(integer) 5
127.0.0.1:6379> LRANGE list 0 -1
1) "5"
2) "4"
3) "3"
4) "2"
5) "1"

127.0.0.1:6379> RPUSH list2 1 2 3 4 5
(integer) 5
127.0.0.1:6379> LRANGE list2 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
```

2）lpop、rpop

```bash
127.0.0.1:6379> LRANGE list 0 -1
1) "5"
2) "4"
3) "3"
4) "2"
5) "1"
127.0.0.1:6379> lpop list
"5"
127.0.0.1:6379> rpop list
"1"

127.0.0.1:6379> LRANGE list 0 -1
1) "4"
2) "3"
3) "2"
```

3）lindex

```bash
127.0.0.1:6379> LRANGE list 0 -1
1) "4"
2) "3"
3) "2"

127.0.0.1:6379> LINDEX list 0
"4"
127.0.0.1:6379> LINDEX list 3
(nil)
```

4）llen

```bash
127.0.0.1:6379> LLEN list
(integer) 3
```



5）lrem key ：删 N个value

```ba
127.0.0.1:6379> RPUSH list2 aaa bbb ccc
(integer) 3
127.0.0.1:6379> LRANGE list2 0 -1
1) "aaa"
2) "bbb"
3) "ccc"
127.0.0.1:6379> LREM list2 2 b
(integer) 0
127.0.0.1:6379> LREM list2 1 b
(integer) 0

# 必须删除存在的值，不能是值的一部分
127.0.0.1:6379> LREM list2 3 bbb
(integer) 1
```



## 4、Set

Redis 的set是string类型的无序集合。它是通过 HashTable 实现的

1）sadd、smembers、sismember

```bash
# SADD key member [member ...]
127.0.0.1:6379> SADD set01 1 1 2 2 3 3
(integer) 3
# SMEMBERS key
127.0.0.1:6379> SMEMBERS set01
1) "1"
2) "2"
3) "3"

# SISMEMBER key member
127.0.0.1:6379> SISMEMBER set01 1
(integer) 1
127.0.0.1:6379> SISMEMBER set01 a
(integer) 0

127.0.0.1:6379> SADD set01 4
(integer) 1
127.0.0.1:6379> SADD set01 4
(integer) 0
```



2）scard：获取集合里面的元素个数

```bash
# SCARD key
127.0.0.1:6379> SCARD set01
(integer) 4
```



3）srem	key	value：删除集合中元素 

```bash
127.0.0.1:6379> SMEMBERS set01
1) "1"
2) "2"
3) "3"
4) "4"
# SREM key member [member ...]
127.0.0.1:6379> SREM set01 1
(integer) 1
127.0.0.1:6379> SMEMBERS set01
1) "2"
2) "3"
3) "4"
```



**4）srandmember	key	某个整数（随机出几个数）**

```bash
# SRANDMEMBER key [count]
127.0.0.1:6379> SRANDMEMBER set02 3
1) "1"
2) "3"
3) "7"

127.0.0.1:6379> SRANDMEMBER set02 3
1) "2"
2) "5"
3) "7"
```



5）spop	key：随机出栈

```bash
127.0.0.1:6379> SMEMBERS set01
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
# spop key [count]
127.0.0.1:6379> SPOP set01
"6"
```

6）smove	key1	key2	key1中的value：**作用是将 key1 里的 value** 赋给 key2 

```bash
127.0.0.1:6379> SMEMBERS set01
1) "2"
2) "3"
3) "b"
4) "a"
127.0.0.1:6379> SMEMBERS set02
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"

# SMOVE source destination member
127.0.0.1:6379> SMOVE set01 set02 a
(integer) 1

127.0.0.1:6379> SMEMBERS set01
1) "2"
2) "3"
3) "b"
127.0.0.1:6379> SMEMBERS set02
1) "2"
2) "a"
3) "4"
4) "3"
5) "1"
6) "5"
```



6）数学集合，sdiff（差集）、sinter（交集）、sunion（并集）

```bash
127.0.0.1:6379> SMEMBERS set01
1) "2"
2) "3"
3) "b"
127.0.0.1:6379> SMEMBERS set02
1) "2"
2) "a"
3) "4"
4) "3"
5) "1"
6) "5"

# SDIFF key [key ...]
127.0.0.1:6379> SDIFF set01 set02	# 差集，在set01中，但不在set02中
1) "b"

# SINTER key [key ...]
127.0.0.1:6379> SINTER set01 set02  # 交集
1) "2"
2) "3"

# SUNION key [key ...]
127.0.0.1:6379> SUNION set01 set02	# 并集
1) "2"
2) "b"
3) "a"
4) "4"
5) "1"
6) "3"
7) "5"
```



## 5、hash：K - V模式不变，但 V 是一个价值对

键值对集合，是一个string类型的field 和 value 映射表，hash 特别适合用于存储对象

**1）hset、hget、hmset、hmget、hgetall、hdel**

```bash
# HSET key field value
127.0.0.1:6379> hset user id 11
(integer) 1

# HGET key field
127.0.0.1:6379> hget user id
"11"
127.0.0.1:6379> hset user name xiaoyun
(integer) 1
127.0.0.1:6379> hget user name
"xiaoyun"

# HMSET key field value [field value ...]
127.0.0.1:6379> hmset customer id 11 name xiaolin age 20
OK

# HMGET key field [field ...]
127.0.0.1:6379> hmget customer id name age
1) "11"
2) "xiaolin"
3) "20"

# HGETALL key
127.0.0.1:6379> hgetall customer
1) "id"
2) "11"
3) "name"
4) "xiaolin"
5) "age"
6) "20"

# HDEL key field [field ...]
127.0.0.1:6379> HDEL user name
(integer) 1
```



2）hlen

```bash
# HLEN key
127.0.0.1:6379> hlen user
(integer) 1
127.0.0.1:6379> hlen customer
(integer) 3
```



3）hexists	key

```bash
# HEXISTS key field 
127.0.0.1:6379> HEXISTS customer id
(integer) 1
127.0.0.1:6379> HEXISTS customer email
(integer) 0
```



**4）hkeys、hvalues**

```bash
# HKEYS key
127.0.0.1:6379> HKEYS customer
1) "id"
2) "name"
3) "age"

# HVALS key
127.0.0.1:6379> HVALS customer
1) "11"
2) "xiaolin"
3) "20"
```



5）hincrby、hincrbyfloat

```bash
# HINCRBY key field increment
127.0.0.1:6379> HINCRBY customer age 2
(integer) 22
127.0.0.1:6379> HINCRBY customer age 2
(integer) 24

127.0.0.1:6379> hset customer score 90.5
(integer) 1

# HINCRBYFLOAT key field increment
127.0.0.1:6379> HINCRBYFLOAT customer score 0.5
"91"
127.0.0.1:6379> HINCRBYFLOAT customer score 0.5
"91.5"
```



6）hsetnx

```bash
# HSETNX key field value
127.0.0.1:6379> HSETNX customer age 100
(integer) 0
127.0.0.1:6379> HSETNX customer email xiaoyun@250.com
(integer) 1
```



## 6、Zset有序集合

Redis zset 和 set 一样也是string类型元素的集合，且不允许重复的成员

不同的是每个元素都会关联一个 double类型的分数

区别：

在 set 基础上，加一个 score 值

之前是 set  k1 v1  v2  v3

现在 zset 是 k1 score1 v1 score2  v2 



Redis 正式通过分数来为集合中的成员进行从小到大的排序。zset 的成员是唯一的，但分数（score）却可以重复



1）、zadd、zrange

```bash
# ZADD key [NX|XX] [CH] [INCR] score member [score member ..
127.0.0.1:6379> zadd zset01 60 v1 70 v2 80 v3 90 v4 100 v5
(integer) 5

# ZRANGE key start stop [WITHSCORES]
127.0.0.1:6379> ZRANGE zset01 0 -1
1) "v1"
2) "v2"
3) "v3"
4) "v4"
5) "v5"

127.0.0.1:6379> ZRANGE zset01 0 -1 withscores
 1) "v1"
 2) "60"
 3) "v2"
 4) "70"
 5) "v3"
 6) "80"
 7) "v4"
 8) "90"
 9) "v5"
10) "100"
```



2）zrangebyscore 	key	startscore	endscore

```bash
# ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
127.0.0.1:6379> ZRANGEBYSCORE zset01 60 90
1) "v1"
2) "v2"
3) "v3"
4) "v4"

# 不包含 [60,90)
127.0.0.1:6379> ZRANGEBYSCORE zset01 60 (90
1) "v1"
2) "v2"
3) "v3"
# 不包含 (60,90)
127.0.0.1:6379> ZRANGEBYSCORE zset01 (60 (90
1) "v2"
2) "v3"

# 限制，可以实现分页
127.0.0.1:6379> ZRANGEBYSCORE zset01 60 90 limit 2 2
1) "v3"
2) "v4"

```



3）zcard、zcount、zrank

```bash
# ZCARD key
127.0.0.1:6379> ZCARD zset01
(integer) 4

# ZCOUNT key min max
127.0.0.1:6379> ZCOUNT zset01 60 80
(integer) 3

# ZRANK key member 获得下标索引
127.0.0.1:6379> ZRANK zset01 v4
(integer) 3

# ZSCORE key member 获得score
127.0.0.1:6379> ZSCORE zset01 v4
"90"
```



4）zrevrank，zrevrange，ZREVRANGEBYSCORE

```bash
#  ZREVRANK key member
127.0.0.1:6379> ZREVRANK zset01 v4
(integer) 0

# ZREVRANGE key start stop [WITHSCORES]
127.0.0.1:6379> ZREVRANGE zset01 0 -1
1) "v4"
2) "v3"
3) "v2"
4) "v1"

# ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
# 注意逆序是从大（max）到小（min）
127.0.0.1:6379> ZREVRANGEBYSCORE zset01 90 60
1) "v4"
2) "v3"
3) "v2"
4) "v1"
```



# 五、解析配置文件

## 1、常见配置

1）、Redis 默认不是以守护进程的方式运行，可以通过该配置项修改，使用 yes 启用守护进程

​	daemonize	no

2）、当 Redis 以守护进程方式运行时， Redis 默认会把 pid 写入 /var/run/redis.pid 文件，可以通过 pidfile 指定

​	pidfile	/var/run/redis.pid

3）、指定 Redis 监听端口，默认端口为 6379

​	port	6379

4）、绑定的主机地址

​	bind	127.0.0.1

5）、当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能

​	timeout	300

6）、指定日志级别，Redis共支持四个级别：debug、verbose、notice、warning

​	loglevel	notice

7）、日志记录方式，默认为标准输出，如果配置 Redis 为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给 /dev/null

​	logfile stdout



11）、指定本地数据库文件名，默认值为 dump.rdb

​	dbfilename	dump.rdb

12）、指定本地数据库存放目录

​	dir	./



15）、设置 Redis 连接密码，如果配置了连接密码，客户端在连接 Redis 时需要通过 AUTH <password> 命令提供密码，默认关闭

​	 requirepass	foobared

16）、设置同一时间最大客户端连接数，默认无限制，Redis 可以同时打开的客户端连接数为 Redis 进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示无限制。

​	maxclients	128

17）、指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的 key，当此方法处理后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis 新的 vm 机制，会把 key 存放到内存，value 会存放在 swap 区

​	maxmemory	<bytes>



## 2、SnapShot

1）save	seconds	count

RDB是整个内存的压缩过的 SnapShot，RDB的数据结构，可以配置复合的快照触发条件，默认

**1 分钟内改了 1 万次或**

**5 分钟内改了 10 次或**

**15 分钟内改了 1 次**

2）禁用

如果想禁用 RDB 持久化的策略，只要不设置任何 save 指令，或者给 save 传入一个空字符串参数也可以

save	""



3）、stop-writes-on-bgsave-error

如果配置成no，表示你不在乎数据不一致或者有其他的手段发现和控制



4）、rdbcompression

对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用 LZF 算法进行压缩。如果你不想消耗 CPU 来进行压缩的话，可以设置为关闭此功能

​	rdbcompression	yes

5）、rdbchecksum

在存储快照后，还可以让 Redis 使用 CRC64算来来进行数据校验，但是这样做会增加大约 10% 的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能

​	rdbchecksum	yes



## 3、Limit

1） CLIENTS

maxclients 10000

2）MEMORY MANAGENT

maxmemory  <bytes>

**3）MAXMEMORY  POLICY**

volatile-lru：使用 LRU 算法一处 key，只对设置了过期时间的键

allkeys-lru：使用 LRU 算法移除 key

volatile-lfu：Least Frequently Used——简称LFU，意为最不经常使用

allkeys-lfu：

volatile-random：在过期集合中移除随机的 key，只对设置了过期时间的键

allkeys-random：移除随机的 key

volatile-ttl：移除那些 TTL 值最小的 key，即那些最近要过期的 key

noeviction：默认，永不过期，生产环境禁用



## 4、安全

默认是没有密码的，使用Linux自身的安全策略

```bash
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) ""
127.0.0.1:6379> ping
PONG

# config set requirepass 设置密码
127.0.0.1:6379> config set requirepass 123456
OK

127.0.0.1:6379> ping
(error) NOAUTH Authentication required.

# AUTH password 使用任何命令前，验证密码
127.0.0.1:6379> AUTH 123456
OK
127.0.0.1:6379> ping
PONG

# 当前 log 目录
127.0.0.1:6379> config get dir
1) "dir"
2) "/usr/local/bin"
```



# 六、Redis持久化

## 1、RDB

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是 SnapShot 快照，它恢复时将快照文件直接读到内存里

Redis 会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何 IO 操作的，这就确保了极高的性能，如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比 AOF 方式更加的高效，RDB 的缺点是最后一次持久化的数据可能丢失。

fork：作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等）数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程



### 1）如何触发 RDB 快照

1）配置文件中默认的快照配置，冷拷贝后重新使用；可以 cp	dump.rdb	dump_new.rdb



2）命令 save 或者 bgsave

save：只管保存，其它不管，全部阻塞

bgsave：Redis 会在后台异步进行快照操作，快照同时还可以响应客户端请求。可以通过 lastsave 命令获取最后一次成功执行快照的时间

3）执行 flushall 命令，也会产生 dump.rdb 文件，但里面是空的，无意义

```bash
# 在哪个目录启动 redis，就在哪个目录生成文件
[root@node-1 myredis]# redis-cli 
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2

127.0.0.1:6379> config get dir
1) "dir"
2) "/myredis"

[root@node-1 bin]# ll /myredis/
total 68
-rw-r--r--. 1 root root   169 Sep 24 02:31 dump.rdb
-rw-r--r--. 1 root root 61798 Sep 24 02:28 redis.conf

# 备份
[root@node-1 myredis]# cp dump.rdb dump_bk.rdb 
[root@node-1 myredis]# ll
total 72
-rw-r--r--. 1 root root   169 Sep 24 02:34 dump_bk.rdb
-rw-r--r--. 1 root root   169 Sep 24 02:31 dump.rdb
-rw-r--r--. 1 root root 61798 Sep 24 02:28 redis.conf

# 谨慎操作，会立即 save ，导致清空 dump.rdb
127.0.0.1:6379> FLUSHALL
OK
127.0.0.1:6379> SHUTDOWN
not connected> exit

# 重启服务
[root@node-1 myredis]# redis-server /myredis/redis.conf
[root@node-1 myredis]# redis-cli
# 数据被清空了
127.0.0.1:6379> KEYS *
(empty list or set)

# 对比查看 flushall 之后dump.rdb修改时间
[root@node-1 myredis]# ll
total 72
-rw-r--r--. 1 root root   169 Sep 24 02:34 dump_bk.rdb
-rw-r--r--. 1 root root   169 Sep 24 02:31 dump.rdb
-rw-r--r--. 1 root root 61798 Sep 24 02:28 redis.conf

# flushall之后，更新了 dump.rdb 时间
[root@node-1 myredis]# ll
total 72
-rw-r--r--. 1 root root   169 Sep 24 02:34 dump_bk.rdb
-rw-r--r--. 1 root root    92 Sep 24 02:38 dump.rdb
-rw-r--r--. 1 root root 61798 Sep 24 02:28 redis.conf
```



### 2）如何恢复

将备份文件（dump.rdb）移动到 Redis安装目录并启动服务即可；config	get	dir 获取目录

**生产使用 Redis 服务器与 Redis持久化备份的服务器绝对不能死同一台机器**

```bash
[root@node-1 myredis]# rm -rf dump.rdb
[root@node-1 myredis]# cp dump_bk.rdb dump.rdb

# 重启 Redis服务，重新恢复数据到内存
127.0.0.1:6379> KEYS *
 1) "k4"
 2) "k6"
 3) "k3"
 4) "k2"
 5) "k10"
 6) "k8"
 7) "k7"
 8) "k5"
 9) "k9"
10) "k1"
```



### **3）优势**

适合大规模的数据恢复，对数据完整性和一致性要求不高

### 4）劣势

在一定间隔时间做一次备份，所以如果 Redis 意外 down 掉的话，就会丢失最后一次快照后的所有修改

Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑

### 5）如何停止

动态所有停止 RDB保存规则的方法：

redis-cli config set save ""



### 6）总结

优点：

- RDB 是一个非常紧凑的文件
- RDB 在保存 RDB 文件时父进程唯一需要做的就是 fork 出一个子进程，接下来的工作全部由子进程来做，父进程不需要再做其它 IO 操作，所以 RDB 持久化方式可以最大化 Redis 的性能
- 与 AOF 相比，在恢复大的数据集的时候，RDB 方式会更快一些

缺点：

- 数据丢失风险大
- RDB 需要经常 fork 子进程来保存数据集到硬盘上，当数据集比较大的时候，fork的过程是非常耗时的，可能会导致 Redis 在一些毫秒级不能响应客户端请求



## 2、AOF

以日志的形式来记录每个写操作，将 Redis 执行过的所有写指令记录下来（读操作不记录），只许追加文件但不可以改写文件，Redis 启动之初会读取该文件重新构建数据，换言之，Redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

**AOF保存的是 appendonly.aof，默认未开启**

​	**appendonly	no**

### 1）AOF 启动

**修改 redis_aof.conf 配置文件，开启 AOF**

**appendonly	yes**

将有数据的 AOF 文件复制一份保存到对应目录（config get dir）（备份的机器上，非生产主机）

正常恢复：重启 Redis 然后重新加载

```bash
# 启动服务器
[root@node-1 bin]# redis-server /myredis/redis_aof.conf
[root@node-1 bin]# redis-cli

# 写入数据
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> set k3 v3
OK
# SHUTDOWN 会生成 dump.rdb
127.0.0.1:6379> SHUTDOWN
not connected> exit

-rw-r--r--. 1 root root     110 Sep 24 04:12 appendonly.aof
-rw-r--r--. 1 root root     118 Sep 24 04:12 dump.rdb
-rwxr-xr-x. 1 root root 4365336 Sep 21 00:07 redis-benchmark
-rwxr-xr-x. 1 root root 8115386 Sep 21 00:07 redis-check-aof
-rwxr-xr-x. 1 root root 8115386 Sep 21 00:07 redis-check-rdb
-rwxr-xr-x. 1 root root 4805688 Sep 21 00:07 redis-cli
lrwxrwxrwx. 1 root root      12 Sep 21 00:07 redis-sentinel -> redis-server
-rwxr-xr-x. 1 root root 8115386 Sep 21 00:07 redis-server
drwxr-xr-x. 2 root root       6 Sep 17 03:50 xl
-rwxr-xr-x. 1 root root     611 Sep 17 03:48 xsync.sh

# 重启服务器，恢复数据，数据正常
127.0.0.1:6379> keys *
1) "k3"
2) "k1"
3) "k2"

# 如果执行 flushall
127.0.0.1:6379> FLUSHALL
OK
127.0.0.1:6379> SHUTDOWN
not connected> exit
     
# 重启服务器，数据被清空
127.0.0.1:6379> keys *
(empty list or set)

# 原因： vi appendonly.aof，发现 flushall 被写入了文件末尾，相当于写的数据全被清空
	 22 set
     23 $2
     24 k3
     25 $2
     26 v3
     27 *2
     28 $6
     29 SELECT
     30 $1
     31 0
     32 *1
     33 $8
     34 FLUSHALL

# 可以手动编辑 vi appendonly.aof，去掉末尾的FLUSHALL 命令；重启服务器，就可以恢复数据了
```



### 2）AOF 异常修复

1. 启动：设置 yes
2. 备份被写坏的 AOF 文件
3. 修复：使用 redis-check-aof --fix 修复 appendonly.aof 文件
4. 恢复：重启 Redis 然后重新加载

```bash
# 模拟损坏
29 SELECT
30 $1
31 0
32 addsfasdfaxiao;;;123465

# 损坏后，无法连接服务器了
[root@node-1 bin]# redis-cli 
Could not connect to Redis at 127.0.0.1:6379: Connection refused

# 使用 redis-check-aof --fix 修复 AOF 文件
[root@node-1 bin]# redis-check-aof --fix appendonly.aof 
0x              85: Expected prefix '*', got: 'a'
AOF analyzed: size=160, ok_up_to=133, diff=27
This will shrink the AOF from 160 bytes, with 27 bytes, to 133 bytes
Continue? [y/N]: y
Successfully truncated AOF

# 重启服务器，数据自动恢复
127.0.0.1:6379> keys *
1) "k3"
2) "k2"
3) "k1"
```

### 3）同步策略

appenfsync

always：同步持久化每次发生数据变更会被立即记录到磁盘，性能较差，但数据完整性较好

**everysec：出厂默认推荐，异步操作，每秒记录；如果一秒内宕机，有数据丢失**

No

```bash
# redis_aof.conf 配置

726 # If unsure, use "everysec".
727 
728 # appendfsync always
729 appendfsync everysec
730 # appendfsync no
```



### 4）rewrite：瘦身减肥

AOF 采用文件追加方式，文件会越来越大，为避免出现此种情况，新增了重写机制；当 AOF 文件的大小超过所设定的阈值时，Redis就会启动 AOF 文件的内容压缩，只保留可以恢复数据的最小指令集，可以使用 bgrewriteaof

**重写原理：**

​	AOF 文件持续增长而过大时，会 fork 出一条新进程来将文件重写（也是先写临时文件最后再 rename），遍历新进程的内存中数据，每条记录有一条 set 语句。重写 AOF 文件的操作，并没有读取旧的 AOF 文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的 AOF 文件，这点和快照有点类似

**触发机制：**

​	Redis 会记录上次重写时的 AOF 大小，默认配置是当 AOF 文件大小是上次 rewrite 后大小的一倍且文件大于 64M 时触发

```bash
770 auto-aof-rewrite-percentage 100
771 auto-aof-rewrite-min-size 64mb
```



### 5）RDB 与 AOF 比较

RDB 与 AOF 可以并存，**优先加载 AOF**，恢复数据

### 6）优劣

**优势：**

立即同步：appendfsync always，同步持久化，每次发生数据变更会被立即记录到磁盘；性能较差，但数据完整性较高

每秒同步：appendfsync everysec，异步操作，每秒记录；如果一秒内宕机，有数据丢失

不同步：appendfsync no



**劣势：**

相同数据集的数据而言 AOF 文件要远大于 rdb 文件，恢复速度慢于 rdb

AOF 运行效率要慢于 rdb，每秒同步策略效率较好，不同效率和 rdb 相同



## 3、总结

1）RDB 持久化方式能够在指定的时间间隔对你的数据进行快照存储

2）AOF持久化记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以 Redis 协议最佳保存每次写的操作到文件末尾， Redis还能对 AOF文件进行后台重写，使得 AOF 文件的体积不至于过大

3）只做缓存：如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化方式

4）同时开启两种持久化方式：

在这种情况下，**当 Redis 重启的时候会优先加载 AOF 文件来恢复原始的数据**，因为在通常情况下 AOF 文件保存的数据集要比 RDB 文件保存的数据集要完整；

RDB的数据不实时，同时使用两者时服务器重启也只会找 AOF 文件。那要不要只使用 AOF 呢？

作者建议不要：

因为 RDB 更适合用于备份数据库（AOF在不断变化不好备份），快速重启，而且不会有 AOF 可能潜在的 bug，留着作为一个万一的手段



## 4、性能建议

因为 RDB 文件只用作后备用途，建议只在 Slave 上持久化 RDB 文件，而且只要 15 分钟备份一次就够了，只保留 save 900 1 这条规则。

如果 Enable AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只 load 自己的 AOF 文件就可以了。代价是带来了持续的 IO，二是 AOF rewrite的最后将 rewrite 过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少 AOF rewrite的频率，AOF重写的基础大小默认值 64M 太小了，可以设到5G以上。默认超过原大小 100%大小时重写可以改到适当的数值

如果不Enable AOF，紧靠Master-Slave Replication实现高可用性也可以。能省掉一大笔 IO 也减少了 rewrite时带来的系统波动。代价是如果 Master/Slave同时挂掉，会丢失十几分钟的数据，启动脚本也要比较两个 Master/Slave中的RDB文件，载入较新的那个。新浪微博就选用了这种架构



# 七、事务

执行一连串的 Redis 命令

3个阶段

**开启（multi） ---》 入队（QUEUED） ---》 执行（exec）**

## 1、基本使用

1）没有隔离级别的概念：队列中的命令没有提交之前都不会实际的被执行

2）**不保证原子性：redis 同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚**

```bash
# 开启事务
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED

# 执行事务
127.0.0.1:6379> EXEC
1) OK
2) OK
3) "v2"
4) OK


127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 vvv 
QUEUED
127.0.0.1:6379> set k2 222
QUEUED

# 放弃事务
127.0.0.1:6379> DISCARD
OK
127.0.0.1:6379> get k2
"v2"

# 事务的原子性，命令加入队列的过程中，只要有一个失败，执行命令全部失败；
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k2 22
QUEUED
127.0.0.1:6379> getk4
(error) ERR unknown command `getk4`, with args beginning with: 
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> EXEC
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379> get k4
(nil)

# 命令加入队列的过程中，没有失败，如果执行过程失败，只影响当前命令的执行，其它命令正常执行
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> INCR k1   # k1 ： v2，真正执行 INCR 操作会失败
QUEUED
127.0.0.1:6379> set k2 22
QUEUED
127.0.0.1:6379> set k3 33
QUEUED
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> get k4
QUEUED
127.0.0.1:6379> EXEC
1) (error) ERR value is not an integer or out of range  # 只影响当前命令的执行，其它不受影响
2) OK
3) OK
4) OK
5) "v4"
```

## 2、watch

1）乐观锁，类似行锁

每条记录有个 version：

每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量

**乐观锁策略：提交版本必须大于记录当前版本才能执行更新**

2）悲观锁，类似表锁

3）Watch

监视一个（或多个key），如果在事务执行之前这个（或这些）key被其它命令锁改动，那么事务将被打断。

```bash
127.0.0.1:6379> set balance 100
OK
127.0.0.1:6379> set debt 0
OK
127.0.0.1:6379> WATCH balance
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> DECRBY balance 20
QUEUED
127.0.0.1:6379> incrby debt 20
QUEUED
127.0.0.1:6379> EXEC
1) (integer) 80
2) (integer) 20

# 先开启监听
127.0.0.1:6379> WATCH balance
OK

# balance 被别的用户修改
127.0.0.1:6379> set balance 800
OK

# 开启监听后，开启事务
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> DECRBY balance 20
QUEUED
127.0.0.1:6379> inCRBY debt 20
QUEUED
# 监听过程中，如果数据被修改，执行事务失败
127.0.0.1:6379> EXEC
(nil)
127.0.0.1:6379> get balance
"800"
```

如果监听到事务执行失败，必须重新获取缓存最新数据，重新执行一遍事务

4）一旦执行了 exec ，之前加的监控锁都会被取消掉



## 3、总结

1）Watch指令，类似乐观锁，事务提交时，如果 key 的值已被别的客户端改变，比如某个 list 已被别的客户端 push/pop 过了，整个事务队列都不会被执行

2）通过Watch 命令在事务执行之前监控了多个 keys，倘若在 **Watch之后又任何 key 的值发生了变化**，**exec 命令执行的事务都将被放弃**，同时返回 nulmulti-bulk 应答通知调用者事务执行失败



# 八、发布和订阅

```bash
# 
127.0.0.1:6379> SUBSCRIBE c1 c2 c3
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "c1"
3) (integer) 1
1) "subscribe"
2) "c2"
3) (integer) 2
1) "subscribe"
2) "c3"
3) (integer) 3
# 订阅者接收到的消息
1) "message"
2) "c1"
3) "xiaoyun"

# PUBLISH channel message  发布消息
127.0.0.1:6379> PUBLISH c1 xiaoyun
(integer) 1

# PSUBSCRIBE pattern [pattern ...]  订阅多个消息
127.0.0.1:6379> PSUBSCRIBE new*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "new*"
3) (integer) 1
# 消息1
1) "pmessage"
2) "new*"
3) "new1"
4) "xiaoying"
# 消息2
1) "pmessage"
2) "new*"
3) "new2"
4) "xiaoyingzi"

# 发布多个消息
127.0.0.1:6379> PUBLISH new1 xiaoying
(integer) 1
127.0.0.1:6379> PUBLISH new2 xiaoyingzi
(integer) 1
```



# 九、主从复制

主机数据更新后根据配置和策略，自动同步到备机的 Master/Slave 机制，Master以写为主，Slave以读为主

## 1、基本使用

**配置从（库）不配置主（库）**

从库配置：Slaveof 	**Master** IP	**Master** 端口

**Slave 每次与  Master 断开之后，都需要重新连接，除非你配置到 redis.conf 文件**

```bash
# 拷贝多个 redis.conf 文件
-rw-r--r--. 1 root root 61809 Sep 25 03:40 redis6379.conf
-rw-r--r--. 1 root root 61809 Sep 25 03:42 redis6380.conf
-rw-r--r--. 1 root root 61809 Sep 25 03:44 redis6381.conf
# 开启 daemonize yes
# pid 文件名字
pidfile /var/run/redis6380.pid
# 指定端口
port 6380
# log 文件名字
logfile "6380.log"
# dump.rdb名字
dbfilename dump6380.rdb

# 启动客户端时使用 -p 指定端口
[root@node-1 bin]# redis-server /myredis/redis6379.conf 
[root@node-1 bin]# redis-cli -p 6379

# SLAVEOF host port	：开启主从复制，只需要配置从机
# 从机80连接主机
127.0.0.1:6380> SLAVEOF 127.0.0.1 6379
OK

# 从机81连接主机
127.0.0.1:6381> SLAVEOF 127.0.0.1 6379
OK

# 主机开始写数据
127.0.0.1:6379> set k4 v4
OK

# 从机80
127.0.0.1:6380> get k4
"v4"
127.0.0.1:6380> get k1
"v1"

# 从机81
127.0.0.1:6381> get k4
"v4"
127.0.0.1:6381> get k2
"v2"

# 主机状态
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6380,state=online,offset=500,lag=1
slave1:ip=127.0.0.1,port=6381,state=online,offset=500,lag=0
master_replid:fbc1af0ae1f56b840ae09070bcb864ce7531dbe7
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:500
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:500

# 从机状态
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:10
master_sync_in_progress:0
slave_repl_offset:542
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:fbc1af0ae1f56b840ae09070bcb864ce7531dbe7
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:542
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:542
```

## 2、常用3招

### 1）一主儿仆

1）主机与从机同时写数据

**只有 Master 才能 write 数据**

```bash
# 主机79
127.0.0.1:6379> set k6 v6
OK

# 从机80
127.0.0.1:6380> set k6 v6
(error) READONLY You can't write against a read only replica.

# 从机81
127.0.0.1:6381> set k6 v6
(error) READONLY You can't write against a read only replica.
```



2）主机宕机

**从机原地待命**

```bash
# 主机79 退出
127.0.0.1:6379> SHUTDOWN
not connected> EXit
[root@node-1 bin]# 

# 从机80、从机81 原地待命
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:1015
master_link_down_since_seconds:24
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:fbc1af0ae1f56b840ae09070bcb864ce7531dbe7
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1015
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1015
```



3）主机重启

```bash
# 主机重新回来了，正常运转
127.0.0.1:6379> KEYS *
1) "k3"
2) "k1"
3) "k6"
4) "k4"
5) "k2"
6) "k5"
127.0.0.1:6379> set k7 v7
OK

# 从机80
127.0.0.1:6380> get k7
"v7"

# 从机81
127.0.0.1:6381> get k7
"v7"
```



4）从机宕机

**其它的主机和从机运转正常，不受影响**

```bash
# 80从机退出
127.0.0.1:6380> SHUTDOWN
not connected> exit

# 79主机
127.0.0.1:6379> set k8 v8
OK

# 81从机
127.0.0.1:6381> get k8
"v8"
```



5）从机重新启动

```bash
# 80从机重新启动
[root@node-1 bin]# redis-server /myredis/redis6380.conf 
[root@node-1 bin]# redis-cli -p 6380
127.0.0.1:6380> info repliaction
127.0.0.1:6380> info replication
# Replication
role:master
connected_slaves:0
master_replid:363bc81e439ed9c62cead50588879d4da1e2b8a9
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# 查看79主机状态，仍然只有 81从机连接
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6381,state=online,offset=767,lag=0
master_replid:13ceb3aa11fd92fa4bb5ea643f31a7db9f24d46c
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:767
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:767

# 79主机
127.0.0.1:6379> get k8
"v8"

# 80从机
127.0.0.1:6380> get k8
(nil)
```



### 2）薪火相传

1）上一个 Slave可以是下一次 Slave 的 Master，Slave同样可以接收其它 Slaves 的连接和同步请求，那么该 Slave 作为了链条中下一个的 Master，可以有效减轻 Master 的写压力

2）中途变更转向：会清除之前的数据，重新建立拷贝最新的

3）Slaveof	主机IP	主机端口

```bash
# 79主机
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6381,state=online,offset=7739,lag=1
slave1:ip=127.0.0.1,port=6380,state=online,offset=7739,lag=1
master_replid:13ceb3aa11fd92fa4bb5ea643f31a7db9f24d46c
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:7739
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:7739
127.0.0.1:6379>

# 81从机连接80
127.0.0.1:6381> SLAVEOF 127.0.0.1 6380
OK

# 79主机，只有一个80从机
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=7809,lag=0
master_replid:13ceb3aa11fd92fa4bb5ea643f31a7db9f24d46c
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:7809
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:7809

# 80从机，是79的从机，同时又是81的主机
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:4
master_sync_in_progress:0
slave_repl_offset:7935
slave_priority:100
slave_read_only:1
connected_slaves:1
slave0:ip=127.0.0.1,port=6381,state=online,offset=7935,lag=0
master_replid:13ceb3aa11fd92fa4bb5ea643f31a7db9f24d46c
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:7935
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:7726
repl_backlog_histlen:210
```

**数据透传**

```bash
# 79主机
127.0.0.1:6379> set k9 v9
OK

# 80从机
127.0.0.1:6380> get k9
"v9"

# 81从机
127.0.0.1:6381> get k9
"v9"
```



### 3）反客为主

Slaveof	no	one

使当前数据库停止与其它数据库的同步，转成主数据库

```bash
# 79主机关闭
127.0.0.1:6379> SHUTDOWN
not connected> exit
[root@node-1 bin]# 

# 80从机转为主机
127.0.0.1:6380> SLAVEOF no one
OK
127.0.0.1:6380> set k10 v10
OK

# 81从机连接新的 80主机
127.0.0.1:6381> SLAVEOF 127.0.0.1 6380
OK
127.0.0.1:6381> get k10
"v10"

# 新的80主机
127.0.0.1:6380> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6381,state=online,offset=8937,lag=1
master_replid:da65277a19eda71f963d5e08a0b82507c6c2e18f
master_replid2:13ceb3aa11fd92fa4bb5ea643f31a7db9f24d46c
master_repl_offset:8937
second_repl_offset:8716
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:7726
repl_backlog_histlen:1212

# 重启79主机，已成了光杆司令
[root@node-1 bin]# redis-server /myredis/redis6379.conf 
[root@node-1 bin]# redis-cli -p 6379
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_replid:adf095d81da3ec7df208a830c7b5fd62e5ccda39
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```



## 3、复制原理

1）Slave启动成功连接到 Master 后会发送一个 sync 命令

2）Master  接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，Master将传送整个数据文件到 Slave，**以完成一次完全同步**

3）全量复制：而 Slave 服务在接收到数据库文件数据后，将其存盘并加载到内存中

4）增量复制：**Master继续将新的所有收集到的修改命令依次传给 Slave，完成同步**

5）但是只要是重新连接 Master，依次完全同步（全量复制）将被自动执行



## 4、哨兵模式：sentinel

反客为主的自动版，能够在后台监控主机是否发生故障，如果故障了根据投票数自动将从库转为主库

```bash
# 1.自定义的 /myredis目录下新建 sentinel.conf 文件
# 2.设置哨兵
sentinel monitor 被监控的host名 127.0.0.1 6379 1
上面最后一个数字1，表示主机挂掉后 Slave投票看谁接替成为主机，得票数多少后成为主机

# 3.启动哨兵
[root@node-1 bin]# redis-sentinel /myredis/sentinel.conf

```

### 1）原主机宕机，剩余从机自动投票

```bash
# 79主机退出
127.0.0.1:6379> SHUTDOWN
not connected> exit
[root@node-1 bin]# 

# 投票，80被自动选为新的主机, 81从机现在连上80主机了
127.0.0.1:6380> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6381,state=online,offset=12997,lag=1
master_replid:da65277a19eda71f963d5e08a0b82507c6c2e18f
master_replid2:13ceb3aa11fd92fa4bb5ea643f31a7db9f24d46c
master_repl_offset:12997
second_repl_offset:8716
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:7726
repl_backlog_histlen:5272
127.0.0.1:6380> 

# 81为从机，连接的主机为80
127.0.0.1:6381> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6380
master_link_status:up
master_last_io_seconds_ago:9
master_sync_in_progress:0
slave_repl_offset:12983
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:da65277a19eda71f963d5e08a0b82507c6c2e18f
master_replid2:13ceb3aa11fd92fa4bb5ea643f31a7db9f24d46c
master_repl_offset:12983
second_repl_offset:8716
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:12983
```

### 2）原有 Master 重启

```bash

```





## 5、复制的缺点