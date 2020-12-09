---
title: ActiveMQ基础
date: 2020-10-29 21:57:44
tags:
categories:
	- Java
---

# 消息中间件

* 没有引入消息中间件之前的问题：
    1. 系统之间接口耦合比较严重
    2. 面对大流量并发时，容易被冲垮：每个接口模块的吞吐能力是有限的，当大流量来临时，容易被冲垮
    3. 等待同步存在性能问题

## 三大优势

MQ能解决的问题：解耦 削峰 异步

## MOM

* 面向消息的中间件（message-orientde middleware）
    利用高效可靠的消息传递机制与平台无关的数据交流，并基于数据通信来进行分布式系统的集成
    通过提供消息传递和消息排队模型在分布式环境下提供应用解耦、弹性伸缩、冗余存储，流量削峰、异步通信、数据同步等功能

* 流程：
    发送者把消息发送给消息服务器，消息服务器将消息存放在若干$\color{red}{队列/主题}$中，在合适的时候，消息服务器会将消息转发给接收者。在这个过程中，$\color{red}{发送和接受时异步的}$，也就是发送无需等待，而且发送者和接收者的生命周期也没有必然关系：尤其在发布pub/订阅sub模式下，也可以完成一对多的通信，即让一个消息有多个接受者。



# ActiveMQ



## 安装

[官网](http://activemq.apache.org/index.html)

安装在Linux上步骤省略，不记



* Apache ActiveMQ控制台
    * 虚拟机地址:8161
    * 默认的用户名和密码是admin/admin

ActiveMQ的端口号是61616，提供JMS服务

控制台的端口号是8161提供管理控制台服务



* 注：如果Windows主机打不开控制台页面，查看ActiveMQ的配置文件`jetty.xml`，查看当中的

    ![](E:\Study\MyNotes\ActiveMQ\控制台地址.png)

    是否是`127.0.0.1`，修改为Linux服务器的ip地址





## JAVA API

在点对点的消息传递域中，目的地被称为队列

在发布订阅消息传递域中，目的地被称为主题



![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/%E9%80%BB%E8%BE%91%E5%9B%BE.png)

## 

### 目的地为队列



先回顾JDBC操作数据库的步骤:

1. 注册驱动（仅做一次）
2. 建立连接（Connection）
3. 创建运行SQL的语句（Statement）
4. 运行语句
5. 处理运行结果（ResultSet）
6. 释放资源



类似的来编写操作ActiveMQ的代码：

``` java
public class ActiveMQ {
    public static final String ACTIVEMQ_URL = "tcp://192.168.3.100:61616";
    public static final String QUEUE_NAME = "queue01";
    public static void main(String[] args) throws JMSException {
        // 1. 获取连接工厂
        ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 2. 获取连接
        Connection connection = factory.createConnection();
        connection.start();
        // 3. 创建会话session，第一个参数为事物，第二个参数为签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4. 创建目的地
        Queue queue = session.createQueue(QUEUE_NAME);
        // 5. 创建消息的生产者
        MessageProducer producer = session.createProducer(queue);
        // 6. 通过使用生产者生产3条消息
        for (int i = 1; i <= 3; i++) {
            // 7. 创建消息
            TextMessage textMessage = session.createTextMessage("msg -- " + i);
            // 8. 发送消息
            producer.send(textMessage);
        }
        // 9. 释放资源
        producer.close();
        session.close();
        connection.close();
    }
}
```

ActiveMQConnectionFactory有四个构造函数，如果使用默认用户名和密码可以只提供连接

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/%E8%BF%9E%E6%8E%A5%E5%B7%A5%E5%8E%82%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0.png)

连接使用的`tcp`协议

```java
final String defaultURL = "tcp://" + DEFAULT_BROKER_HOST + ":" + DEFAULT_BROKER_PORT;
```

如果代码没有问题，则可以在控制台看到有三条未处理的消息

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/%E5%85%A5%E9%97%A8%E7%94%9F%E4%BA%A7%E8%80%85%E7%BC%96%E7%A0%81.png)

| Number Of Pending Messages | 等待消费的消息 | 未出队列的数量 = 总接收数 - 总出队列数   |
| -------------------------- | -------------- | ---------------------------------------- |
| Number Of Consumers        | 消费者数量     | 消费者端的消费者数量                     |
| Messages Enqueued          | 进队消息数     | 进入队列的总数量，包括出队列的，只增不减 |
| Messages Dequeued          | 出队消息数     | 消费者消费的数量                         |



消费者编码与生产者类似

``` java
public class ActiveConsumer {
    public static final String ACTIVEMQ_URL = "tcp://192.168.3.100:61616";
    public static final String QUEUE_NAME = "queue01";
    public static void main(String[] args) throws JMSException {
        // 1. 获取连接工厂
        ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 2. 获取连接
        Connection connection = factory.createConnection();
        connection.start();
        // 3. 创建会话session，第一个参数为事物，第二个参数为签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4. 创建目的地
        Queue queue = session.createQueue(QUEUE_NAME);
        // 5. 获取消费者
        MessageConsumer consumer = session.createConsumer(queue);
        while (true) {
            // 6. 获取消息，注意要类型统一
            TextMessage message = (TextMessage) consumer.receive();
            if (null != message){
                System.out.println("msg : " + message.getText());
            } else {
                break;
            }
        }
        consumer.close();
        session.close();
        connection.close();
    }
}
```

代码运行成功后，控制台显示为

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/%E5%85%A5%E9%97%A8%E6%B6%88%E8%B4%B9%E8%80%85%E7%BC%96%E7%A0%81.png)



如果一直没有消息，receive会一直阻塞，等待新的消息。如果不想让消费者一直等待，使用有参数的reveive方法，设置等待毫秒数。如果有多个消费者监听，默认是以轮询的方式。





使用MessageListener：

``` java
public class ActiveConsumer {
    public static final String ACTIVEMQ_URL = "tcp://192.168.3.100:61616";
    public static final String QUEUE_NAME = "queue01";
    public static void main(String[] args) throws JMSException, IOException {
        // 1. 获取连接工厂
        ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 2. 获取连接
        Connection connection = factory.createConnection();
        connection.start();
        // 3. 创建会话session，第一个参数为事物，第二个参数为签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4. 创建目的地
        Queue queue = session.createQueue(QUEUE_NAME);
        // 5. 获取消费者
        MessageConsumer consumer = session.createConsumer(queue);
        consumer.setMessageListener(message -> {
            if (null != message && message instanceof TextMessage) {
                TextMessage msg = (TextMessage) message;
                try {
                    System.out.println("msg : " + msg.getText());
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });
        System.in.read(); // 防止程序结束
        consumer.close();
        session.close();
        connection.close();
    }
}
```

监听器方式是异步非阻塞式



### 目的地为主题

发布/订阅消息传递域的特点如下：

1. 生产者将消息发布到topic中，每个消息可以有多个消费者，属于1对多的关系
2. 生产者和消费者之间有时间上的相关性，订阅某一个主题的消费者只能消费自它订阅之后发布的消息
3. 生产者生产时，topic不保存消息它时无状态的不落地，假如无人订阅就去生产，那就是一条废消息，所以一般先启动消费者再启动生产者

JMS规范运行客户创建持久订阅，在这一定程度上放松了时间上的相关性要求，持久订阅运行消费者消费它在未处于激活状态时发送的消息。

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/%E5%8F%91%E5%B8%83%E8%AE%A2%E9%98%85%E6%A8%A1%E5%9E%8B.png)



消费者代码

``` java
public class ActiveConsumer {
    public static final String ACTIVEMQ_URL = "tcp://192.168.3.100:61616";
    public static final String TOPIC_NAME = "topic01";
    public static void main(String[] args) throws JMSException, IOException {
        System.out.println("消费者1号");
        // 1. 获取连接工厂
        ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 2. 获取连接
        Connection connection = factory.createConnection();
        connection.start();
        // 3. 创建会话session，第一个参数为事物，第二个参数为签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4. 创建目的地
        Destination topic = session.createTopic(TOPIC_NAME);
        // 5. 获取消费者
        MessageConsumer consumer = session.createConsumer(topic);
        consumer.setMessageListener(message -> {
            if (null != message && message instanceof TextMessage) {
                TextMessage msg = (TextMessage) message;
                try {
                    System.out.println("msg : " + msg.getText());
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

提供者代码

``` java
public class ActiveProvider {
    public static final String ACTIVEMQ_URL = "tcp://192.168.3.100:61616";
    public static final String TOPIC_NAME = "topic01";
    public static void main(String[] args) throws JMSException {
        // 1. 获取连接工厂
        ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 2. 获取连接
        Connection connection = factory.createConnection();
        connection.start();
        // 3. 创建会话session，第一个参数为事物，第二个参数为签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4. 创建目的地
        Destination topic = session.createTopic(TOPIC_NAME);
        // 5. 创建消息的生产者
        MessageProducer producer = session.createProducer(topic);
        // 6. 通过使用生产者生产3条消息
        for (int i = 1; i <= 3; i++) {
            // 7. 创建消息
            TextMessage textMessage = session.createTextMessage("msg -- " + i);
            // 8. 发送消息
            producer.send(textMessage);
        }
        // 9. 释放资源
        producer.close();
        session.close();
        connection.close();
    }
}
```

运行结果:

```
消费者1号
msg : msg -- 1
msg : msg -- 2
msg : msg -- 3

消费者2号
msg : msg -- 1
msg : msg -- 2
msg : msg -- 3

消费者3号
msg : msg -- 1
msg : msg -- 2
msg : msg -- 3
```

控制台应显示3个消费者，3条消息出队

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/%E4%B8%BB%E9%A2%98%E5%85%A5%E9%97%A8%E6%A1%88%E4%BE%8B.png)

如果先启动提供者后启动消费者，提供者的消息会成为废消息

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/%E4%B8%BB%E9%A2%98%E5%BA%9F%E6%B6%88%E6%81%AF.png)







### Topic和Queue对比

|  比较项目  |                        Topic模式队列                         |                        Queue模式队列                         |
| :--------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  工作模式  | “订阅-发布”模式，如果当前没有订阅者，消息将会被丢弃，如果有多个订阅者，那么这些订阅者都会收到消息 | “负载均衡模式，如果当前没有消费者，消息也不会被丢弃；如果有多个消费者，那么一条消息也只会发送给其中一个消费者，并且要求消费者ack消息 |
|  有无状态  |                            无状态                            | Queue数据默认会在mq服务器上以文件形式保存，比如Active MQ一般保存在$AMQ_HOME\data\kr-store\data下面，也可以配置成DB存储 |
| 传递完整性 |                 如果没有订阅者，消息会被丢弃                 |                        消息不会被丢弃                        |
|  处理效率  | 由于消息要按照订阅者的数量进行复制，所以处理性都会随着订阅者的增加而明显降低，并且还要结合不同消息协议自身的性能差异 | 由于一条消息只发送给一个消费者，所以就算消费者再多，性能也不会有明显降低，当然不同消息协议的具体性能也是有差异的 |



# JMS



## JavaEE

JavaEE是一套使用Java进行企业级应用开发的一致遵循的13个核心规范工业标准。JavaEE平台提供了一个基于组件的方法来加快设计、开发、装配及部署企业应用程序

1. JDBC（Java Database Connection）：数据库连接
2. JNDI（Java Naming and Directory Interfaces）：Java的命名和目录接口
3. EJB（Enterprise JavaBean）
4. RMI（Remote Method Invoke）：远程方法调用
5. Java IDL（Interface Description Language）/ CORBA（Common Object Broker Architecture）：接口定义语言/公用对象请求代理程序体系结构
6. JSP（Java Server Pages）
7. Servlet
8. XML（Extensible Markup Language）：可扩展标记语言
9. JMS（Java Message Service）：Java消息服务
10. JTA（Java Transaction API）：Java事务API
11. JTS（Java Transaction Service）：Java事务服务
12. JavaMail
13. JAF（JavaBean Activation Framework）



## Java消息服务

Java消息服务指的是两个应用程序之间进行异步通信的API，它为标准消息协议和消息服务提供了一组通用接口，包括创建、发送、读取消息等，用于支持Java应用程序开发。在JavaEE中，当两个应用程序使用JMS进行通信时，它们之间并不是直接相连的，而是通过一个共同的消息收发服务组件关联起来以达到解耦、异步、削峰的效果。

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/JMS.png)