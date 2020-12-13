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