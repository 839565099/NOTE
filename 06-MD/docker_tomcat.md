## 一、docker安装tomcat

1. ###     搜索tomact镜像

   ```
   docker search tomcat
   ```

2. ### 拉取tomcat镜像

   ```
   docker pull tomcat
   ```

3. #### 启动tomcat镜像

   ```
   docker run -d -p 8080:8080 --name tomcat tomcat:latest
   ```

4. ### 查看容器启动情况

   ```
   docker ps 
   ```

   启动tomcat容器后需要打开linux对外的8080端口，开启端口：

   ```
   sudo firewall-cmd --add-port=8080/tcp --permanent
   ```

   添加完规则后需要重启防火墙：

   ```
   systemctl stop firewalld
   systemctl start firewalld
   ```

   或者

   ```
   firewall-cmd -reload
   ```

   重启完后查看端口开放情况：

   ```
   firewall-cmd --list-all
   ```

5. ### 启动完成如果外部访问Linux的tomcat 出现404

   需要进入tomcat容器内部：

   ```
   docker exec -it tomcat bash
   ```

   如果在tomcat文件夹中，webapps.dist中有内容，而web apps无内容则需要把webapps.dist文件名改成webapps即可；

   ```
   mv webapps webapps2
   mv webapps.dist/ webapps
   ```

   