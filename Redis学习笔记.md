# 一、NoSQL入门简介

## 1、3V+3高

大数据时代的3V：海量（Volume）， 多样（Variety），实时（Velocity）

互联网需求的3高：高并发（），高可扩展（），高性能（）



## 2、数据如何存放

以女装、女包为例

1、商品基本信息：名称、价格，出厂日期，生产厂商等

关系型数据库：MySQL/Oracle

2、商品描述、详情、评价信息（多文字类）

多文字信息描述类，IO读写性能变差：文档数据库 MongDB 中

3、商品的图片

分布式的文件系统中：淘宝自己的TFS、Google的GFS、Hadoop的HDFS

4、商品的关键字

搜索引擎，淘宝内用：ISearch

5、商品的波段性的热点高频信息

内存数据库：Tair、Redis、 Memcache

6、商品的交易、价格计算、积分累计

支付接口，支付宝



总结大型互联网应用的难点和解决方案：

难点：数据类型多样性，数据源多样性和变化重构，数据源改造而数据服务平台不需要大面积重构

解决办法：

EAI和统一数据平台服务层

UDSL：映射、API、热点缓存

 

## 3、NoSQL数据模型简介

以一个电商客户、订单、订购、地址模型来对比关系型数据库和非关系型数据库

1）、关系型数据库：ER图

![](G:\03_Markdown\Images\redis\电商订单ER图.png)

2）、NoSQL如何设计：聚合模型

​	1.KV键值

​	2.BSON：Binary JSON内嵌的文档对象和数组对象

​	3.列族

​	4.图形

3）、问题和难点

为什么上述的情况可以用聚合模型来处理：

​	1.高并发的操作是不太建议由关联查询的，

​	2.互联网公司用冗余数据来避免关联查询

​	3.分布式事物是支持不了太多的并发的 



## 4、NoSQL数据库的四大分类

​	1.KV键值：新浪（BerkeleyDB+redis），美团（redis+tair），阿里、百度：memcache+redis

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

​	1.Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用

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

# 启动 redis 客户端，需要使用 -p，因为服务端已经占用了 6379 端口
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
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> get k1
"v1"

127.0.0.1:6379> append k1 name
(integer) 6
127.0.0.1:6379> get k1
"v1name"

127.0.0.1:6379> strlen k1
(integer) 6
```

2）Incr、decr、incrby、decrby：必须是数字才能进行加减

```bash
127.0.0.1:6379> set k2 2
OK
127.0.0.1:6379> set k3 v3
OK
127.0.0.1:6379> incr k2
(integer) 3
127.0.0.1:6379> INCR k2
(integer) 4

# 步进
127.0.0.1:6379> INCRBY k2 2
(integer) 6
127.0.0.1:6379> INCRBY k2 2
(integer) 8
```

```bash
127.0.0.1:6379> DECR k2
(integer) 7
127.0.0.1:6379> DECR k2
(integer) 6

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
127.0.0.1:6379> GETRANGE k1 0 -1
"v1name"
127.0.0.1:6379> GETRANGE k1 0 3
"v1na"

127.0.0.1:6379> SETRANGE k1 3 abc
(integer) 6
127.0.0.1:6379> get k1
"v1nabc"

# 索引不能使用 -1 
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
127.0.0.1:6379> setex k4 10 v4
OK
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
127.0.0.1:6379> SETNX k1 aaaa
(integer) 0
127.0.0.1:6379> get k1
"ab456"
127.0.0.1:6379> SETNX k22 222
(integer) 1

```



**5）mset、mget、msetnx**

```bash
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3
OK
127.0.0.1:6379> keys *
1) "k2"
2) "k3"
3) "k22"
4) "k1"

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
127.0.0.1:6379> MSETNX k3 v3 k4 v4
(integer) 0
```



## 3、List

Redis 列表是简单的字符串列表，按照顺序排序。一可以添加一个元素到列表的头部（左边）或者尾部（右边）。它的底层实际是个链表

1）lpush、rpush、lrange

```bash
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





## 4、hash

键值对集合，是一个string类型的field 和 value 映射表，hash 特别适合用于存储对象

## 5、Set

Redis 的set是string类型的无序集合。它是通过 HashTable 实现的



## 6、Zset有序集合

Redis zset 和 set 一样也是string类型元素的集合，且不允许重复的成员

不同的是每个元素都会关联一个 double类型的分数

Redis 正式通过分数来为集合中的成员进行从小到大的排序。zset 的成员是唯一的，但分数（score）却可以重复







