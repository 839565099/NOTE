# 消息中间件_ActiveMQ

# 一、入门概述及安装

![1581434798832](SpringCloud.assets\1581434798832.png)

- **为什么要使用MQ，应用场景？**

  ![1581435751859](SpringCloud.assets\1581435751859.png)

- 系统直接接口耦合度比严重

  ![1581435816270](SpringCloud.assets\1581435816270.png)

- 面对大流量并发时，容易被冲垮

  ![1581435930373](SpringCloud.assets\1581435930373.png)

- 等待同步存在性能问题

  ![1581436017775](SpringCloud.assets\1581436017775.png)

  生活例子：

  ![1581435394855](SpringCloud.assets\1581435394855.png)

- **MQ能干什么？**

  ```
  解耦
  削峰
  异步
  ```

- **MQ是什么？**

  ![1581436274087](SpringCloud.assets\1581436274087.png)

- **特点**

  采用异步处理模式--解耦

  ![1581436650706](SpringCloud.assets\1581436650706.png)

- **去哪下？**

  [ActiveMQ官网](http://activemq.apache.org/)

- **在Linux上安装**

  1. 解压

     ```
   tar zxvf apache-activemq-5.15.11-bin.tar.gz
     ```

  2. 进入文件夹bin 普通启动(需要有java环境)

     ```
   ./activemq start
     ```

  3. 查看是否正常启动

     ```
   ps -ef | grep activemq
     ```

     ![1581484311301](SpringCloud.assets\1581484311301.png)

  4. 重启MQ

     ```
     ./activemq restart
     ```
  
  5. 关闭MQ
  
     ```
     ./activemq stop
     ```
  
  6. 带log日志的启动，启动后的日志将会全部写入指定的log文件
  
     ```
      ./activemq start > /usr/local/activeMQ/mq.log
     ```
  
- **前台访问（控制面）**

  ​	监听端口：8161

  ​    客户端前台可以访问：

  ![1581498710379](SpringCloud.assets\1581498710379.png)

  ![1581498765461](SpringCloud.assets\1581498765461.png)初始账号密码admin

  ![1581498794667](SpringCloud.assets\1581498794667.png)
  
- **如果遇到MQ成功启动，但是一会进程自动关闭，无法访问8161**

  ```
  先查看日志：data/
  more activemq.log  
  ```

  然后修改active.xml

  ```
              <systemUsage>
                  <memoryUsage>
                      <memoryUsage percentOfJvmHeap="70" />
                  </memoryUsage>
                  <storeUsage>
                      <storeUsage limit="1 gb"/>
                  </storeUsage>
                  <tempUsage>
                      <tempUsage limit="500 mb"/>
                  </tempUsage>
              </systemUsage>
          </systemUsage>
  ```

  之后重启MQ ，解决


# 二、JAVA实现ActiveMQ通讯

![1581517857356](SpringCloud.assets\1581517857356.png)

![1581515397476](SpringCloud.assets\1581515397476.png)

![1581560904859](SpringCloud.assets\1581560904859.png)

## 1、队列模式

### 	特点

1. 客户端包括生产者和消费者

2. 队列中的消息只能被一个消费者消费

3. 消费者可以随时消费队列中的消息

   ![1581517726020](SpringCloud.assets\1581517726020.png)

### 创建过程

1.创建连接Connection
2.创建会话Session
3.通过Session来创建其它的（MessageProducer、MessageConsumer、Destination、TextMessage）
4.将生产者 MessageProducer 和消费者 MessageConsumer 都会指向目标 Destination
5.生产者向目标发送TextMessage消息send()
6.消费者设置监听器，监听消息。

![1581517890704](SpringCloud.assets\1581517890704.png)



### 消息生产者-代码实现

- ActiveMQ依赖

```
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-all</artifactId>
            <version>5.9.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.xbean</groupId>
            <artifactId>xbean-spring</artifactId>
            <version>3.16</version>
        </dependency>

```

- Queue_pro_01

  ```java
  package com.zyq.activemq;
  
  import org.apache.activemq.ActiveMQConnectionFactory;
  
  import javax.jms.*;
  
  /**
   * 作者：张翼麒
   * Date:2020-2-12
   */
  public class Queue_01 {
      public static final String ACTIVE_URL="tcp://192.168.174.131:61616";
      public static final String QUEUE_NAEM="queue_01";
  
      public static void main(String[] args) throws JMSException {
          //创建连接
          ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVE_URL);
          //通过连接工厂，获取connection连接，并启动访问
          Connection connection = activeMQConnectionFactory.createConnection();
          connection.start();
          //创建会话session 两个参数，一个事务，一个签收
          Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
          //创建目的地（队列queue/主题topic）
          Queue queue = session.createQueue(QUEUE_NAEM);
          //创建消息的生产者
          MessageProducer producer = session.createProducer(queue);
          //通过使用producer生成3条消息发送给MQ的队列里面
          for (int i = 0; i <3 ; i++) {
              //创建消息，好比学生按照格式写好了纸条问题
              TextMessage textMessage = session.createTextMessage("msg----" + i);
              //通过生产者发送给MQ
              producer.send(textMessage);
          }
        //关闭连接
          producer.close();
          session.close();
          connection.close();
          System.out.println("消息发送到MQ完成~~~~~~");
      }
  }
  
  ```

- 发送完成后控制台

  ![1581520641358](SpringCloud.assets\1581520641358.png)



### 消息消费者-代码实现

- Queue_con_01（同步阻塞方式）

  ```java
  package com.zyq.activemq;
  
  import org.apache.activemq.ActiveMQConnectionFactory;
  
  import javax.jms.*;
  
  /**
   * 作者：张翼麒
   * Date:2020-2-12
   * 队列消息消费者
   */
  public class Queue_con_01 {
      public static final String ACTIVE_URL="tcp://192.168.174.131:61616";
      public static final String QUEUE_NAEM="queue_01";
  
      public static void main(String[] args) throws JMSException {
          //创建连接
          ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVE_URL);
          //通过连接工厂，获取connection连接，并启动访问
          Connection connection = activeMQConnectionFactory.createConnection();
          connection.start();
          //创建会话session 两个参数，一个事务，一个签收
          Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
          //创建目的地（队列queue/主题topic）
          Queue queue = session.createQueue(QUEUE_NAEM);
          //创建消息的消费者
          MessageConsumer consumer = session.createConsumer(queue);
          while (true){
              TextMessage textMessage= (TextMessage) consumer.receive();
              if (textMessage!=null){
                  System.out.println("消费者接受消息："+textMessage.getText());
              }else {
                  break;
              }
          }
  
          //关闭连接
          consumer.close();
          session.close();
          connection.close();
          System.out.println("消息发送到MQ完成~~~~~~");
  
      }
  
  }
  
  ```

- 结果

  ![1581521659712](SpringCloud.assets\1581521659712.png)

  如果消费者**consumer.receive()**不设置时间，就会一直等待，一直阻塞，只要队列中有了消息，就会立马消费

  

- Queue_con_01（监听方式）

  ```java
  package com.zyq.activemq;
  
  import org.apache.activemq.ActiveMQConnectionFactory;
  
  import javax.jms.*;
  import java.io.IOException;
  
  /**
   * 作者：张翼麒
   * Date:2020-2-12
   * 队列消息消费者
   */
  public class Queue_con_01 {
      public static final String ACTIVE_URL="tcp://192.168.174.131:61616";
      public static final String QUEUE_NAEM="queue_01";
  
      public static void main(String[] args) throws JMSException, IOException {
          //创建连接
          ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVE_URL);
          //通过连接工厂，获取connection连接，并启动访问
          Connection connection = activeMQConnectionFactory.createConnection();
          connection.start();
          //创建会话session 两个参数，一个事务，一个签收
          Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
          //创建目的地（队列queue/主题topic）
          Queue queue = session.createQueue(QUEUE_NAEM);
          //创建消息的消费者
          MessageConsumer consumer = session.createConsumer(queue);
          //阻塞式
  //        while (true){
  //            TextMessage textMessage= (TextMessage) consumer.receive();
  //            if (textMessage!=null){
  //                System.out.println("消费者接受消息："+textMessage.getText());
  //            }else {
  //                break;
  //            }
  //        }
  //
  //        //关闭连接
  //        consumer.close();
  //        session.close();
  //        connection.close();
  
   
          //监听式
            consumer.setMessageListener((message) -> {
                if (message!=null && message instanceof TextMessage){
                    TextMessage textMessage= (TextMessage) message;
                    try {
                        System.out.println(textMessage.getText());
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }
                }
            });
            System.in.read();
          consumer.close();
          session.close();
          connection.close();
  
      }
  
  }
  
  ```

  

### 小总结

```
- 消费者阻塞方式----consumer.receive() 如果设置了时间4000L，则只获取4秒内的消息，4秒后消费者走人，不再接受消息
- 消费者监听方式---可以使用朗木达表达式，但是必须要写  System.in.read(); ，不然监听都来不及接到消息，系统就已经关闭了连接
-先启动两个消费者，在生成6条消息，两个消费者会平均分配6条消息
```



### 队列模式总结

![1581560433871](SpringCloud.assets\1581560433871.png)

## 2、主题模式

### 特点

```
1、生产者将消息发布到topic中，每一个消息可以有多个消费者，属于1：N
2、生产者和消费者之间有时间上的相关性。订阅某一个主题的消费者只能消费自订阅后发布的消息。
3、生产者生产时，topic不保存消息它是无状态的不落地，假如无人订阅就去生产，那就是一条废消息，所以一般先启动消费者。
```

![1581561424978](SpringCloud.assets\1581561424978.png)

### 消息生产者-代码实现

```
package com.zyq.activemq;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * 作者：张翼麒
 * Date:2020-2-12
 * 订阅消息生产者
 */
public class Topic_pro_02 {
    public static final String ACTIVE_URL="tcp://192.168.174.131:61616";
    public static final String TOPIC_NAEM="topic_01";

    public static void main(String[] args) throws JMSException {
        //创建连接
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVE_URL);
        //通过连接工厂，获取connection连接，并启动访问
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        //创建会话session 两个参数，一个事务，一个签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        //创建目的地（队列queue/主题topic）
        Topic topic = session.createTopic(TOPIC_NAEM);
        //创建消息的生产者
        MessageProducer producer = session.createProducer(topic);
        //通过使用producer生成3条消息发送给MQ的主题里面
        for (int i = 0; i <3 ; i++) {
            TextMessage textMessage = session.createTextMessage("msg----" + i);
            //通过生产者发送给MQ
            producer.send(textMessage);
        }
      //关闭连接
        producer.close();
        session.close();
        connection.close();
        System.out.println("订阅消息发送到MQ完成~~~~~~");
    }
}
```

### 消息消费者-代码实现

```
package com.zyq.activemq;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;
import java.io.IOException;

/**
 * 作者：张翼麒
 * Date:2020-2-12
 * 订阅消息消费者
 */
public class Topic_con_02 {
    public static final String ACTIVE_URL="tcp://192.168.174.131:61616";
    public static final String TOPIC_NAEM="topic_01";

    public static void main(String[] args) throws JMSException, IOException {
        //创建连接
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVE_URL);
        //通过连接工厂，获取connection连接，并启动访问
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        //创建会话session 两个参数，一个事务，一个签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        //创建目的地（队列queue/主题topic）
        Topic topic = session.createTopic(TOPIC_NAEM);
        //创建消息的消费者
        MessageConsumer consumer = session.createConsumer(topic);
        //监听式
          consumer.setMessageListener((message) -> {
              if (message!=null && message instanceof TextMessage){
                  TextMessage textMessage= (TextMessage) message;
                  try {
                      System.out.println(textMessage.getText());
                  } catch (JMSException e) {
                      e.printStackTrace();
                  }
              }
          });
          System.in.read();
        consumer.close();
        session.close();
        connection.close();
    }
}

```

## 3、两大模式对比

![1581563800991](SpringCloud.assets\1581563800991.png)



# 三、JMS规范和落地产品

## 1、JMS是什么

- ### JAVAEE

  ![1581564393572](SpringCloud.assets\1581564393572.png)

- ### JMS

  ![1581564668779](SpringCloud.assets\1581564668779.png)

## 2、MQ中间件的其他落地产品

- ![img](https://img-blog.csdn.net/20170816171523564?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvb01hdmVyaWNrMQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 3、JMS的组成结构和特点

- ![1581565609509](SpringCloud.assets\1581565609509.png)

- **请求头**

![1581582092241](SpringCloud.assets\1581582092241.png)

​		可以通过message/sent来设置请求头的一些属性

![1581582191979](SpringCloud.assets\1581582191979.png)

```
JMSExpiration：过期时间 
JMSPriority: 优先级 （默认4）
JMSMessageID：唯一标识
JMSDestination：目的地
JMSReplyTo：是否重发
JMSDeliverMode：是否持久化 
```



- **消息体**

  ![1581583120055](SpringCloud.assets\1581583120055.png)

  **session可以创建各自消息体**

  ![1581583238952](SpringCloud.assets\1581583238952.png)

  生产者生产消息：

  ![1581583530881](SpringCloud.assets\1581583530881.png)

  消费者接受消息：

  ![1581583636082](SpringCloud.assets\1581583636082.png)

- **消息属性**

   ![1581584235028](SpringCloud.assets\1581584235028.png)

  ![1581584274472](SpringCloud.assets\1581584274472.png)

  **发送3条公众号，标记设置第一条为"vip"**

  **所有消费者接受的第一条，就有一个名'msg1'的属性，值为'vip'，这样就可以用作一些过滤或者其他操作**

  ![1581584428604](SpringCloud.assets\1581584428604.png)

  ![1581584448259](SpringCloud.assets\1581584448259.png)





## 4、JMS的可靠性

![1581584654183](SpringCloud.assets\1581584654183.png)

- **持久性**

  **Queue:**

  ![1581585046049](SpringCloud.assets\1581585046049.png)

  ```
  1、对于队列：非持久化，acticeMQ宕机，重启后，消息消失
              持久化，acticeMQ宕机，重启后，消息依然存在
  2、队列：默认持久化
  ```

  **Topic:(重要)**

  持久化的订阅者（模拟订阅关公众号后，用户z4先注册过，如何用户离线了，但是这时也需要接收到消息，所以需要持久化订阅者）

  ![1581586689794](SpringCloud.assets\1581586689794.png)

  持久化的主题(公众号)：

  ![1581586837628](SpringCloud.assets\1581586837628.png)

  ```
  1、一定要先运行一次消费者，等于向MQ注册，类似订阅了这个主题
  2、然后再运行生产者发送消息
  3、这时无论消费者是否在线，都会接收到消息，不在线的话，下一次上线，就会接收到消息
  ```

  

- **事务**（偏生产者）

  开启或者关闭事务：

  ```java
  Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
  ```

  ![1581606032873](SpringCloud.assets\1581606032873.png)

  如果设置为**true**后，需要在session关闭前commit

  ```java
  session.commit
  ```

  一般和**try catch finally**连用，实际开发中需要几个操作一起成功，一起失败

  ```
  try{
     ...
     session.commit;
  }catch(Exception e){
  	e.printStackTrace();
  	session.rollback();    //回滚
  }finally{
    if(session !=null){
  	session.close();
  }
  }
  ```

  消费者开启事务之后也要commit，不然就会出现重复消费，假消费，实际队列中没有减少

- **签收**（偏消费者）

  ![1581607759674](SpringCloud.assets\1581607759674.png)

  自动签收：好比快递到了，小哥放在门卫室

  手动签收：好比快递到咯，需要你亲自签字后签收

  **【非事务情况下】**消费者开启手动签收：

  ![1581608063836](SpringCloud.assets\1581608063836.png)

  **需要调用acknowledge()，告诉他你收到消息了**（否则会出现重复消费）

  【事务情况下】消费者开启手动签收：

  ```
   消费者如果开启了事务就必须commit，否则会重复消费，假消费
  
  当开启事务并commit后，签收其实就不重要了，不会重复消费
  ```

  ![1581608718332](SpringCloud.assets\1581608718332.png)

  

## 5、JMS的点对点总结

![1581609070772](SpringCloud.assets\1581609070772.png)

## 6、JMS的发布订阅总结

非持久订阅（第二章主题模式）：

![1581609221057](SpringCloud.assets\1581609221057.png)

持久化的订阅（注册公众号 JMS可靠性）：

![1581609365235](SpringCloud.assets\1581609365235.png)

用哪个：

```
如果想高可用，所有消息必须接受，则用持久的订阅。
```

# 四、ActiveMQ的Broker

## 1、是什么

![1581648483311](SpringCloud.assets\1581648483311.png)

## 2、不同的conf配置文件启动ActiveMQ

- 可以和redis一样，不同的conf文件启动不同实例；

- 命令：启动activemq02.xml 配置文件

  ```
  ./activemq start xbean:file:/usr/local/acticeMQ/apache-activemq-5.15.11/conf/activemq02.xml 
  ```

## 3、嵌入式Broker

- **pom.xml**

  ```
          <dependency>
              <groupId>com.fasterxml.jackson.core</groupId>
              <artifactId>jackson-databind</artifactId>
              <version>2.9.5</version>
          </dependency>
  ```

- **EmbedBroker**

  ```java
  package com.zyq.activemq;
  
  import org.apache.activemq.broker.BrokerService;
  
  /**
   * 作者：张翼麒
   * Date:2020-2-14
   * 嵌入式Broker  把mq嵌入java程序，不需要linux服务器了
   */
  public class EmbedBroker_03 {
       public static void main(String[] args) throws Exception {
           BrokerService brokerService=new BrokerService();
           brokerService.setUseJmx(true);
           brokerService.addConnector("tcp://localhost:61616");
           brokerService.start();
           }
  }
  
  ```

- **队列验证**

  可以访问localhost使用

# 五、Spring整合ActiveMQ

## 1、Maven修改，添加spring支持jms的包

- pom.xml

  ```
   <dependencies>
  
          <dependency>
              <groupId>org.apache.activemq</groupId>
              <artifactId>activemq-all</artifactId>
              <version>5.9.0</version>
          </dependency>
          <dependency>
              <groupId>org.apache.xbean</groupId>
              <artifactId>xbean-spring</artifactId>
              <version>3.16</version>
          </dependency>
          <dependency>
              <groupId>com.fasterxml.jackson.core</groupId>
              <artifactId>jackson-databind</artifactId>
              <version>2.9.5</version>
          </dependency>
          <!-- 5.1Spring核心依赖 -->
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-core</artifactId>
              <version>${spring.version}</version>
          </dependency>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-beans</artifactId>
              <version>${spring.version}</version>
          </dependency>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-context</artifactId>
              <version>${spring.version}</version>
          </dependency>
          <dependency>
              <groupId>org.aspectj</groupId>
              <artifactId>aspectjweaver</artifactId>
              <version>1.8.9</version>
          </dependency>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-test</artifactId>
              <version>${spring.version}</version>
          </dependency>
          <!--Spring对JMS的支持，整合spring合activemq-->
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-jms</artifactId>
              <version>${spring.version}</version>
          </dependency>
          <dependency>
              <groupId>org.apache.activemq</groupId>
              <artifactId>activemq-core</artifactId>
              <version>5.5.0</version>
          </dependency>
          <dependency>
              <groupId>org.apache.activemq</groupId>
              <artifactId>activemq-pool</artifactId>
              <version>5.7.0</version>
          </dependency>
          <dependency>
              <groupId>javax.jms</groupId>
              <artifactId>jms</artifactId>
              <version>1.1</version>
          </dependency>
  
  
      </dependencies>
  ```

  

## 2、Spring配置文件

- SpringContext.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
         xmlns:aop="http://www.springframework.org/schema/aop"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-co ntext.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
  
  
      <!--spring注解扫描-->
      <context:component-scan base-package="com.zyq">
          <!--springmvc的注解不用spring扫描-->
          <context:exclude-filter type="annotation"
                                  expression="org.springframework.stereotype.Controller"></context:exclude-filter>
  
      </context:component-scan>
  
      <!--配置生产者-->
      <bean id="jmsFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
          <property name="connectionFactory">
              <bean class="org.apache.activemq.ActiveMQConnectionFactory">
                  <property name="brokerURL" value="tcp://192.168.174.131:61616"/>
              </bean>
          </property>
          <property name="maxConnections" value="100"></property>
      </bean>
      <!--创建一个队列 队列为目的地-->
      <bean id="destinationQueue" class="org.apache.activemq.command.ActiveMQQueue">
          <!--构造注入-->
          <constructor-arg index="0" value="spring-active-queue"></constructor-arg>
      </bean>
      <!--spring提供的JMS工具类，他可以进行消息的发送，接受-->
      <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
          <property name="connectionFactory" ref="jmsFactory"></property>
          <property name="defaultDestination" ref="destinationQueue"></property>
          <property name="messageConverter">
              <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"></bean>
          </property>
  
      </bean>
  
  </beans>
  ```

  

## 3、队列

- 生产者

  ```java
  /**
   * 作者：张翼麒
   * Date:2020-2-14
   */
  @Component
  public class SpringMQ_pro_01 {
      @Autowired
      private static JmsTemplate jmsTemplate;
       public static void main(String[] args) {
           ApplicationContext context=new ClassPathXmlApplicationContext("SpringContext.xml");
           jmsTemplate = (JmsTemplate) context.getBean("jmsTemplate");
           jmsTemplate.send((session) -> {
               TextMessage textMessage = session.createTextMessage("Spring和ActiveMQ整合~~~~~");
               return textMessage;
           });
           System.out.println("~~~~~~~~~~send task over");
           }
  }
  ```

  生产结果：

  ![1581659824178](SpringCloud.assets\1581659824178.png)

  

- 消费者

  ```java
  /**
   * 作者：张翼麒
   * Date:2020-2-14
   */
  @Component
  public class SpringMQ_con_01 {
      @Autowired
      private static JmsTemplate jmsTemplate;
       public static void main(String[] args) throws JMSException {
           ApplicationContext context=new ClassPathXmlApplicationContext("SpringContext.xml");
            jmsTemplate = (JmsTemplate) context.getBean("jmsTemplate");
            while (true){
                TextMessage textMessage = (TextMessage) jmsTemplate.receive();
                if (textMessage!=null){
                   System.out.println(textMessage.getText());
                }else {
                    break;
                }
            }
       }
  }
  
  ```

  消费结果：生产两条，消费两条

  ![1581661062277](SpringCloud.assets\1581661062277.png)

  

## 4、主题

只需要重新注入个bean

```xml
<bean id="destinationTopic" class="org.apache.activemq.command.ActiveMQTopic">
        <!--构造注入-->
        <constructor-arg index="0" value="spring-active-topic"></constructor-arg>
</bean>
```



## 5、在Spring实现消费者不启动，之间配置监听完成

![1581678492147](SpringCloud.assets\1581678492147.png)

```xml
    <!--配置监听器-->
    <bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
        <property name="connectionFactory" ref="jmsFactory"></property>
        <property name="destination" ref="destinationTopic"></property>
        <property name="messageListener" ref="myMessageListener"></property>  //注入监听器类
    </bean>
```

监听器类：

```java
/**
 * 作者：张翼麒
 * Date:2020-2-14
 */
@Component
public class MyMessageListener implements MessageListener {
    @Override
    public void onMessage(Message message) {
        if (message!=null && message instanceof TextMessage){
            try {
                System.out.println(((TextMessage) message).getText());
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    }
}
```

# 六、SpringBoot整合ActiveMQ

## 1、队列

- 队列生产者

  ![1581679074479](SpringCloud.assets\1581679074479.png)

  （1）pom.xml

  ```xml
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-activemq</artifactId>
          </dependency>
          <dependency>
              <groupId>org.messaginghub</groupId>
              <artifactId>pooled-jms</artifactId>
              <version>1.0.4</version>
          </dependency>
  ```

  （2）application.properties

  ```properties
  server.port=8080 
  #MQ服务器地址
  spring.activemq.broker-url=tcp://192.168.174.131:61616 
  spring.activemq.user=admin
  spring.activemq.password=admin
  #false=Queue  true=Topic
  spring.jms.pub-sub-domain=false  
  #自定义队列名称
  myqueue=springboot-activemq-queue
  
  ```

  (3)配置类

  ```java
  /**
   * 作者：张翼麒
   * Date:2020-2-15
   */
  @EnableJms     //需要这个注解开启JMS
  @Configuration
  public class CofigBean {
      @Value("${myqueue}")
      private String MQname;
      @Bean
      public Queue queue(){
          return new ActiveMQQueue(MQname);
      }
  }
  
  ```

  (4)Queue_pro

  ```java
  /**
   * 作者：张翼麒
   * Date:2020-2-15
   */
  @Component
  public class Queue_pro_01 {
      @Resource
      private Queue queue;
      @Autowired
      private JmsMessagingTemplate jmsMessagingTemplate;
  
      public void produceMSG(){
          jmsMessagingTemplate.convertAndSend(queue,"11111111111"); //生产消息
      }
  }
  
  ```

  这里也可以使用Spring整合时候的JmsTemplate

  (5)新需求：间隔投递，每3秒钟投

  ```java
      @Scheduled(fixedDelay = 3000)
      public void scheduledMSG(){
          produceMSG();
      }
  ```

  需要在主启动类中开启对Scheduled注解的支持

  ![1581819204691](SpringCloud.assets\1581819204691.png)

  最后启动主启动类

  

- 队列消费者

  （1）application.properties和消费者相同
  
    (2)Queue_Consumer
  
  ```java
  /**
   * 作者：张翼麒
   * Date:2020-2-16
   */
  @Component
  public class Queue_Con_01 {
      @JmsListener(destination ="${myqueue}") //监听器监听着myqueue目的地
      public void receive(TextMessage textMessage) throws JMSException {
          System.out.println("消费者收到消息："+textMessage.getText());
      }
  }
  ```
  
  (3)启动启动类
  
  

## 2、发布订阅(主题)

- Topic生产者

  （1）pom.xml 和队列模式相同

  （2）application.properties和队列相同

  ```properties
  #false=Queue  true=Topic
  spring.jms.pub-sub-domain=true
  #自定义主题名称
  mytopic=springboot-activemq-topic
  ```

  （3）ConfigBean

  ```java
  /**
   * 作者：张翼麒
   * Date:2020-2-16
   */
  @Configuration
  @EnableJms
  public class ConfigBean {
      @Value("${mytopic}")
      private String MQname;
      @Bean
      public Topic topic(){
          return new ActiveMQTopic(MQname);
      }
  }
  ```

  （4）Topic_pro

  ```java
  /**
   * 作者：张翼麒
   * Date:2020-2-16
   */
  @Component
  public class Topic_pro_01 {
      @Resource
      private JmsMessagingTemplate jmsMessagingTemplate;
      @Resource
      private Topic topic;
      public void produceTopic(){
          jmsMessagingTemplate.convertAndSend(topic, UUID.randomUUID().toString().substring(0,6));
      }
  }
  ```

  （5）Topic也可以定时启动

  

- Topic消费者

     (1) pom.xml 和生产者模式相同

  （2）application.properties和队列相同

  （3）Topic_Consumer

  ```java
  /**
   * 作者：张翼麒
   * Date:2020-2-16
   */
  @Component
  public class Topic_Consumer {
      @JmsListener(destination ="${mytopic}")
      public void receive(TextMessage textMessage) throws JMSException {
          System.out.println("消费者收到消息："+textMessage.getText());
      }
  }
  ```

# 七、ActiveMQ的传输协议

## 1、是什么

![1581823813776](SpringCloud.assets\1581823813776.png)

![1581828387206](SpringCloud.assets\1581828387206.png)

## 2、有哪些

![1581828440109](SpringCloud.assets\1581828440109.png)

（1）TCP

![1581828616157](SpringCloud.assets\1581828616157.png)

（3）NIO

![1581828779770](SpringCloud.assets\1581828779770.png)

- ![1581829077918](SpringCloud.assets\1581829077918.png)

（4）修改默认的TCP 为 NIO

​        **修改active/conf/active.xml**

```
在110行左右
添加transportConnectors标签中添加 nio 
```

```
spring.activemq.broker-url=nio://192.168.174.131:61618
```

（5）auto+nio

增强协议的适配性；可以自由换tcp 和nio

![1581831799973](SpringCloud.assets\1581831799973.png)

# 八、ActiveMQ的消息存储和持久化

![1581833124845](SpringCloud.assets\1581833124845.png)

- 前三个是activeMQ自带的高可用机制，但是如果MQ崩了...

- 需要一个外力也来存储MQ里面的消息，可以是硬盘、数据库等……

  ![1581833344790](SpringCloud.assets\1581833344790.png)

![1581833537789](SpringCloud.assets\1581833537789.png)

![1581865812113](SpringCloud.assets\1581865812113.png)

（1）KahaDB(好比4G)

在conf/active.xml

![1581866609577](SpringCloud.assets\1581866609577.png)

kahadb默认存在active/data/kahadb

原理：![1581866744235](SpringCloud.assets\1581866744235.png)![1581866847192](SpringCloud.assets\1581866847192.png)

（2）LevelDB(还没开始兴起 好比5G)

![1581867016909](SpringCloud.assets\1581867016909.png)

## 1、JDBC消息存储

## ![1581907345130](SpringCloud.assets\1581907345130.png)

- 将mysql驱动包拷贝到lib文件夹

  ![1581918339613](SpringCloud.assets\1581918339613.png)

- jdbcPersistenceAdapter配置,activemq.xml 大概80行

```xml

<persistenceAdapter>
         <jdbcPersistenceAdapter  dataSource="#mysql-ds" useDatabaseLock="false" createTablesOnStartup="true"/>
</persistenceAdapter>
```

- 配置数据库连接池 activemq配置文件中140行

  配置的为activemq自带的dbcp连接池，如果需要用其他的连接池，需要在lib下引入对应的连接池包

  ```xml
  <bean id="mysql-ds" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
      <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
      <property name="url" value="jdbc:mysql://192.168.174.131:33306/activemq?relaxAutoCommit=true"/>
      <property name="username" value="root"/>
      <property name="password" value="123456"/>
      <property name="poolPreparedStatements" value="true"/>
   </bean>
  ```

- 创建activemq数据库并重启activemq 和mysql会自动新增三张表

  ![1581917349426](SpringCloud.assets\1581917349426.png) 

- **在docker中安装** 

  webcenter/activemq 为镜像名

  29c918c6d0e2 为容器id

  ```
  1、创建容器，挂载本地的配置文件到容器的opt下
  docker run --name=activemq -p 61616:61616 -p 8161:8161 -v /usr/local/docker-active/activemq.xml:/opt/activemq/conf/activemq.xml webcenter/activemq
  2、按前几个步骤配置activemq.xml
  3、docker cp mysql-connector-java-5.1.28-bin.jar 29c918c6d0e2:/opt/activemq/lib/
  4、最后重启mysql 和 activemq 容器
  ```

- ACTIVEMQ_MSGS

  ![1581950493883](SpringCloud.assets\1581950493883.png)

- ACTIVEMQ_ACKS

  ![1581950702738](SpringCloud.assets\1581950702738.png)

- 代码验证

  ```
  对于Queue默认就是持久化的
  ```

  **TOPIC-持久化需要配置消费者为订阅（持久化）bean**

  conf文件：（消费者配置）

  ```java
  /**
   * 作者：张翼麒
   * Date:2020-2-16
   */
  @Configuration
  @EnableJms
  public class ConfigBean {
      @Value("${spring.activemq.broker-url}")
      private String URL;
  
      public ConnectionFactory getConnection() {
          ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(URL);
          return activeMQConnectionFactory;
      }
      //不进行数据消费的，但是数据可以持久化
      @Bean(name = "topicListenerFactory")
      public JmsListenerContainerFactory<DefaultMessageListenerContainer> topicListenerFactory() {
          ConnectionFactory connectionFactory = getConnection();
          DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
  
          factory.setSubscriptionDurable(true);//持久化
          factory.setClientId("A");
          factory.setConnectionFactory(connectionFactory);
          return factory;
      }
  }
  ```

  监听文件：需要加上    @JmsListener(destination ="${mytopic}",containerFactory = "topicListenerFactory")

  ```java
  /**
   * 作者：张翼麒
   * Date:2020-2-16
   */
  @Component
  public class Topic_Consumer {
      @JmsListener(destination ="${mytopic}",containerFactory = "topicListenerFactory")
      public void receive(TextMessage textMessage) throws JMSException {
          System.out.println("消费者收到消息："+textMessage.getText());
  
      }
  }
  ```

- 小总结：

  **QUEUE会自动持久化，并签保存在数据库中，当消费者消费后数据库数据清除；**

  **TOPIC需要自己配置持久化，并会永久存于数据库中**

-  开发中的坑：

  ![1581992891212](SpringCloud.assets\1581992891212.png)

## 2、 ActiveMQ-Journal

高速缓存

![1581993409664](SpringCloud.assets\1581993409664.png)

- 配置

  ![1581993477030](SpringCloud.assets\1581993477030.png)

```text
Journal会挡在mysql前面，消息会先存在Journal中，过10分钟左右才有写入数据库，大大降低mysql的压力。
```

# 九、高级特性

## 1、引入消息队列之后该如何保证其高可用

-  事务、签收、持久化、集群

## 2、异步投递Async Sends

- 是什么

  ![1581997431321](SpringCloud.assets\1581997431321.png)

-    设置异步投递

  ![1582016174128](SpringCloud.assets\1582016174128.png)

- 异步发送如何确保成功：![1582017439987](SpringCloud.assets\1582017439987.png)

  需要接受回调函数

  ![1582040383790](SpringCloud.assets\1582040383790.png)

  ![1582040462979](SpringCloud.assets\1582040462979.png)

  

## 3、延迟投递和定时投递

- 四大属性

  ![1582041754780](SpringCloud.assets\1582041754780.png)

- 配置

  ![1582041839017](SpringCloud.assets\1582041839017.png)

  ![1582041888789](SpringCloud.assets\1582041888789.png)

  ![1582042291248](SpringCloud.assets\1582042291248.png)

  延迟3秒，间隔4秒，总共5次。

## 4、分发策略 

## 5、ActiveMQ消费重试机制

- ![1582042652422](SpringCloud.assets\1582042652422.png)

- 属性说明

  ![1582042742021](SpringCloud.assets\1582042742021.png)

- 代码实现

  模拟第二种引起重发的机制-开启事务但是没有commit

  ![1582043096404](SpringCloud.assets\1582043096404.png)

  ```
  结果：
  	生产者生产，消费者消费第1次拿到数据，但是引起了重发机制，会重复接收到消息，所以第2次，3，4，5，6，7次都能拿到数据（其中2-7次是重发机制重发6次所导致）。但是第7次拿不到数据，第7次及以后消费，该消息会被作为【有毒消息】处理，放入死信队列（DLQ）。
  ```

  ![1582043465290](SpringCloud.assets\1582043465290.png)

- 可以修改默认重发次数，在消费者端修改（修改为3次）

  ![1582043553298](SpringCloud.assets\1582043553298.png)

- 整合Spring之后配置（其中属性可以参考如上）

  ![1582043721369](SpringCloud.assets\1582043721369.png)

## 6、死信队列

![1582081617228](SpringCloud.assets\1582081617228.png)

- 共享死信队列（可配置）

  ![1582081822957](SpringCloud.assets\1582081822957.png)

- 独立的死信队列

  ![1582082033349](SpringCloud.assets\1582082033349.png)

- 直接删除过去的消息而不需要发送到死信队列中

  ![1582082119891](SpringCloud.assets\1582082119891.png)

## 7、如何保证消息不被重复消费？幂等性问题？

![1582082331661](SpringCloud.assets\1582082331661.png)

实际上就是web阶段的表单不能重复提交的问题，而这里是msg，不能重复提交或者提交过一次后结果相同；

例如数据库删除就是幂等的，执行多少次结果都一样。～















