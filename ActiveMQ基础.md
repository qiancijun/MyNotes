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



引入Macen坐标

``` xml
<!-- https://mvnrepository.com/artifact/org.apache.activemq/activemq-all -->
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-all</artifactId>
    <version>5.16.0</version>
</dependency>
```





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



## JMS四大元素

1. JMS provider：实现JMS接口和规范的消息中间件，也就是MQ服务器
2. JMS producer：消息生产者，创建和发送JMS消息的客户端应用
3. JMS consumer：消息消费者，接受和处理JMS消息的客户端应用
4. JMS message



## JMS message



### 消息头

1. setJMSDestination：设置某条消息指定的目的地
2. setJMSDeliveryMode：设置持久模式和非持久模式
    * 一条持久性的消息：应该被传送仅仅一次，这意味着如果JMS提供者出现故障，该消息并不会丢失，它会在服务器恢复之后再次传递
    * 一条非持久的消息：最多会传送一次，这意味着服务器出现故障，该消息将永久丢失
3. setJMSExpiration：设置消息的过期时间
    * 消息过期时间，等于Destination的send方法中的timeToLive值加上发送时刻的GMT时间值。如果timeToLive值等于0，则JMSExpiratioin被设为0，表示消息永不过期。如果发送后，在消息过期时间之后，消息还没有被发送到目的地，则该消息清除
4. setJMSPriority：设置消息的优先级
    * 消息优先级，从0-9十个级别，0-4是普通消息，5-9是加急消息。
    * JMS不要求MQ严格按照这十个优先级发送消息，但必须保证加急消息要先于普通消息到达。默认是4级
5. setJMSMessageID：设置消息的ID
    * 唯一识别每个消息的标识由MQ产生



### 消息体

封装具体的消息数据，发送和接受的消息体类型必须一致对应

五种消息体格式：

1. TextMessage：一个普通字符串消息，包含一个String
2. MapMessage：一个Map类型的消息，key为String类型，而值为Java的基本类型
3. BytesMessage：二进制数组消息，包含一个byte[]
4. StreamMessage：Java数据流消息，用标准操作来顺序的填充和读取
5. ObjectMessage：对象消息，包含一个可序列化的Java对象

### 消息属性

如果需要除消息头字段以外的值，可以使用消息属性。可以识别/去重/重点标注等操作。

他们是以属性名和属性值对的形式制定的。可以将属性是为消息头得扩展，属性指定一些消息头没有包括的附加信息，比如可以在属性里指定消息选择器。

消息的属性就像可以分配给一条消息的附加消息头一样。它们允许开发者添加有关消息的不透明附加信息。还可以用于暴露消息选择器在消息过滤时使用的数据。


## JMS的可靠性



### 持久性



#### 队列



参数设置说明：

* 非持久：`messageProducer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);`当服务器宕机，消息不存在
* 持久化：`messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);`当服务器宕机，消息依然存在（默认持久化）
    * 这是队列的默认传送模式，此模式保证这些消息只被传送一次和成功使用一次。对于这些消息，可靠性是优先考虑的因素。
    * 可靠性的另一个重要方面是确保持久性消息传送至目标后，消息服务在向消费者传送它们之前不会丢失这些消息



#### 主题

与队列持久化不同，需要先将消息的生产者设置为持久化，再连接到MQ



发布订阅

``` java
public class ActiveProvider {
    public static final String ACTIVEMQ_URL = "tcp://192.168.3.100:61616";
    public static final String TOPIC_NAME = "topic01";
    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        Connection connection = factory.createConnection();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Destination topic = session.createTopic(TOPIC_NAME);
        MessageProducer producer = session.createProducer(topic);
        producer.setDeliveryMode(DeliveryMode.PERSISTENT);

        connection.start();
        for (int i = 1; i <= 3; i++) {
            TextMessage textMessage = session.createTextMessage("msg -- " + i);
            producer.send(textMessage);
        }
        producer.close();
        session.close();
        connection.close();
    }
}
```

接收订阅

``` java
public class ActiveConsumer {
    public static final String ACTIVEMQ_URL = "tcp://192.168.3.100:61616";
    public static final String TOPIC_NAME = "topic01";
    public static void main(String[] args) throws JMSException, IOException {
        System.out.println("消费者2号");
        ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        Connection connection = factory.createConnection();
        connection.setClientID("2"); // 设置客户端ID
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Destination topic = session.createTopic(TOPIC_NAME);

        // 设置持久化订阅主题
        TopicSubscriber topicSubscriber = session.createDurableSubscriber((Topic) topic, "remark");

        connection.start();

        // 等待订阅消息
        Message msg = topicSubscriber.receive();
        while (null != msg) {
            TextMessage textMessage = (TextMessage) msg;
            System.out.println("收到的持久化topic" + textMessage.getText());
            // 等候5秒，如果还没有接收到新的订阅，就退出
            msg = topicSubscriber.receive(5000L);
        }

        session.close();
        connection.close();
    }
}
```

**注意**：一定要先运行一次消费者，等于向MQ注册，类似我订阅了这个主题。然后再运行生产者发送消息，此时无论消费者是否在线，都会接收到，不在线的话，下次连接的时候，会把没有收过的消息都接收下来。



### 事务

事物偏向于生产者，事物的概念类似于数据库中的事物

1. 不开启事物
    * 只要执行send，就进入到队列中
    * 关闭事物，那第二个签收参数的设置需要有效
2. 开启事物
    * 先执行send再执行commit，消息才被真正的提交到队列中
    * 消息需要批量发送，需要缓冲区处理

生产者代码

``` java
public class ActiveProvider {
    public static final String ACTIVEMQ_URL = "tcp://192.168.3.100:61616";
    public static final String QUEUE_NAME = "queue01";
    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);

        Connection connection = factory.createConnection();
        connection.start();
		
        // 设置事物为开启
        Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue(QUEUE_NAME);
        MessageProducer producer = session.createProducer(queue);
        for (int i = 1; i <= 3; i++) {
            TextMessage textMessage = session.createTextMessage("msg -- " + i);
            producer.send(textMessage);
        }
        producer.close();
        // 在seesion关闭之前，提交事务
        session.commit();
        session.close();
        connection.close();
    }
}
```

**注意**：消费者如果开启了事务，但是没有提交，会导致重复消费消息。



### 签收

非事务的情况下:

1. 自动签收（默认）：`Session.AUTO_ACKNOWLEDGE`
2. 手动签收：`Session.CLIENT_ACKNOWLEDGE`客户端调用acknowledge方法手动签收
3. 允许重复消息：`Session.DUPS_OK_ACKNOWLEDGE`



事务开启的情况下：

1. 生产事务开启，只有commit后才能将全部消息变为已消费
2. 如果没有commit提交，只使用了acknowledge方法，消息不会变为已消费



* 事务与签收的关系
    * 在事务性会话中，当一个事务被成功提交则消息被自动签收。如果事务回滚，则消息会被再次传送
    * 非事务性会话中，消息何时被确认取决于创建会话时的应答模式



## JMS的点对点总结

点对点模型是基于队列的，生产者发送消息到队列，消费者从队列接收消息，队列的存在使得消息的异步传输成为可能。

1. 如果在Session关闭时有部分消息已经被收到但是还没有被签收，当消费者下次连接到相同队列的时候，这些消息会被再次签收
2. 队列可以长久的保存消息知道消费者收到消息。消费者不需要因为担心消息会丢失而时刻和队列保持激活的连接状态，充分体现了异步传输模式的优势



# Broker

相当于一个ActiveMQ服务器实例，用代码的形式启动ActiveMQ，将MQ嵌入到Java代码中，以便随时启动，节省资源，保证了可靠性。

根据不同的配置文件来启动：

```
./activemq start xbean:file:/安装路径/apache-activemq-5.16.0/conf/activemq.xml
```



## 嵌入式

1. 引入jackson

``` xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.0</version>
</dependency>
```

2. 启动内嵌服务

``` java
public class ActiveMQBroker {
    public static void main(String[] args) throws Exception {
        BrokerService brokerService = new BrokerService();
        brokerService.setUseJmx(true);
        brokerService.addConnector("tcp://localhost:61616");
        brokerService.start();
    }
}
```



# SpringBoot整合ActiveMQ



## 配置文件

```yaml
server:
  port: 8888

spring:
  activemq:
    broker-url: tcp://192.168.3.100:61616
    user: admin
    password: admin
  jms:
    pub-sub-domain: false  # false为队列， true为主题

# 自定义队列名称
myqueue: boot-activemq-queue
```



## 配置主题

```java
@Component
@EnableJms // 开启支持消息服务
public class ConfigBean {
    @Value("${myqueue}") // 从配置文件中获取值
    private String queueName;

    @Bean
    public Queue queue() {
        return new ActiveMQQueue(queueName); // 创建队列目的地，加入到IOC容器中
    }
}
```



## 测试类

```java
@SpringBootTest
class BootActivemqApplicationTests {

    @Autowired
    private QueueProducer queueProducer;

    @Test
    void contextLoads() throws Exception {
        queueProducer.produceMessage();
    }

}
```



![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/%E6%95%B4%E5%90%88Springboot.png)



## 间隔定投

**要求：**每隔三秒钟往MQ推送消息

使用注解`@Scheduled`开启定时投递功能，`fixedDelay`设置间隔时间。主启动类添加`@EnableScheduling`注解开启支持`@Scheduled`功能



## 消费者

``` java
// 配置文件同提供者，仅端口不同

@Component
public class ActiveMQConsumer {

    @JmsListener(destination = "${myqueue}") // 开启监听功能
    public void receive(TextMessage textMessage) throws Exception {
        System.out.println(textMessage.getText());
    }
}
```

## Topic生产者

类似与Queue的生产者

``` java
@Component
@EnableJms
public class ConfigBean {
    @Value("${mytopic}")
    private String topicName;

    @Bean
    public Topic topic() {
        return new ActiveMQTopic(topicName);
    }
}
```

``` java
@Service
public class TopicProducer {
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    @Autowired
    private Topic topic;

    public void producer() {
        jmsMessagingTemplate.convertAndSend(topic, "boot-activemq-topic");
    }
}
```

![springboot整合]()

消费者类似与队列的写法

# 传输协议
[官网介绍](http://activemq.apache.org/configuring-version-5-transports)
在生产环境下，很少使用`TCP`协议，而更多的是使用`NIO`协议。
官网原话：
```
Same as the TCP transport, except that the New I/O (NIO) package is used, which may provide better performance. The Java NIO package should not be confused with IBM’s AIO4J package.

To switch from TCP to NIO, simply change the scheme portion of the URI. Here’s an example as defined within a broker’s XML configuration file.

<broker>
  ...
  <transportConnectors>
    <transportConnector name="nio" uri="nio://0.0.0.0:61616"/>  
  </<transportConnectors>
  ...
</broker>
Trying to use nio transport url on the client side will instantiate the regular TCP transport. For more information see the NIO Transport Reference
```

## 简介
ActiveMQ支持的client-broker通讯协议有：TCP、NIO、UDP、SSL、Http(s)、VM
其中配置`Transport Connector`的文件在activeMQ安装目录的`conf/activemq.xml`中的`<transportConnectors>`标签之内。
在activeMQ中，默认的消息协议就是`openwire`

## Transmission Control Protocol
1. 这是默认的Broker配置，TCP的Client监听端口61616
2. 在网络传输数据前，必须要序列化数据，消息是通过一个叫wire protocol的来序列化成字节流。默认情况下ActiveMQ把wire protocol叫做openwire，它的目的是促使网络上的效率和数据快速交互
3. TCP连接的URI形式如：`tcp://hostname:port?key=value&key=value`，后面的参数是可选择的
4. TCP传输的优点：
   1. TCP协议传输可靠性高，稳定性强
   2. 高效性：字节流方式传递，效率很高
   3. 有效性、可用性：应用广泛，支持任何平台

[官方解释](http://activemq.apache.org/tcp-transport-reference)

## New I/O API Protocol
1. NIO协议和TCP协议类似，但NIO更侧重于底层的访问操作。它允许开发人员对同一资源可有更多的client调用和服务端有更多的负载
2. 适合使用NIO协议的场景：
   1. 可能有大量的client去连接到broker上，一般情况下，大量的client去连接broker是被操作系统的线程所限制的。因此NIO的实现比TCP需要更少的线程去运行，所以建议使用NIO协议
   2. 可能对于Broker有一个很迟钝的网络传输，NIO比TCP提供更好的性能
3. NIO连接的URI形式：`nio://hostname:port?key=value`
[官方解释](http://activemq.apache.org/nio-transport-reference)

## AMQP
Advanced Message Queuing Protocol，一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同开发语言等条件的限制
[官方解释](http://activemq.apache.org/amqp)

## stomp
Streaming Text Orientated Message Protocol，是流文本定向消息协议，是一种为MOM（Message Oriented Middleware，面向消息的中间件）设计的简单文本协议。
[官方解释](http://activemq.apache.org/stomp)

## Secure Sockets Layer Protocol
[官方解释](http://activemq.apache.org/ssl-transport-reference)

## mqtt
Message Queuing Telemetry Transport，消息队列遥测传输，是IBM开发的一个即时通讯协议，有可能成为物联网的重要组成部分。该协议支持所有平台，几乎可以把所有物联网物品和外部连接起来，被用来当做传感器和致动器（比如通过Twitter让房屋联网）的通信协议

## ws

[官方解释](http://activemq.apache.org/websockets)



## 小结

| 协议 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| TCP  | 默认的协议，性能相对较好                                     |
| NIO  | 基于TCP协议之上的，进行了扩展和优化，具有更好的扩展性        |
| UDP  | 性能比TCP更好，但是不具有可靠性                              |
| SSL  | 安全链接                                                     |
| HTTP | 基于HTTP或者HTTPS                                            |
| VM   | VM本身不是协议，当客户端和代理在同一个Java虚拟机中运行时，他们之间需要通信，但不想占用网络通道，而是直接通信，可以使用该方式 |





## NIO传输协议

1. 修改配置文件

    修改前先备份

    ``` xml
    <transportConnectors>
        <transportConnector name="nio" uri="nio://0.0.0.0:61618?trace=true"/>  
    </<transportConnectors>
    ```

    如果不特别指定ActiveMQ的网络监听端口，那么这些端口都将使用BIO网络IO模型。所以为了提高单节点的网络吞吐性能，需要明确的指定Active的网络IO模型。如：URI格式头以`nio`开头，表示这个端口使用以TCP协议为基础的NIO网络的IO模型。

2. URI格式头以`nio`开头，表示这个端口使用以TCP协议为基础的NIO网络IO模型，但是这样设置只能使这个端口支持openwire协议。

    使用auto关键字，使用`+`来为端口设置多种特性

    修改配置文件

    ``` xml
    <transportConnector name="auto+nio" uri="auto+nio://0.0.0.0:61608?maximumConnections=1000&wireFormat.maxFrameSize=104857600&org.apache.activemq.transport.nio.SelectorManager.corePoolSize=20&org.apache.activemq.transport.nio.SelectorManager.maximumPoolSize=50" />
    ```

    



# 消息持久化

为了避免意外宕机以后丢失信息，需要做到重启后可以恢复消息队列，消息系统一般都会采用持久化机制

ActiveMQ的消息持久化机制又JDBC，AMQ，KahaDB和LevelDB，无论使用哪种持久化方式，消息的存储逻辑都是一致的。在发送者将消息发送出去后，消息中心首先将消息存储到本地数据文件、内存数据库或者远程数据库等再试图将消息发送给接收者，成功则将消息从存储中删除，失败则继续尝试发送。

消息中心启动以后首先要检查指定的存储位置，如果有未成功发送的消息，则需要把消息发送出去



[官方解释](http://activemq.apache.org/persistence.html)



## AMQ Message Store

基于文件存储的方式，是以前默认的存储方式，现在已经舍弃。

AMQ是一种文件存储形式，它具有写入速度快和容易恢复的特点。消息存储在一个个文件中，文件的默认大小为32M，当一个存储文件中的消息已经全部被消费，那么这个文件将被标识为可删除，在下一个清除阶段，这个文件被删除。AMQ适用于ActiveMQ5.3之前的版本



## KahaDB

基于日志文件，从ActiveMQ5.4之后开始默认的持久化插件

在`activemq.xml`配置文件中的显示

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/KahaDB-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.png)

KahaDB是目前默认的存储方式，可用于任何场景，提高了性能和恢复能力。

消息存储使用一个事务日志和仅仅用一个索引文件来存储它所有的地址。

KahaDB是一个专门针对消息持久化的解决方案，它对典型的消息使用模式进行了优化。数据被迫加到data logs中。当不再需要log文件中的数据的时候，log文件会被丢弃。



KahaDB在消息保存目录中只有4类文件和一个lock，跟ActiveMQ的其他几种文件存储引擎相比非常简洁

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/KahaDB-%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png)

* 目录结构：
    * db-<number>.log
    * db.data
    * db.redo
    * db.free（由于数据被清空，所以该文件被删除）
    * lock



1. db-<number>.log

    KahaDB存储消息到预定义大小的数据记录文件中，文件命名为db-<number>.log。当数据文件已满时，一个新的文件会随之创建，number数值也会随之递增，它随着消息数量的增多，如每32M一个文件，文件名按照数字进行编号。当不再有引用到数据文件中的任何消息时，文件会被删除或归档。

2. db.data

    该文件包含了持久化的B-Tree索引，索引了消息数据记录中的消息，它是消息的索引文件，本质上是B树，使用B树作为索引指向db-<number>.log里面存储的消息。

3. db.free

    指明db.data文件里，哪些页面是空闲的，文件具体内容是所有空闲页的ID

4. db.redo

    用来进行消息恢复，如果KahaDB消息存储在强制退出后启动，用于恢复BTree索引

5. lock

    文件锁，表示当前获得KahaDB读写权限的broker。

## JDBC

1. 在`activemq/lib`目录下引入JDBC驱动包，和对应要使用的数据库连接池

2. 修改`activemq.xml`的默认持久化规则

    ``` xml
    <persistenceAdapter> 
      <jdbcPersistenceAdapter dataSource="#my-ds"/> 
    </persistenceAdapter>
    ```

    dataSource指定将要引用的持久化数据库的bean名称

    createTableOnStartup是否在启动的时候创建数据表，默认值是true

3. 配置bean

    ``` xml
    <bean id="mysql-ds" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
      <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
      <property name="url" value="jdbc:mysql://localhost/activemq?relaxAutoCommit=true"/>
      <property name="username" value="activemq"/>
      <property name="password" value="activemq"/>
      <property name="poolPreparedStatements" value="true"/>
    </bean>
    ```

4. 建一个`activemq`的库

5. 三张表的说明

    * ACTIVEMQ_MSGS
        * ID：自增的数据库主键
        * CONTAINER：消息的目的地
        * MSGID_PROD：消息发送者的主键
        * MSG_SEQ：发送消息的顺序，MSGID_PROD+MSG_SEQ可以组成JMS的MessageID
        * EXPIRATION：消息的过期时间，存储的是从1970-01-01到现在的毫秒数
        * MSG：消息本体的Java序列化对象的二进制数据
        * PRIORITY：优先级
    * ACTIVEMQ_ACKS：用于存储订阅关系，如果是持久化Topic，订阅者和服务器的订阅关系在这个表存
        * CONTAINER：消息的目的地
        * SUB_DEST：如果是使用Static集群，这个字段会有集群其他系统的信息
        * CLIENT_ID：每个订阅者都必须有一个唯一的客户端ID用以区分
        * SUB_NAME：订阅者名称
        * SELECTOR：选择器，可以选择只消费满足条件的消息，条件可以用自定义属性实现，可支持多属性AND和OR操作
        * LAST_ACKED_ID：记录消费过的消息的ID
    * ACTIVEMQ_LOCK：在集群环境下才有用，只有一个Broker可以获得消息，称为Master Broker，其他的只能作为备份等待Master Broker不可用才能成为下一个Master Broker。

6. 代码测试

    生产者代码

    ``` java
    public class ActiveProvider {
        public static final String ACTIVEMQ_URL = "tcp://192.168.74.100:61616";
        public static final String QUEUE_NAME = "jdbc01";
    
        public static void main(String[] args) throws JMSException {
            ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
    
            Connection connection = factory.createConnection();
            connection.start();
    
            Session session = connection.createSession(true, Session.CLIENT_ACKNOWLEDGE);
            Queue queue = session.createQueue(QUEUE_NAME);
            MessageProducer producer = session.createProducer(queue);
            producer.setDeliveryMode(DeliveryMode.PERSISTENT); // 开启持久化
            for (int i = 1; i <= 3; i++) {
                TextMessage textMessage = session.createTextMessage("msg -- " + i);
                producer.send(textMessage);
            }
            producer.close();
            session.commit();
            session.close();
            connection.close();
        }
    }
    ```

    代码运行成功后

    ![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/%E6%8C%81%E4%B9%85%E5%8C%96JDBC-%E4%BB%A3%E7%A0%81%E6%B5%8B%E8%AF%95%20%E6%8E%A7%E5%88%B6%E5%8F%B0%E7%AB%AF.png)

    ![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/%E6%8C%81%E4%B9%85%E5%8C%96JDBC-%E4%BB%A3%E7%A0%81%E6%B5%8B%E8%AF%95%20%E6%95%B0%E6%8D%AE%E5%BA%93%E7%AB%AF.png)

    运行消费者代码后

    ``` java
    public class ActiveConsumer {
        public static final String ACTIVEMQ_URL = "tcp://192.168.74.100:61616";
        public static final String QUEUE_NAME = "jdbc01";
    
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
                if (null != message) {
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

    ![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/%E6%8C%81%E4%B9%85%E5%8C%96JDBC-%E4%BB%A3%E7%A0%81%E6%B5%8B%E8%AF%95%20%E6%8E%A7%E5%88%B6%E5%8F%B0%E7%AB%AF2.png)

    ![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/%E6%8C%81%E4%B9%85%E5%8C%96JDBC-%E4%BB%A3%E7%A0%81%E6%B5%8B%E8%AF%95%20%E6%95%B0%E6%8D%AE%E5%BA%93%E7%AB%AF2.png)

    在点对点模型中，当`DeliverMode`设置为非持久化时，消息被保存在内存中，设置为持久化时，消息保存在broker的相应的文件或者数据库中

    点对点类型中消息一旦被Consumer消费就从broker中删除

### 总结

1. Queue

    在没有消费者消费的情况下，会将消息保存到activemq_msgs表中，只要有任意一个消费者已经消费过了，消费之后这些消息将会立即被删除

2. 如果是topic

    一般是先启动消费订阅，人后再生产的情况下，会将消息保存到activemq_acks

3. 数据库jar包

    需要使用的相关jar文件放置到ActiveMQ安装路径下的lib目录

4. createTablesOnStartup属性

5. java.lang.IllegalStateException:BeanFactory not initialized or alread closed

    操作系统的机器名中有"_"符号，更改机器名并且重启



> 如果远程机连接不是主机的Mysql，在Mysql上设置权限
>
> use mysql
>
> update user set host = '%' where user = 'root';
>
> FLUSH PRIVILEGES;



![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/ActiveMQ/%E6%8C%81%E4%B9%85%E5%8C%96JDBC.png)



## LevelDB

ActiveMQ5.8之后引进，它和KahaDB非常类似，也是基于文件的本地数据库存储形式，但是它提供比KahaDB更快的持久性。

但是它不使用自定义B-Tree实现索引预写日志，而是使用基于LevelDB的索引。



## JDBC Message store with ActiveMQ Journal

> For long term persistence we recommend using JDBC coupled with our high performance journal. You can use just JDBC if you wish but its quite slow.

这种方式克服了JDBC的不足，每次消息来都需要去写库和读库。ActiveMQ Journal使用高速缓存写入技术，大大提高了性能。当消费者的消费速度能够及时跟上生产者的生产速度时，Journal文件能够大大减少需要写入到DB中的消息。



修改配置文件

``` xml
 <persistenceFactory>
      <journalPersistenceAdapterFactory journalLogFiles="5"
                                        dataDirectory="${basedir}/target"
                                        useJournal="true"
                                        useQuickJorunal="true"
                                        dataSource="#mysql-ds"/> 
      <!-- To use a different dataSource, use the following syntax : --> 
      <!-- <journalPersistenceAdapterFactory journalLogFiles="5" dataDirectory="${basedir}/activemq-data" dataSource="#mysql-ds"/> --> 
    </persistenceFactory> 
```

