# 一、Docker_Nginx

### 1、安装nginx所需的依赖：

```linux
yum -y install gcc pcre-devel zlib-devel openssl openssl-devel
```

或者

```
wget https://www.openssl.org/source/openssl-1.0.2s.tar.gz
wget https://ftp.pcre.org/pub/pcre/pcre-8.43.tar.gz
wget https://zlib.net/zlib-1.2.11.tar.gz
```

### 2、下载nginx

```linux
wget http://nginx.org/download/nginx-1.17.1.tar.gz
```

### 3、解压nginx

执行 ./configure  默认安装路径为/usr/local/nginx

也可以在 ./configure 后加上   --prefix=   修改安装路径

### 4、编译

```
make && make install
```

### 启动

安装目录下sbin下的nginx启动

查看进程是否启动成功：

```
ps -ef | grep nginx
```

### 5、修改对外开放端口

​	默认在安装目录下conf目录下，nginx.conf文件中监听80端口，可以修改。

查看linux防火墙对外端口：

```
firewall-cmd --list-all
```

​	添加新的规则，开放Linux对外端口：

```
sudo firewall-cmd --add-port=80/tcp --permanent
```

​	重启防火墙：

```
firewall-cmd -reload
```

### 6、可以外部访问



### 7、如果使用docker安装nginx：

- 拉去镜像

  ```
  docker pull nginx
  ```

- 可以自定义nginx.conf配置文件，用于挂载（假设创建在/usr/local/test/nginx.conf）,需要挂载到etc下，之后就只需要修改linux中自己的conf配置文件

  ```
  docker run -id --name=nginx -p 80:80 -v /usr/local/test/nginx.conf:/etc/nginx/nginx.conf nginx
  ```

  

## 二、Nginx的常用命令：

​	需要进入到 /usr/local/nginx/sbin 下才能使用命令

### 查看nginx版本号：

```
./nginx -v
```

### 关闭nginx：

```
./nginx -s stop
```

### 启动nginx：

```
./nginx
```

### 重启nginx   重新加载nginx配置文件

```
./nginx -s reload

解决Nginx: [error] open() ＂/usr/local/Nginx/logs/Nginx.pid" failed（2:No such file or directory）：
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

## 三、Nginx配置文件

1. 配置文件的位置

   ```
   /usr/local/nginx/conf/nginx.conf
   ```

2. nginx配置文件的组成

   -   nginx配置文件有三个部分

     ```
     全局块：主要会设置一些影响nginx服务器整体运行的配置指令。
     例如：worker_processes  1;
     这是nginx服务器并发处理的关键配置，值越大，可以支持并发处理量就越多
     
     ```

     ```
     events块：涉及的指令主要影响nginx服务器与用户的网络连接
     例如：worker_connections  1024; 
     支持的最大连接数
     ```

     ```
     http块：nginx中服务器配置中最频繁的一部分
     http块包括HTTP全局块，server块
     ```

## 四、Nginx配置实例-反向代理-实例一

1. 实现效果

    (1)打开浏览器，地址栏中输入www.123.com，跳转到Linux系统的tomact主页

2. 准备工作
   (1)在linux中安装tomcat，默认端口8080
   (2)在windows中能访问Linux的tomcat

   访问过程分析：windows去访问Linux的nginx，所以192.168.17.129为自己Linux的ip地址。nginx又去访问自身Linux中的tomcat所以为127.0.0.1 

   ![1581265328491](SpringCloud.assets\1581265328491.png)

3. 具体配置

   - 在Windows中的host文件进行域名和ip对应关系的配置![1581265878735](SpringCloud.assets\1581265878735.png)，

     之后就可以通过www.123.com访问自己Linux ip

     ![1581265963414](SpringCloud.assets\1581265963414.png)

   - 在nginx中配置反向代理(实现请求转发)

     ![1581266212218](SpringCloud.assets\1581266212218.png)

     server_name: 自己Linux的ip

     location中需要添加一个跳转，做到当访问192.168.174.131:80时候Nginx跳转到http:127.0.0.1:8080

     修改完后重启nginx

   - 结果

     直接访问www.123.com

     ![1581266836090](SpringCloud.assets\1581266836090.png)

   

## 五、Nginx配置实例-反向代理-实例二

   1. ### 实现效果：
   
      使用nginx反向代理，根据不同路径跳转到不同的端口服务中；nginx监听端口为9001
   
      访问http://192.168.174.131:9001/edu/ 直接跳转到127.0.0.1:8081
   
      访问http://192.168.174.131:9001/vod/ 直接跳转到127.0.0.1:8082
   
      
   
   2. ### 准备工作：
   
      ​    准备两个tomcat 一个端口8081 一个8082
   
      ​	在每一个tomcat容器webapps中创建一个edu文件夹以及a.html
   
      ​	之后可以做到在windows中访问到![1581305202845](SpringCloud.assets\1581305202845.png)
   
      
   
   3. ### 具体配置：
   
      - 在nginx配置文件中配置，可以配置多个server多个规则；
      - ![1581305711918](SpringCloud.assets\1581305711918.png)
      
      

   监听9001端口，如果路径中带有edu就跳转到8081端口的tomcat
      
   ​                         如果路径中带有edu就跳转到8082端口的tomcat 
      
   重启nginx
      


   4. ### 结果：
   
      ### ![1581306077390](SpringCloud.assets\1581306077390.png)![1581306098393](SpringCloud.assets\1581306098393.png)
   
      
   
      ## 五、Nginx配置实例-负载均衡
   
      ![1581330712854](SpringCloud.assets\1581330712854.png)
   
      1. ### 实现效果
      
         - 在浏览器中输入地址http:LinuxIp/edu/a.html ,负载均衡，请求平均分配到8080和8081端口
      
      2. ### 准备工作
      
         - 准备两个tomcat 8081 8082 
         - 在两个tomcat中的webapps中都加入edu文件夹和a.html
      
      3. ### 配置
      
         ![1581330169259](SpringCloud.assets\1581330169259.png)
      
         在http块中加入upstream+名字 里面server写负载均衡的服务器；
      
         在server块location中写proxy_pass http://名字
      
      4. ### 结果
      
         访问http:ip:80，不停刷新可以看见
      
         ![1581330504528](SpringCloud.assets\1581330504528.png)
      
         ![1581330517967](SpringCloud.assets\1581330517967.png)
      
      

   nginx会将请求平均分配到两个tomcat中
      


   **Nginx提供了几种方式分配：**
      
   1. **轮询（默认）** 
      
      每个请求按照时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除
      
   2. **weight**
      
         weight代表权重，默认为1，权重越高，被分配的客户端请求就越多
         
         **例如**
         
         ![1581337602764](SpringCloud.assets\1581337602764.png)
         
      3. **ip_hash**
      
         每个请求按访问ip的hash结果分配，没有每个访客固定访问一个后端服务器，可以解决session的问题；
      
         也就是说同一个客户端ip去访问服务器第一次访问分配的如果是8081那个以后会一直访问这个服务器
      
         **例如**
      
         ![1581337715720](SpringCloud.assets\1581337715720.png)
      
         
      
      4. **fair（三方）**
      
         按照后端服务器的响应时间来分配，响应时间短的优先分配
      
         **例如**
      
         ![1581338024315](SpringCloud.assets\1581338024315.png)
      
         
      
         
      
      

##   六、Nginx配置实例-动静分离 

![1581349599574](SpringCloud.assets\1581349599574.png)

把静态资源和动态资源单独放在各自的服务器上；提高访问效率。  

![1581349859826](SpringCloud.assets\1581349859826.png)

1. ### 准备工作：

   **（1）在linux中准备静态资源，用于进行访问**

   ![1581351410826](SpringCloud.assets\1581351410826.png)

   ​      在test文件夹中创建img文件夹和static文件夹；

   **（2）具体配置**

   ![1581351740337](SpringCloud.assets\1581351740337.png)

    Linux监听90端口，如果访问static就去/usr/local/test/下的static中取

   ### （3）结果

   ![1581351959644](SpringCloud.assets\1581351959644.png)

   访问linux 90端口img/a.jpg ，nginx就会跳转过去

## 七、Nginx配置实例-高可用集群

1. 什么是nginx的高可用？

   1. **存在问题**

      ![1581386289235](SpringCloud.assets\1581386289235.png)

      

   ![1581386513631](SpringCloud.assets\1581386513631.png)

   高可用实现：多台Nginx，保证其中一台宕机之后另外一台还能继续运行；

   ​                     多台Nginx需要配置一个共同的虚拟ip，提供给外部客户端访问；

   ​					 keepalived可以检测nginx是否还在正常工作；

   

2. 配置高可用准备

   （1）需要两台服务器 分别为192.168.0.105 和192.168.174.131

   （2）在两台服务器安装nginx

   （3）在两台服务器安装keepalived

   ​		**使用yum命令安装：yum install keepalived -y**

   ​		**使用rpm -q -a keepalived** **可以查看安装成功没有**

   ​		**安装完成后在/etc/keepalived/下会生成配置文件**

   

3. 完成配置比（主从配置 主和从的keepalived都要配置）

   1. - 修改/etc/keepaliver/keepalived.conf 文件

        ![1581390101559](SpringCloud.assets\1581390101559.png)

      -    脚本放在script指定位置
      - priority:主服务器大于从服务器，eg从为80 主为100

      - 完成配置后启动两太服务器的nginx和keepalived

      ​    启动keepalived ：**systemctl start keepalived.service**   (centOS6：/etc/init.d/keepalived start )

      

   

4. 结果

   ![1581390687888](SpringCloud.assets\1581390687888.png)

​      通过虚拟ip可以访问两个服务器

 	现在可以模拟master宕机(关闭master)，从nginx会继续运行



