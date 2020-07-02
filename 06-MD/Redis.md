

# Redis

# 一、Redis介绍

## 1、NOSQL基本简介

### 1、演变

①一开始，一个网站的访问量一般都不大，用单个数据库完全可以轻松应付。

![1585038158939](SpringCloud.assets\1585038158939.png)

②后来，访问量上升，程序员们开始大量的使用缓存技术来缓解数据库的压力，优化数据库的结构和索引。在这个时候，Memcached就自然的成为一个非常时尚的技术产品。

![1585038171976](SpringCloud.assets\1585038171976.png)

③由于数据库的写入压力增加，**Memcached只能缓解数据库的读取压力**。**读写集中在一个数据库上让数据库不堪重负，大部分网站开始使用主从复制技术来达到读写分离**，以提高读写性能和读库的可扩展性。Mysql的master-slave模式成为这个时候的网站标配了。

![1585038187162](SpringCloud.assets\1585038187162.png)

④后来MySQL主库的写压力开始出现瓶颈，而数据量的持续猛增，由于MyISAM使用表锁，在高并发下会出现严重的锁问题，大量的高并发MySQL应用开始使用InnoDB引擎代替MyISAM。

同时，开始流行使用分表分库来缓解写压力和数据增长的扩展问题，分表分库成了一个热门技术。

![1585038203526](SpringCloud.assets\1585038203526.png)

⑤现在的架构。

![1585038215888](SpringCloud.assets\1585038215888.png)

### 2、RDBMS和NOSQL对比

RDBMS

- 高度组织化结构化数据，结构化查询语言（SQL）
- 数据和关系都存储在单独的表中
- 数据操纵语言，数据定义语言
- 严格的一致性
- 基础事务

NoSQL

- 代表着不仅仅是SQL，没有声明性查询语言，没有预定义的模式
- 键 - 值对存储，列存储，文档存储，图形数据库
- **最终一致性**，而非ACID属性
- 非结构化和不可预知的数据
- CAP定理
- 高性能，高可用性和可伸缩性

### 3、在分布式数据库中CAP原理CAP+BASE

传统数据库：ACID，即：原子性、一致性、隔离性、持久性。

CAP原理:

- C:Consistency（强一致性）；
- A:Availability（可用性）；
- P:Partition tolerance（分区容错性）；

**CAP理论就是说在分布式存储系统中，最多只能实现上面的两点**。

![1584972348179](SpringCloud.assets\1584972348179.png)

**而由于当前的网络硬件肯定会出现延迟丢包等问题，所以分区容忍性是我们必须需要实现的**。

所以我们只能在一致性和可用性之间进行权衡，**没有NoSQL系统能同时保证这三点。**

- `CA` : 传统Oracle数据库。单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
- `AP`  : 大多数网站架构的选择。满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。
- `CP` : Redis、Mongodb。满足一致性，分区容忍必的系统，通常性能不是特别高。

 注意：分布式架构的时候必须做出取舍。

一致性和可用性之间取一个平衡。多余大多数web应用，其实并不需要强一致性。因此牺牲C换取P，这是目前分布式数据库产品的方向。

> BASE就是为了解决关系数据库强一致性引起的问题而引起的可用性降低而提出的解决方案。
> BASE其实是下面三个术语的缩写：
>
> - 基本可用（Basically Available）
> - 软状态（Soft state）
> - 最终一致（Eventually consistent）

## 2、docker安装redis

拉去镜像：

```
docker pull redis
```

运行容器：

```
docker run -p 9000:9000 --name redis -v /usr/local/redis/redis.conf:/etc/redis/redis.conf -v /usr/local/docker/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
```

- 在/usr/local/redis/下编写redis.conf文件，此处将redis端口改为9000

进入容器：

```
docker exec -it 7c1c5e8496e7 redis-cli -p 9000
```

进入集群容器：

```
docker exec  -it 4483b2a6af16 redis-cli  -c -p 7001
```

- 需要加上-c参数，打开集群支持

## 3、普通安装

```
 wget http://download.redis.io/releases/redis-4.0.8.tar.gz
```

解压：

```
tar -zxvf redis-4.0.8.tar.gz
```

编译：

```
cd redis-4.0.8
make && make install
```



# 二、Redis入门

## 1、相关基础

基本命令:

- `select idx`: 选择第idx库。
- `dbsize` : 查看当前数据库的key的数量。
- `flushdb`：清空当前库。
- `Flushall`: 通杀全部库。
- 统一密码管理，16个库都是同样密码，要么都OK要么一个也连接不上。
- Redis索引都是从零开始。
- 默认端口`6379`。

Redis和其他`key-value`缓存产品都有如下特点:

- Redis支持**数据的持久化**，可以将内存中的**数据保持在磁盘**中，重启的时候可以再次加载进行使用；
- Redis不仅仅支持简单的key-value类型的数据，同时还提供`list，set，zset，hash`等数据结构的存储；
- Redis**支持数据的备份**，即master-slave模式的数据备份；

出现一个报错的问题解决方案: https://blog.csdn.net/u011627980/article/details/79891597?utm_source=blogxgwz9

## 2、Redis数据类型

| 结构类型   | 结构存储的值                                                 | 结构的读写能力                                               |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **STRING** | **可以是字符串、整数或者浮点数**                             | 对整个字符串或者字符串的其中一部分执行操作、对整数和浮点数执行自增或自减操作 |
| **LIST**   | 一个链表，链表上的每个节点都包含了一个字符串                 | 从**两端压入或者弹出元素**  对单个或者多个元素 进行修剪，只保留一个范围内的元素 |
| **SET**    | 包含字符串的**无序**收集器（unordered collection），并且被包含的每个字符串都互不相同的。 | 添加、获取、移除单个元素，检查一个元素是否存在于集合中、 **计算交集、并集、差集** ，从集合里面随机获取元素 |
| **HAST**   | 包含**键值对**的无序散列表                                   | 添加、获取、移除单个键值对 ，获取所有键值对 检查某个键是否存在 |
| **ZSET**   | 字符串成员（member）与浮点数分值（score）之间的**有序映射**，元素的排列顺序由**分值的大小**决定 | 添加、获取、删除元素 ，根据分值范围或者成员来获取元素，计算一个键的排名 |

![1584972900003](SpringCloud.assets\1584972900003.png)

### 1、基本命令

```shell
127.0.0.1:6379> select 0
OK
127.0.0.1:6379> keys *
1) "k2"
2) "k3"
127.0.0.1:6379> move k2 2 # 将k2移动到２号数据库
(integer) 1
127.0.0.1:6379> select 2
OK
127.0.0.1:6379[2]> keys *
1) "k2"
127.0.0.1:6379[2]> select 0
OK
127.0.0.1:6379> keys *
1) "k3"
127.0.0.1:6379> exists k3
(integer) 1
127.0.0.1:6379> exists k1
(integer) 0
127.0.0.1:6379> del k3
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> set k1 v100  # 默认会覆盖
OK
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
127.0.0.1:6379> type k1
string
127.0.0.1:6379> lpush mylist 1 2 3 4 5
(integer) 5
127.0.0.1:6379> type mylist
list
127.0.0.1:6379> ttl k1 # ttl查看还能存活的时间 -1表示永久, -2表示已经死了
(integer) -1
127.0.0.1:6379> expire k2 10 # 设置k2 的ttl为10秒钟
(integer) 1
127.0.0.1:6379> ttl k2
(integer) 7
127.0.0.1:6379> ttl k2
(integer) 4
127.0.0.1:6379> ttl k2
(integer) -2
127.0.0.1:6379> keys *
1) "mylist"
2) "k1"
127.0.0.1:6379> 
```

### 2、string

![1584972975249](SpringCloud.assets\1584972975249.png)

```shell
127.0.0.1:6379> del mylist
(integer) 1
127.0.0.1:6379> append k1 12345 # 追加
(integer) 9
127.0.0.1:6379> get k1
"v10012345"
127.0.0.1:6379> STRLEN k1 # 长度
(integer) 9
127.0.0.1:6379> set k2 2
OK
127.0.0.1:6379> INCR k2 # 自增
(integer) 3
127.0.0.1:6379> INCR k2
(integer) 4
127.0.0.1:6379> get k2
"4"
127.0.0.1:6379> DECR k2
(integer) 3
127.0.0.1:6379> get k2
"3"
127.0.0.1:6379> INCRBY k2 10 # + num
(integer) 13
127.0.0.1:6379> get k2
"13"
127.0.0.1:6379> DECRBY k2 10
(integer) 3
127.0.0.1:6379> set k3 v3
OK
127.0.0.1:6379> INCR k3 # incr, decr,incrby,decrby都只能对数字进行操作
(error) ERR value is not an integer or out of range
127.0.0.1:6379> get k1
"v10012345"
127.0.0.1:6379> GETRANGE k1 0 -1 # getrange 获取指定范围的值
"v10012345"
127.0.0.1:6379> GETRANGE k1 0 3
"v100"
127.0.0.1:6379> SETRANGE k1 0 xxx
(integer) 9
127.0.0.1:6379> get k1
"xxx012345"
127.0.0.1:6379> setex k4 10 v4 # set with expire
OK
127.0.0.1:6379> ttl k4
(integer) 7
127.0.0.1:6379> ttl k4
(integer) 2
127.0.0.1:6379> ttl k4
(integer) -2
127.0.0.1:6379> get k4
(nil)
127.0.0.1:6379> keys *
1) "k2"
2) "k3"
3) "k1"
127.0.0.1:6379> setnx k1 v111 # 存在就设值不进去
(integer) 0
127.0.0.1:6379> get k1
"xxx012345"
127.0.0.1:6379> setnx k5 v5 # 不存在就设值
(integer) 1
127.0.0.1:6379> get k5
"v5"
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3 # 一次设值多个
OK
127.0.0.1:6379> mget k1 k2 k3
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> 

```

### 3、List

![1584973124812](SpringCloud.assets\1584973124812.png)

```shell
127.0.0.1:6379> LPUSH list01 1 2 3 4 5 # 从左边分别推入1、2、3、4、5
(integer) 5
127.0.0.1:6379> LRANGE list01 0 -1
1) "5"
2) "4"
3) "3"
4) "2"
5) "1"
127.0.0.1:6379> RPUSH list02 1 2 3 4 5
(integer) 5
127.0.0.1:6379> LRANGE list02 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
127.0.0.1:6379> LPOP list01 # 弹掉最左边的,是5
"5"
127.0.0.1:6379> LPOP list02
"1"
127.0.0.1:6379> RPOP list01
"1"
127.0.0.1:6379> RPOP list02
"5"
127.0.0.1:6379> LINDEX list01 1 # 相当于 s.charAt()
"3"
127.0.0.1:6379> LINDEX list02 2
"4"
127.0.0.1:6379> LLEN list01 # s.length()
(integer) 3
127.0.0.1:6379> LRANGE list01 0 -1
1) "4"
2) "3"
3) "2"
127.0.0.1:6379> RPUSH list03 1 1 1 2 2 2 3 3 3 4 4 4
(integer) 12
127.0.0.1:6379> LREM list03 2 3 # 删除N个value , 删除2个3
(integer) 2
127.0.0.1:6379> LRANGE list03 0 -1
 1) "1"
 2) "1"
 3) "1"
 4) "2"
 5) "2"
 6) "2"
 7) "3"
 8) "4"
 9) "4"
10) "4"
127.0.0.1:6379> LTRIM list03 3 5 # s = s.substring(3, 5)
OK
127.0.0.1:6379> LRANGE list03 0 -1
1) "2"
2) "2"
3) "2"
127.0.0.1:6379> LRANGE list02 0 -1
1) "2"
2) "3"
3) "4"
127.0.0.1:6379> RPOPLPUSH list02 list03 # RPOPLPUSH,将第一个的最后一个放到第二个的头部
"4"
127.0.0.1:6379> LRANGE list03 0 -1
1) "4"
2) "2"
3) "2"
4) "2"
127.0.0.1:6379> LSET list03 1 x # s.set(idx, value);
OK
127.0.0.1:6379> LRANGE list03 0 -1
1) "4"
2) "x"
3) "2"
4) "2"
127.0.0.1:6379> LINSERT list03 before x java
(integer) 5
127.0.0.1:6379> LRANGE list03 0 -1
1) "4"
2) "java"
3) "x"
4) "2"
5) "2"
127.0.0.1:6379> 		
```

总结:

- 它是一个字符串链表，left、right都可以插入添加；
- 如果键不存在，创建新的链表；
- 如果键已存在，新增内容；
- 如果值全移除，对应的键也就消失了。
- 链表的操作无论是头和尾效率都极高，但假如是对中间元素进行操作，效率就很惨淡了。

### 4、Set

![1584973183675](SpringCloud.assets\1584973183675.png)

```shell
127.0.0.1:6379> sadd set01 1 1 2 2 3 3 # 创建set01并加入元素
(integer) 3
127.0.0.1:6379> smembers set01 # 查看set中的所有元素,注意是去重的s
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> SISMEMBER set01 1 #判断set01中是否有 ‘1’
(integer) 1
127.0.0.1:6379> SISMEMBER set01 x
(integer) 0
127.0.0.1:6379> SCARD set01 # 元素个数
(integer) 3
127.0.0.1:6379> SREM set01 2 # 删除2
(integer) 1
127.0.0.1:6379> SMEMBERS set01
1) "1"
2) "3"
127.0.0.1:6379> sadd set01 5 8 3 12 6
(integer) 4
127.0.0.1:6379> SMEMBERS set01
1) "1"
2) "3"
3) "5"
4) "6"
5) "8"
6) "12"
127.0.0.1:6379> SRANDMEMBER set01 2 # 随机取两个
1) "6"
2) "12"
127.0.0.1:6379> SRANDMEMBER set01 2 # 随机取两个
1) "1"
2) "6"
127.0.0.1:6379> SPOP set01# 随机出栈
"1"
127.0.0.1:6379> SPOP set01
"12"
127.0.0.1:6379> SMEMBERS set01
1) "3"
2) "5"
3) "6"
4) "8"
127.0.0.1:6379> flushdb # 清空了一下
OK
127.0.0.1:6379> SADD set01 1 2 3 4 5
(integer) 5
127.0.0.1:6379> SADD set02 1 2 3 a b
(integer) 5
127.0.0.1:6379> SDIFF set01 set02 # 差集
1) "4"
2) "5"
127.0.0.1:6379> SINTER set01 set02 # 交集
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> SUNION set01 set02 # 并集
1) "b"
2) "3"
3) "1"
4) "5"
5) "a"
6) "4"
7) "2"
127.0.0.1:6379> 
```

注意上面差集: **在第一个set里面而不在后面任何一个set里面的项**。

### 5、Hash

![1584973229376](SpringCloud.assets\1584973229376.png)

![1584973241132](SpringCloud.assets\1584973241132.png)

```shell
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> hset user name z3
(integer) 1
127.0.0.1:6379> hget user name
"z3"
127.0.0.1:6379> hmset customer id 11 name li4 age 24 # 批量设置
OK
127.0.0.1:6379> hmget customer id name age
1) "11"
2) "li4"
3) "24"
127.0.0.1:6379> HGETALL customer# 获取所有 key、value、key、value..格式呈现
1) "id"
2) "11"
3) "name"
4) "li4"
5) "age"
6) "24"
127.0.0.1:6379> HDEL customer name # 删除
(integer) 1
127.0.0.1:6379> HGETALL customer 
1) "id"
2) "11"
3) "age"
4) "24"
127.0.0.1:6379> HLEN customer # 长度
(integer) 2
127.0.0.1:6379> HEXISTS customer id # 判断是否存在
(integer) 1
127.0.0.1:6379> HEXISTS customer email
(integer) 0
127.0.0.1:6379> HKEYS customer　# 获取所有key
1) "id"
2) "age"
127.0.0.1:6379> HVALS customer
1) "11"
2) "24"
127.0.0.1:6379> HINCRBY customer age 2 #增
(integer) 26
127.0.0.1:6379> HINCRBY customer age 2
(integer) 28
127.0.0.1:6379> HSET customer score 91.5
(integer) 1
127.0.0.1:6379> HINCRBYFLOAT customer score 0.5
"92"
127.0.0.1:6379> HSETNX customer age 26 #不存在就赋值
(integer) 0
127.0.0.1:6379> HSETNX customer email abc@163.com
(integer) 1
127.0.0.1:6379> 
```

6、ZSet（有序集合）

ZSet就是在`set`的基础上，加一个`score`值。里面按照`score`值排序。也就是每两个是一个整体。

例如:

- 之前是set01    :   `v1   v2   v3`；
- 现在是zset01    :   `score1  v1   score2  v2    score3  v3`；

![1584973300559](SpringCloud.assets\1584973300559.png)

```shell
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> ZADD zset01 60 v1 70 v2 80 v3 90 v4 100 v5 # 添加,每个整体有两个内容v,score
(integer) 5
127.0.0.1:6379> ZRANGE zset01 0 -1
1) "v1"
2) "v2"
3) "v3"
4) "v4"
5) "v5"
127.0.0.1:6379> 
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
127.0.0.1:6379> ZRANGEBYSCORE zset01 60 90
1) "v1"
2) "v2"
3) "v3"
4) "v4"
127.0.0.1:6379> ZRANGEBYSCORE zset01 60 (90
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> ZRANGEBYSCORE zset01 (60 (90
1) "v2"
2) "v3"
127.0.0.1:6379> ZRANGEBYSCORE zset01 60 90 limit 1 2
1) "v2"
2) "v3"
127.0.0.1:6379> ZREM zset01 v5 # 删除v5
(integer) 1
127.0.0.1:6379> ZCARD zset01 # 个数
(integer) 4
127.0.0.1:6379> ZCOUNT zset01 60 80 # 一个范围的个数
(integer) 3
127.0.0.1:6379> ZRANK zset01 v3 # 排名
(integer) 2
127.0.0.1:6379> ZSCORE zset01 v2
"70"
127.0.0.1:6379> ZREVRANK zset01 v3
(integer) 1
127.0.0.1:6379> ZREVRANGE zset01 0 -1
1) "v4"
2) "v3"
3) "v2"
4) "v1"
127.0.0.1:6379> 
```

# 三、持久化

![1584974955957](SpringCloud.assets\1584974955957.png)

两种形式：

![1584974982679](SpringCloud.assets\1584974982679.png)

## 1、RDB（Redis Database）

- **基本概念**

  概念: **在指定的时间间隔内将内存中的数据集快照写入磁盘， 也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里**。

   	Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到 一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能 

- **第一种RDB持久化方式——手动持久化**

  ```
  save
  ```

  ![1584975059576](SpringCloud.assets\1584975059576.png)

  生成dump.rdb文件![1584975117015](SpringCloud.assets\1584975117015.png)

RDB相关配置文件	![1584975309193](SpringCloud.assets\1584975309193.png)

- **save指令工作原理**

  ![1584975494361](SpringCloud.assets\1584975494361.png)

  

- **第二种RDB持久化方式——后台持久化**

  ```
  bgsave
  ```

  作用：手动后台保存操作，不是立即执行

  ![1584975714847](SpringCloud.assets\1584975714847.png)

- **后台持久化工作原理**

  ![1584975873086](SpringCloud.assets\1584975873086.png)![1584975914035](SpringCloud.assets\1584975914035.png)

- **问题**：以上两种方式都是自己手动执行，忘记怎么办？

- **第三种RDB持久化方式——自动执行**

  需要修改配置文件，无需手动执行，也是属于执行bgsave

  ![1584976501514](SpringCloud.assets\1584976501514.png)

  

- **RDB三种方式对比**

  ![1584976692008](SpringCloud.assets\1584976692008.png)

  ![1584976723057](SpringCloud.assets\1584976723057.png)

- **如何恢复**

  将备份文件 (`dump.rdb`) 移动到 redis 安装目录并启动服务即可。

## 2、AOF(Append Only File)

- **基本概念**

  以日志的形式来记录**每个写操作**。

  ​	**将Redis执行过的所有写指令记录下来(读操作不记录)**， 只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。**AOF保存的是`appendonly.aof`文件**。

- **RDB产生的问题**

  ![1584977317018](SpringCloud.assets\1584977317018.png)

- **AOF写数据的三种策略（appendfsync）**

  ![1584977579782](SpringCloud.assets\1584977579782.png)

- **AOF功能开启**

  ```
  appendonly yes|no
  ```

  作用：是否开启AOF功能，默认不开启

  ```
  appendfsync always|everysec|no
  ```

  作用：AOF写数据策略

  ![1584977910122](SpringCloud.assets\1584977910122.png)

  之后当用户写入命令，就会自动写入aof文件

- **AOF重写**

  AOF写数据时会遇到这种情况：

  ![1584978137718](SpringCloud.assets\1584978137718.png)

  set值覆盖，或者可以把incr优化成set num 3 ，可以把不必要的命令不写入aof文件。

  ![1584978246246](SpringCloud.assets\1584978246246.png)

  **AOF重写的作用**：

  降低磁盘占用率，提高磁盘利用率；

  提高持久化效率，降低持久化写时间，提高IO性能；

  降低数据恢复用时，提高数据恢复效率；

  

  **AOF重写方式：**

  手动重写：

  ```
  bgrewriteaof
  ```

  自动重写

  ```
  auto-aof-rewrite-min-size size
  auto-aof-rewrite-percentage percentage
  ```

  ![1584978556725](SpringCloud.assets\1584978556725.png)

  经过优化重写后，aof文件只写入set name 345；

  **begrewriteaof指令工作原理**

  ![1584978682345](SpringCloud.assets\1584978682345.png)

  ![1584978882551](SpringCloud.assets\1584978882551.png)

## 3、RDB vs AOF

![1584979057512](SpringCloud.assets\1584979057512.png)![1584979104386](SpringCloud.assets\1584979104386.png)

# 四、事务

## 1、什么是事务

问题：Redis执行指令的过程中，多条连续指令被干扰，打断，插队会对结果遭影响。

**事务是可以一次执行多个命令，本质是一组命令的队列。一个事务中的 所有命令都会序列化，按顺序地串行化执行而不会被其它命令插入，不许加塞。**

事务能做的事: **一个队列中，一次性、顺序性、排他性的执行一系列命令。**

## 2、事务的基本操作

开启事务：

```
multi
```

作用：设定事务的开始位置，此指令后，后续指令均加入到事务中；

执行事务：

```
exec
```

作用：设定事务的结束位置，同时执行事务，与multi成对出现，成对使用；

![1585014464552](SpringCloud.assets\1585014464552.png)

取消事务：

```
discard
```

作用：终止当前事务，发生在multi后，exec前；

![1585014586000](SpringCloud.assets\1585014586000.png)

## 3、事务注意事项

定义事务过程中，命令格式输入错误怎么办？

- 语法错误：指令格式错误

  ​				处理结果：事务包含错误命令语法，整个事务将不会执行。

- 运行错误：指令格式正确，但无法正确执行，例如：对list进行incr操作

  ​				处理结果：事务能过执行，但是会跳过运行错误的指令

## 4、锁

**场景**：市场双11人们抢购商品，一会货架就空了，需要补货，此时有4个业务员有权限补货，补货可能是一系列操作，如何防止不重复操作？

**分析**：多个客户端可能同时操作同一个数据，并且该数据一旦被修改就不用再去操作了；

​			此时就需要在操作前锁定该数据，一旦在自己操作过程中有其他人操作，数据变化就，就终止自己事务操作。

**加锁操作**：

```
watch key1 [key2....]
```

作用：对key添加**监视锁**，在multi之前执行，在事务执行exec前如果key发生变化，终止事务执行；

**取消锁**：

```
unwatch
```

作用：取消所有监视锁

![1585016277558](SpringCloud.assets\1585016277558.png)

在执行exec前name或者age发生变化，事务将会终止；显示nil；

如果出现nil，修改失败：

- 先解锁
- 获取新值重新监视

```shell

127.0.0.1:6379> unwatch   #如果发现事务执行失败，就先解锁
OK
127.0.0.1:6379> watch money    #获取最新的值，再次监视
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 60
QUEUED
127.0.0.1:6379> incrby out 60
QUEUED
127.0.0.1:6379> exec  #比对监视的值是否发生变化，如果没有发生变化，可以执行，如果变化就执行失败，继续解锁再次监视。
1) (integer) 1020
2) (integer) 80
```



## 5、分布式锁

**场景**：双11如何避免最后一个商品不会被多个人同时购买？（超买问题）

**使用分布式锁**：

```
setnx lock-key value
```

以及**设置过**锁就返回设置**失败**，**第一次**设锁就设置**成功**

- 设置成功，拥有控制权，进行下一步操作
- 设置失败，不具有控制权，进入排队或等待

**释放锁(删除锁的key)**：

```
del lock-key
```

![1585017602136](SpringCloud.assets\1585017602136.png)

获取锁后其他人将不会再获取锁

**存在问题**：

​	如果获取了锁的那一个人宕机了……怎么办？

**解决方法**：

```
expire lock-key second
pexpire lock-key milliseconds
```

给锁key添加一个时间限定，时间到了不释放锁，就自动放弃锁。

![1585018256327](SpringCloud.assets\1585018256327.png)

lock-name获取锁，无论他自己是否del锁，都会在10秒后自动释放；

![1585018317741](SpringCloud.assets\1585018317741.png)

# 五、Redis消息订阅发布

**概念:**

​		进程间的一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息；

左边窗口开始订阅`c1、c2、c3`三个频道。右边还没有操作。

![1585019215810](SpringCloud.assets\1585019215810.png)

然后右边开始发布消息。

![1585019237337](SpringCloud.assets\1585019237337.png)

**总结:**

先订阅后发布后才能收到消息，

- 可以一次性订阅多个，`SUBSCRIBE c1 c2 c3`。
- 消息发布，`PUBLISH c2 hello-redis`。
- 订阅多个，通配符`*`，`PSUBSCRIBE new*`。
- 收取消息， `PUBLISH new1 redis2015`。

# 六、删除策略

**删除对象**：expire 控制有时效性，过期之后还在内存中的数据

## 1、过期数据

**Redis中数据特征**：

Redis是一种内存级数据库，所有数据均存放再内存中，内存中的数据可以通过TTL指令获取其状态

- XX：具有时效性
- -1：永久有效
- -2：**已经过期**的数据或者**被删除**的数据或**未定义**的数据

**问题**：**过期的数据**真的删除了吗？  

## 2、数据删除策略

-  **定时删除（cpu压力大）**

  ![1585026832826](SpringCloud.assets\1585026832826.png)

- **惰性删除（内存压力大）**

  ![1585026974518](SpringCloud.assets\1585026974518.png)

- **定期删除（折中方案，轮询周期去做）**

  ![1585027457123](SpringCloud.assets\1585027457123.png)

  redis服务器会每秒执行server.hz（10）次serverCron方法

  再执行databasesCron方法，轮询16个数据库；

  再在每个数据库执行activeExpireCycle方法，随机挑选W个key，看它有没有过期。

  ![1585027706804](SpringCloud.assets\1585027706804.png)

- 三个删除策略对比

  ![1585027745123](SpringCloud.assets\1585027745123.png)

## 3、逐出算法

解决的问题：**添加新数据时候，redis内存不足**（数据都没有过期，都长期存在，这种情况无法用删除策略解决）

 ![1585028138778](SpringCloud.assets\1585028138778.png)

![1585028188118](SpringCloud.assets\1585028188118.png)

![1585028305658](SpringCloud.assets\1585028305658.png)

配置

![1585028408164](SpringCloud.assets\1585028408164.png)

# 七、redis.conf基础配置

- 设置服务器以守护进程方式运行

  ```
  daemonize yes|no
  ```

- 绑定主机地址

  ```
  bind 127.0.0.1
  ```

- 设置服务器端口

  ```
  prot 6379
  ```

- 设置数据库数量

  ```
  databases 16
  ```

- 设置服务器日志级别

  ```
  loglevel  debug|verbose|notice|warning
  ```

- 日志名

  ```
  logfile 端口号.log
  ```

- 设置同一时间最大客户端连接数

  ```
  maxclients 0
  ```

- 客户端闲置等待最大是时长300秒，达到最大时间后断开连接，关闭功能设置 0

  ```
  timeout 300
  ```

- 导入并加载指定配置文件信息，用于快速创建redis公共配置比较多的redis 实例配置文件

  ```
  include /path/server-端口.conf
  ```

- 部网络可以直接访问

  ```
  protected-mode no
  ```

- dir./ 数据目录，数据库的写入会在这个目录。rdb、aof文件也会写在这个目录

  ```
  dir "/usr/local/redis_cluster/general-redis-cluster/redis-4.0.8/src"
  ```

  

# 八、主从复制

## 1、主从复制介绍

**互联网三高**

- 高并发
- 高性能
- 高可用

**单台Redis存在的问题**：

![1585038276001](SpringCloud.assets\1585038276001.png)

**多台Redis**：

![1585038614576](SpringCloud.assets\1585038614576.png)

**主从复制的作用**：

![1585038884391](SpringCloud.assets\1585038884391.png)

## 2、主从复制工作流程

![1585038993518](SpringCloud.assets\1585038993518.png)

### 阶段一：建立连接

![1585039315318](SpringCloud.assets\1585039315318.png)

**建立连接——方式一（不主流）**：

在从服务器客户端之间操作：

```
slaveof 主机ip 主机端口
```

主机显示连接成功：

![1585042205858](SpringCloud.assets\1585042205858.png)

**建立连接——方式二（不主流）：**

启动slave服务器时候就带上参数

![1585042279134](SpringCloud.assets\1585042279134.png)

**建立连接——方式三（主流）：**

配置文件添加：

```
slaveof <masterip> <masterport>
```

通过master客户端可以查看状态：

```
info
```

![1585042743946](SpringCloud.assets\1585042743946.png)

**断开连接：**

```
slaveof no one
```

**其他内容：**

![1585042938955](SpringCloud.assets\1585042938955.png)

### 阶段二：数据同步阶段工作流程

**目的：在slave初次连接后，复制master所有数据到slave，slave随着master更新**

![1585103007382](SpringCloud.assets\1585103007382.png)

**说明**：**全量复制**是指master通过持久化技术RDB拍快照的方式，将全部数据给slave；

​		  **部分复制**（增量复制）是指在全量复制的过程中，可能用户还在输入指令，需要保证这些指令也不丢失，就用缓存区存取指令，最后给slave； 

### 阶段三：命令传播

**命令传播作用**：**实时保持master和slave的数据同步**

命令传播阶段也可能启动**部分复制**，**场景**：有一台slave断网了，就需要缓冲区来复制；

- 服务器运行ID 

  ![1585103781363](SpringCloud.assets\1585103781363.png)

- 复制缓冲区（执行部分复制的核心内容）

  ![1585107554550](SpringCloud.assets\1585107554550.png)![1585107509009](SpringCloud.assets\1585107509009.png)

  同步之后，master来了指令，同步给slave，但是其中一台断网了，这就会导致几台slave数据不相同；这就需要复制缓冲区，先把命令存进缓冲区；

  ![1585107837708](SpringCloud.assets\1585107837708.png)

- **数据同步+命令传播阶段工作流程**

  ![1585108863678](SpringCloud.assets\1585108863678.png)

### 主从复制工作流程总结：

![1585109310777](SpringCloud.assets\1585109310777.png)

命令传播阶段：**通过心跳机制维持**

# 九、哨兵模式

## 1、哨兵介绍

**使用场景**：**搭建好主从复制后**，master宕机，这时候就需要哨兵来，确认选取一个slave作为新master。

![1585110907869](SpringCloud.assets\1585110907869.png)

**哨兵**：哨兵（sentinel）是一个分布式系统，用于对主从结构中的每台服务器进行**监控**，当出现故障时候通过投票机制**选取**新的master并将所有slave连接到新master

![1585111048366](SpringCloud.assets\1585111048366.png)

注意：哨兵也是一台redis服务器，只不过不提供数据服务，哨兵通常配置数量为单数（因为双数投票机制可能打平）

## 2、启动哨兵模式

- 首先配置一主双从的redis服务器（9005为master）

  ![1585152207131](SpringCloud.assets\1585152207131.png)

  依次启动主从服务器

  ```
  ./redis-server 配置文件.conf
  ```

- 配置三个哨兵，监控redis一主双从的master机器，3个哨兵端口不同，监控的master的ip 端口相同

- 配置sentinel.conf

  ![1585152491301](SpringCloud.assets\1585152491301.png)

  **还需配置**

  ```
  protected-mode no
  ```

  ![1585152524131](SpringCloud.assets\1585152524131.png)

  启动哨兵

  ```
  ./redis-sentinel 哨兵配置文件.conf
  ```

- 哨兵模式就搭建成功

  查看哨兵状态

  ```
  进入哨兵客户端：
  redis-cli -p 26379
  ```

  执行info命令：

  ![1585153416368](SpringCloud.assets\1585153416368.png)

- 测试：关闭master，看哨兵是否会选取下一个master

  ![1585156754572](SpringCloud.assets\1585156754572.png)

- **注意**：

  **如果原先宕机的master重新启动，将会变为slave，不再会是master**（重启全部redis后9006依然是master）

  ![1585157021549](SpringCloud.assets\1585157021549.png)

  并且哨兵会在redis.conf中配置新的master的配置：

  ![1585157469098](SpringCloud.assets\1585157469098.png)

## 3、工作原理

哨兵主要工作就是：主从切换

- 主从切换会经历三个过程

  ```
  1、监控
  2、通知
  3、转移
  ```

- **阶段一：监控**

  目的：哨兵之间同步信息

  ![1585191182571](SpringCloud.assets\1585191182571.png)

- **第二阶段：通知**

  保持连接

- **第三阶段：故障转移**

  第一个哨兵发现master挂了之后会给它标记一个状态：**s_down**；

  之后在哨兵网络中通知其他哨兵，说master挂了，当其他哨兵也确认挂了之后，会给宕机的master标记：**o_down**；

  之后哨兵会在哨兵中选出一个sentinel，去处理那个宕机的master；

  ![1585192360675](SpringCloud.assets\1585192360675.png)



# 十、集群

## 1、集群

![1585192616860](SpringCloud.assets\1585192616860.png)![1585192681112](SpringCloud.assets\1585192681112.png)

## 2、Redis集群建构设计

![1585192872654](SpringCloud.assets\1585192872654.png)

## 3、cluster集群建构搭建

**搭建3主3从集群**

需要启动6个redis并修改配置文件：

![1585201113413](SpringCloud.assets\1585201113413.png)

![1585201183440](SpringCloud.assets\1585201183440.png)

- 依次启动6个redis

- 查看启动

  ```
  ps -ef |grep redis
  ```

  ![1585201234018](SpringCloud.assets\1585201234018.png)
  
- 启动集群前需要有**ruby** 和 **gem**环境

  **安装ruby**

  **安装gem**

  ```
  wget -c http://production.cf.rubygems.org/rubygems/rubygems-1.8.24.tgz
  ```

  ```
  tar xzcf rubygems-1.8.24.tgz
  ```

  ```
  cd rubygems-1.8.24
  ```

  ```
  ruby setup.rb
  ```

  之后把gem命令放在/usr/bin/gem

  ```
  ln -s /usr/local/ruby/bin/rubygems-1.8.24/bin/gem /usr/bin/gem
  ```

  测试ruby，gem

  ![1585208400429](SpringCloud.assets\1585208400429.png)

- 执行集群命令

  ```
  ./redis-trib.rb create --replicas 1 192.168.174.131:8005 192.168.174.131:8006 192.168.174.131:8007 192.168.174.131:8008 192.168.174.131:8009 192.168.174.131:8010
  ```

  **如果出现次错误**

  ![1585208265309](SpringCloud.assets\1585208265309.png)

  **解决方法**：

  ```
  gem install redis
  ```

  如果出现：

  ![1585209652145](SpringCloud.assets\1585209652145.png)

  就去清空redis数据库的数据即可

- 集群搭建成功

  ![1585209556484](SpringCloud.assets\1585209556484.png)
  
- **注意：如果集群中某个slave宕机了，不会对集群功能有多大影响，只是该slave的master会提示；slave重新上线就行**

  ![1585232554828](SpringCloud.assets\1585232554828.png)

- **注意：如果集群中某个master宕机了，该master的slave会翻身做主成为新master；即使宕机的master恢复，也只能做从；**

  ![1585233338410](SpringCloud.assets\1585233338410.png)

# 十一、企业级解决方案

## 1、缓存预热

**现象**：服务器启动迅速就宕机了

**说明**：缓存预热就是系统上线后，**提前将相关的缓存数据直接加载到缓存系统**。避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题！用户直接查询事先被预热的缓存数据！	

**问题**：服务器启动或者重启的时候，里面还没有数据，**此时用户直奔数据库查询**，就会对服务器造成压力；**请求量较高**；**主从之间吞吐量较大**，数据同步操作较频繁，就会造成；

**解决方法**：

![1585234996807](SpringCloud.assets\1585234996807.png)

![1585235007325](SpringCloud.assets\1585235007325.png)

## 2、缓存雪崩

**发生场景**：当Redis服务器重启或者大量缓存在同一时期失效时,此时大量的流量会全部冲击到数据库上面,数据库有可能会因为承受不住而宕机

![1585235900681](SpringCloud.assets\1585235900681.png)

![1585235889294](SpringCloud.assets\1585235889294.png)

![1585235081761](SpringCloud.assets\1585235081761.png)

![1585235229683](SpringCloud.assets\1585235229683.png)

**问题**：

- **短时间内**
- **大量key集中过期**

**解决方案**：

![1585235474388](SpringCloud.assets\1585235474388.png)

![1585235576498](SpringCloud.assets\1585235576498.png)

![1585235746930](SpringCloud.assets\1585235746930.png)

## 3、缓存击穿

![1585236617240](SpringCloud.assets\1585236617240.png)

![1585236654601](SpringCloud.assets\1585236654601.png)

**问题分析**：

- 单个key高热数据
- key过期
- 类似出现一个微博热搜，突然访问量激增

![1585236847480](SpringCloud.assets\1585236847480.png)

![1585237017023](SpringCloud.assets\1585237017023.png)

## 4、缓存穿透

![1585237198579](SpringCloud.assets\1585237198579.png)

**问题排查：

- Redis中大面积出现未命中

- 出现非正常的URL访问（）

  ![1585237537836](SpringCloud.assets\1585237537836.png)![1585237687185](SpringCloud.assets\1585237687185.png)![1585237851543](SpringCloud.assets\1585237851543.png)

## 5、性能指标监控

- 性能指标 Performance

  |           Name            |       Description        |
  | :-----------------------: | :----------------------: |
  |          latency          | Redis响应一个请求的时间  |
  | instantaneous_ops_per_sec |   平均每秒处理请求总量   |
  |   hit rate(calculated)    | 缓存命中率（计算出来的） |

- 内存指标 Memory

  |           Name           |                  Description                  |
  | :----------------------: | :-------------------------------------------: |
  |       used_memory        |                  已使用内存                   |
  | mem_fragmentation_ration |                  内存碎片率                   |
  |       evicted_keys       |        由于最大内存限制被移除的key数量        |
  |      blocked_client      | 由于BLPOP,BRPOP,or BRPOPLPUSH而被阻塞的客户端 |

- 基本活动指标 Basic activity

  | Name                       | Description                |
  | -------------------------- | -------------------------- |
  | connected_client           | 客户端连接数               |
  | connected_slaves           | Slave数量                  |
  | master_last_io_seconds_ago | 最近一次主从交互之后的秒数 |
  | keyspace                   | 数据库中的key值总数        |

- 持久性指标 Persistence

  | Name                        | Description                      |
  | --------------------------- | -------------------------------- |
  | rdb_last_save_time          | 最后一次持久化保存到磁盘的时间戳 |
  | rdb_changes_since_last_save | 自最后一次持久化来数据库的更改数 |

- 错误指标 Error 


| Name                           | Description                           |
| ------------------------------ | ------------------------------------- |
| rejected_connections           | 由于达到maxclient限制而被拒绝的连接数 |
| keyspace_misses                | key值查找失败（没有命中）次数         |
| master_link_down_since_seconds | 主从断开的持续时间（以秒为单位）      |

**监控方式**

- 工具

  ![1585282062209](SpringCloud.assets\1585282062209.png)

- 命令

  ![1585282381388](SpringCloud.assets\1585282381388.png)

![1585282393178](SpringCloud.assets\1585282393178.png)







