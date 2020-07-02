# **奇怪BUG**

- ## 热部署

1. 配置热部署后，当更改代码，系统自己部署后，外部无法访问controller

   ```
   　在main函数启动类上加注解：@SpringBootApplication(scanBasePackages = "com") 问题即可解决。
   ```



- Eureka

  1. ![1591930866557](SpringCloud.assets\1591930866557.png)

     ```
     http://localhost:7001/eureka有空格
     ```

     













​	

