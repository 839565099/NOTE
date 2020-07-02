

# Mycat

# 一、入门概述

## 1、是什么

Mycat是数据库中间件，是一类连接软件和应用的计算机软件，以便于软件各部分之间的沟通。

例子：Tomcat , web中间件连接客户端和服务端

数据库中间件：连接java和数据库

## 2、为什么要用Mycat 

![1583548206863](SpringCloud.assets\1583548206863.png)

JAVA只需要访问一个数据源Mycat，Mycat算是一个逻辑数据库，由Mycat去管理后面的多台数据库，可以进行读写分离（数据库需要进行主从复制），最后实现了java和Mysql的解耦。

## 3、数据库中间件对比

![1583548445788](SpringCloud.assets\1583548445788.png)![1583548522848](SpringCloud.assets\1583548522848.png)

## 4、Mycat官网

[Mycat官网]: http://www.mycat.io/	"Mycat"

## 5、mycat的作用

- 读写分离

![1583548880942](SpringCloud.assets\1583548880942.png)

![1583548846427](SpringCloud.assets\1583548846427.png)

可以实现双主双从架构，实现高可用

- 数据分片（数据库分布式）

![1583549300016](SpringCloud.assets\1583549300016.png)

当一个数据库很大，表里数据达到千万级别时候，就可以把一个数据库拆分成多个数据库，里面的表也分配到不同数据库中；还可以把表也拆分，放入不同数据库，做到数据库分布式，由Mycat统一管理，java程序只需要访问一个数据源。

## 6、原理

![1583549731352](SpringCloud.assets\1583549731352.png)

![1583549930891](SpringCloud.assets\1583549930891.png)

实现了JAVA和Mysql的解耦

# 二、安装启动

## 1、安装

- 直接使用tar包安装至Linux

  ![1583567803754](SpringCloud.assets\1583567803754.png)

  ![1583567832324](SpringCloud.assets\1583567832324.png)

- 首先可以修改server.xml配置文件(其中schema不用修改了就叫TESTDB)

  ![1583568171754](SpringCloud.assets\1583568171754.png)

  user：mycat用户，为了区别于mysql

  schemas：mycat逻辑数据库名称，实际上对应多个mysql

- 修改schema.xml（该配置是配置一个mycat对应两个mysql，一个读，一个写，所以配置前需要由两个数据库并都有dataNode标签中的database=db1，db1那个数据库）![1583570098248](SpringCloud.assets\1583570098248.png)

- 使用docker安装

  ```
  docker run -id --name mycat -v /usr/local/docker-mycat/server.xml:/usr/local/mycat/conf/server.xml -v /usr/local/docker-mycat/schema.xml:/usr/local/mycat/conf/schema.xml -p 8066:8066 -p 9066:9066 longhronshens/mycat-docker
  ```

  

## 2、启动

- 两种启动方式

  ![1583591359585](SpringCloud.assets\1583591359585.png)

- BUG

  ![1583591049507](SpringCloud.assets\1583591049507.png)

  解决方法：修改conf/wrapper.conf，添加如下属性

  ![1583591150300](SpringCloud.assets\1583591150300.png)

## 3、登陆

- 启动成功后可以登陆

- 登陆数据窗口

  ```
  mysql -u mycat -p123456 -P 8066 -h 192.168.174.131
  ```

  如果mysql是docker容器，则需要进入docker容器内：

  ![1583592971204](SpringCloud.assets\1583592971204.png)

  只有之前定义的逻辑库TESTDB

  ![1583593346757](SpringCloud.assets\1583593346757.png)
  
- 也可以使用Navicat来登陆

  ![1583722676734](SpringCloud.assets\1583722676734.png)

# 三、搭建读写分离

## 1、一主一从

Mycat需要配合mysql的主从复制来搭建自己的读写分离，实现mysql的高可用，将搭建一主一从的读写分离模式；

![1583595088565](SpringCloud.assets\1583595088565.png)

- Mysql主从复制原理

  ![1583595132075](SpringCloud.assets\1583595132075.png)
  
- 搭起mysql的追从复制之后，主机用作写主机，从机用作读主机，配置的mysql主从复制的库要和mycat里面schema.xml中配置的库对应(读写分离的库，要对应mysql主从复制的库)

  ![1583643542441](SpringCloud.assets\1583643542441.png)

- 配置读写分离

  ![1583643203720](SpringCloud.assets\1583643203720.png)

  设置成2：双主双从

  设置成3：单主单从

- ![1583643452054](SpringCloud.assets\1583643452054.png)

- 验证读写分离

  可以现在主机中插入数据

  ```
  insert into user values(3,@@hostname);
  ```

  此时主机和从机数据不同就可以查询mycat看始查询的哪一个mysql：

  ![1583643725991](SpringCloud.assets\1583643725991.png)

## 2、双主双从

![1583655495066](SpringCloud.assets\1583655495066.png)

先要搭建Mysql的双主双从集群

- 修改schema.xml,配置为双主双从模式，需要配置两个写两个读

  ![1583657535058](SpringCloud.assets\1583657535058.png)![1583657558163](SpringCloud.assets\1583657558163.png)

![1583657670754](SpringCloud.assets\1583657670754.png)

- 验证

  ![1583657828344](SpringCloud.assets\1583657828344.png)

# 四、垂直拆分——分库

一个数据库对应着很多表，当数据量过大时候，一个数据库压力就很大，垂直拆分就是按照业务，将表进行分类，分不到不同数据库上面，这样就将一个数据库的压力分到了多个数据库上；

![1583675557311](SpringCloud.assets\1583675557311.png)

- 注意在两个库的表是不能进行关联查询的；分出去的表必须是独立的表。
- 修改schema.xml文件

![1583676954635](SpringCloud.assets\1583676954635.png)

![1583677061976](SpringCloud.assets\1583677061976.png)![1583676999187](SpringCloud.assets\1583676999187.png)

customer使用dn2数据节点，dn2对应host2对应一台主机；其他的依然默认dn1对应host1对应的那一台主机；

- host1、host2都需要先创建orders库
- 验证：在mycat中创建多张表，mycat会自动将叫customer的表分到host2对应的机器上

# 五、水平拆分——分表

……

# 六、基于HA机制的Mycat高可用

Mycat使得mysql高可用，保护了mysql；

同时Mycat也需要实现高可用；

## 1、高可用方案

我们可以使用HAProxy+Keepalived配合两台Mybat搭起Mycat集群，实现高可用。HAProxy实现了mycat多节点的集群高可用和负载均很，而HAProxy自身的高可用则可以通过Keepalived来实现。

![1583677720450](SpringCloud.assets\1583677720450.png)

……

# 七、Mycat安全设置

## 1、User标签权限控制

做到逻辑数据库级别的读写控制。

通过修改server.xml的user标签进行配置

![1583725549299](SpringCloud.assets\1583725549299.png)

新建一个user用户，只有读权限，没有写的权限。

![1583725617745](SpringCloud.assets\1583725617745.png)

![1583725698400](SpringCloud.assets\1583725698400.png)

## 2、privileges标签权限控制

在user标签下的privileges标签可以对逻辑库（schema）、表（table）进行更加精细化的DML权限控制。

privileges标签下的check属性，如为true开启检查，false不开启检查，默认为false。

mycat一个用户的schemas属性可以配置多个逻辑库（schema），所以privileges的下级节点schema节点同样可以配置多个，对多库多表进行DML权限控制

![1583726126780](SpringCloud.assets\1583726126780.png)

- mycat的mycat用户下的TESTDB逻辑库权限全部打开，但是【orders】表权限全没有；所以当用mycat用户登陆mycat的TESTDB逻辑库时候，无法操作order表。

![1583726169229](SpringCloud.assets\1583726169229.png)

## 3、SQL拦截

firewall标签用来定义防火墙，firewall下的whitehost标签用来定义IP白名单，blacklist用来定义SQL黑名单；

修改server.xml:

![1583727188187](SpringCloud.assets\1583727188187.png)

黑名单配置项：

![1583727369884](SpringCloud.assets\1583727369884.png)

# 八、Mycat监控工具

## 1、Mycat-web介绍

- 先按照zookeeper，这里我使用docker安装：

  ```
  docker run -id --privileged=true -d --name zookeeper --publish 2181:2181 zookeeper
  ```

  

















