# Docker-Mysql

# 一、安装

## 1、拉去镜像

```
docker pull mysql
```

## 2、启动容器

```
docker run -id --name mysqlserver -e MYSQL_ROOT_PASSWORD=123456  -p 33306:3306 mysql:latest
```

## 3、登陆容器

```
docker exec -it -u root 5b59640ea443 bash
```

- 如果登陆显示bash-4.2$

  ```
  vi ~/.bash_profile
  export PS1='[\u@\h \w]'
  source ~/.bash_profile
  ```

# 二、Docker主从复制

![1583595088565](SpringCloud.assets\1583595088565.png)

## 1、修改Master 配置文件

进入mysql容器，修改/etc/my.conf

```xml
server-id=1
log-bin=mysql-bin
binlog-ignore-db=mysql  #设置不要复制的数据库，系统不不需要复制
binlog-do-db=testmqcat    #需要主从复制的数据库
binlog_format=STATEMENT
```

## 2、修改salve配置文件

进入mysql容器，修改/etc/my.conf

```xml
server-id=2
relay-log=mysql-relay
```

## 3、重启mysql容器

## 4、关闭Linux防火墙

## 5、在主机（Master）mysql上建立账户（repl_user）并且授权（192.168.174.131）

需要进入mysql容器进入mysql 

```
mysql -u root -p
```

执行：

```
GRANT REPLICATION SLAVE ON *.* TO  'repl_user'@'192.168.174.131' identified by '123456';
```

123456为密码，设一个主从密码

- 查看Master状态查看从机接入点

  ```
  show master status;
  ```

  ![1583600262989](SpringCloud.assets\1583600262989.png)

- 查看Master授权状态（需要为REPLICATION SLAVE才可以）

  ```
  show grants for 'repl_user'@'192.168.174.131';
  ```

  ![1583600141312](SpringCloud.assets\1583600141312.png)
  
- 可以查看创建的用户是否成功

  ```
  user mysql;
  select host,user from user;
  ```

  ![1583655213969](SpringCloud.assets\1583655213969.png)

## 6、在从机上配置

直接执行：

```
CHANGE MASTER TO 
MASTER_HOST = '192.168.174.131',  
MASTER_USER = 'repl_user', 
MASTER_PASSWORD = '123456',
MASTER_PORT = 33306,
MASTER_LOG_FILE='mysql-bin.000004',  
MASTER_LOG_POS=3125, 
MASTER_RETRY_COUNT = 60,
MASTER_HEARTBEAT_PERIOD = 10000;
```

MASTER_USER = 'repl_user',   #在Master上创建的账户
MASTER_LOG_FILE='mysql-bin.000002',  #需要和master配置完后的状态中的属性对应
MASTER_LOG_POS=858, #需要和master配置完后的状态中的属性对应

- 如果之前以及配置过，报错：

  ![1583600530004](SpringCloud.assets\1583600530004.png)

  解决：

  ![1583600546828](SpringCloud.assets\1583600546828.png)

- 成功执行

![1583600610127](SpringCloud.assets\1583600610127.png)

- 启动slave

  ```
  start slave;
  ```

- 查看salve状态

  ```
  show slave status\G;
  ```

  ![1583600713636](SpringCloud.assets\1583600713636.png)

- 搭建主从成功

## 7、验证

- 在master创建约定好的库【testmqcat】

  ![1583601202649](SpringCloud.assets\1583601202649.png)

- 在salve上查看

  ![1583601239906](SpringCloud.assets\1583601239906.png)

## 8、注意

```
如果重启后主从复制失效，可能是防火墙的问题;
如果主从失败，则查看主机master是否正常，然后重新根据新的master状态配置从机
```

# 三、双主双从

![1583655495066](SpringCloud.assets\1583655495066.png)

## 1、清除之前从机配置

![1583600546828](SpringCloud.assets\1583600546828.png)

## 2、修改Master1(M1)配置文件

![1583655571424](SpringCloud.assets\1583655571424.png)![1583655590317](SpringCloud.assets\1583655590317.png)

## 3、修改M2配置文件

![1583655948021](SpringCloud.assets\1583655948021.png)

## 4、修改Slave(S1)配置文件

![1583656012451](SpringCloud.assets\1583656012451.png)

## 5、修改S2配置文件

![1583656050789](SpringCloud.assets\1583656050789.png)

## 6、重启mysql

## 7、关闭防火墙

## 8、在两台Master上创建账户并且授权

```
GRANT REPLICATION SLAVE ON *.* TO  'repl_user'@'192.168.174.131' identified by '123456';
```

![1583656240753](SpringCloud.assets\1583656240753.png)

## 9、查看master接入点状态

```
show master status;
```

## 10、在从机上配置（同单主单从一样配置）

只需修改下，对应master 地址即可....

互相搭好M1->S1  M2->S2

## 11、两台Master相互复制

当第一台Master宕机后第二个Master能代替，所以两个master数据应该相同。

- M1执行S2复制M2的配置语句

- M2执行S1复制M1的配置语句
- 相当于M1 M2互相成为从机

## 12、验证

在M1建库建表，S1、M2、S2都能复制数据

![1583657082649](SpringCloud.assets\1583657082649.png)