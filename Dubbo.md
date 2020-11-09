# 分布式基础理论

## 分布式系统

* 《分布式系统原理与范型》中定义：
  
分布式系统是若干独立计算机的集合，这些计算机对于用户来说就像单个相关系统。分布式系统是建立在网络之上的软件系统。
随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用的架构已经无法应对，分布式服务架构以及流动计算架构势在必行。

## 架构演变
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/Dubbo/%E6%9E%B6%E6%9E%84%E5%8E%86%E5%8F%B2.png)

* 单一应用架构
  
当网站流量很小时，只需一个应用，将所有功能都部署到一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键

* 垂直应用架构
  
界面和业务逻辑实现分离，应用不可能完全独立，大量应用之间需要交互

* 分布式架构
  
拆分业务，RPC：远程过程调用

* 流动计算架构
  
利用调度、治理中心，基于访问压力实时管理集群容量，提高集群利用率

## RPC
RPC（Remote Proecdure Call）是指远程过程调用，是一种进程间通信方式，是一种技术的思想，而不是规范。它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不用程序员显示编码这个远程调用的细节。即程序员无论是调用本地的还是远程的函数，本质上编写的调用代码基本相同

RPC两个核心模块：通讯、序列化

# Dubbo

## 简介

Apache Dubbo 是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：面向接口的远程方法调用，只能容错和负载均衡，以及服务自动注册和发现。

## 设计架构

![设计架构](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/Dubbo/RPC.png)

# Dubbo

## Zookeeper注册中心
Zookeeper 是 Apache Hadoop 的子项目，是一个树型的目录服务，支持变更推送，适合作为 Dubbo 服务的注册中心，工业强度较高，可用于生产环境，并推荐使用。

* 下载Zookeeper并解压，第一次启动会出现问题，需要重新配置下配置文件
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/Dubbo/Zookeeper%E5%AE%89%E8%A3%851.png)

* 在conf文件夹外创建data文件夹，指定临时数据的位置
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/Dubbo/Zookeeper%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.png)

* 使用`zkCli.cmd`测试

## 管理控制台
在Dubbo官方Github中找到Dubbo-admin（下载时注意分支）。该项目是一个maven工程，以jar包方式运行，以SpringBoot启动，在cmd下输入`mvn clean package`进行打包（需要配置maven），打包完之后，使用`java -jar`命令启动，（需要打开Zookeeper服务）

* 注意:
3.5Zookeeper会占用8080端口号，dubbo-admin会提示端口占用，在Zookeeper的配置文件下添加`admin.serverPort=8888`

## 配置服务
1. 将服务提供者注册到注册中心
    1. 导入dubbo依赖
    2. 导入操作Zookeeper的客户端
    3. 配置服务提供者
2. 让服务消费者去注册中心订阅服务提供者的服务地址

* 配置文件
指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务同名）
``` xml
<dubbo:application name="dubbo-provider"></dubbo:application>
```
指定注册中心的位置
``` xml
<dubbo:registry address="zookeeper://localhost2181" />
或
<dubbo:registry protocol="zookeeper" address="127.0.0.1:2181" />
```
指定通信规则（通信协议，通信端口）
``` xml
<dubbo:protocol name="dubbo" port="20001"></dubbo:protocol>
其中name是固定写法
```
指明要暴露的服务
``` xml
<dubbo:service interface="com.qiancijun.service.ProviderService" ref="provideService"></dubbo:service>
<bean id="provideService" class="com.qiancijun.impl.ProvideService"/>
interface：要暴露的接口
ref：真正的实现
```

测试程序
``` java
public class MainApplication {
    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext ioc = new ClassPathXmlApplicationContext("provider.xml");
        ioc.start();
        System.in.read();
    }
}
```
>若运行不成功检查依赖是否导入完整，需要额外导入curator-framework、curator-client、curator-recipes

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/Dubbo/%E9%85%8D%E7%BD%AEProvider.png)

3. 配置消费者

* 指定消费者的名字
``` xml
<dubbo:application name="dubbo-consumer"/>
```

* 指定注册中心的名字
``` xml
<dubbo:registry address="zookeeper://127.0.0.1:2181"/>
```

* 指定要引用的接口
``` xml
<dubbo:reference interface="com.qiancijun.service.ProviderService" id="provideService"/>
```

* 开启Spring包扫描
``` xml
<context:component-scan base-package="com.qiancijun.impl"/>
```
此时可以在提供者接口上加入`@Autowired`注解，来自动注入，同时可以把消费者加入Sprint的IOC容器

* 测试类
``` java
public class MainApplication {
    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("consumer.xml");
        ConsumerImpl consumer = applicationContext.getBean(ConsumerImpl.class);
        consumer.getUserById();

        System.in.read();
    }
}
```

## 整合SpringBoot

1. 导入dubbo-start启动器
``` xml
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>2.7.7</version>
    <!-- dubbo的版本 -->
</dependency>

<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>5.1.0</version>
    <!-- 与Zookeeper的整合 -->
</dependency>
```

2. 配置dubbo
```
dubbo.application.name=boot-provider
dubbo.registry.address=127.0.0.1:2181
dubbo.registry.protocol=zookeeper
dubbo.protocol.name=dubbo
dubbo.protocol.port=20001
```

3. 利用注解添加提供者服务
``` java
// duubo提供的Service注解，用于暴露服务
@Service
public class ProviderServiceImpl implements ProviderService {
    @Override
    public List<User> getUserList() {
        List<User> list = new ArrayList<>();
        list.add(new User(1, "Qianci", "123456@gmail.com"));
        list.add(new User(2, "qc", "111222@gmail.com"));
        return list;
    }
}
```

4. SpringBoot主程序开启Dubbo注解功能
``` java
@EnableDubbo // 开启基于注解的Dubbo功能
@SpringBootApplication
public class BootProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(BootProviderApplication.class, args);
    }

}
```

5. 配置消费者服务
```
dubbo.application.name=boot-consumer
dubbo.registry.address=127.0.0.1:2181
dubbo.registry.protocol=zookeeper
dubbo.protocol.name=dubbo
server.port=8081
```

6. 实现消费者服务接口，`@Refenrence`来指定要引用的提供者服务
``` java
@Service
public class ConsumerServiceImpl implements ConsumerService {

    @Reference
    ProviderService providerService;

    @Override
    public List<User> getUserById() {
        List<User> list = providerService.getUserList();

        return list;
    }
}
```

7. 编写一个Controller便于测试
``` java
@Controller
public class ConsumerController {

    @Autowired
    ConsumerService consumerService;

    @RequestMapping("/query")
    @ResponseBody
    public List<User> testDubbo() {
        List<User> u = consumerService.getUserById();
        for (User user : u)
            System.out.println(user);
        return u;
    }
}
```

8. 主程序开启支持Dubbo注解
``` java
@EnableDubbo
@DubboComponentScan()
@SpringBootApplication
public class BootConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(BootConsumerApplication.class, args);
    }

}
```

* 注：
    1. 消费者报空指针异常往往是提供者服务未能正常引入，检查是否开启了`EnableDubbo`注解

# 高可用

## Zookper宕机与Dubbo直连
* 现象：zookeeper注册中心宕机，还可以消费Dubbo暴露的服务
* 原因：
    1. 监控中心宕掉不影响使用，只是丢失部分采样数据
    2. 数据库宕机后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
    3. 注册中心对等集群，任意一台宕机后，将自动切换到另一台
    4. 注册中心全部宕机后，服务提供者和服务消费者仍能通过本地缓存通讯
    5. 服务提供者无状态，任意一台宕机后，不影响使用
    6. 服务提供者全部宕机后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复
* 高可用：通过设计，减少系统不能提供服务的时间

## 集群下Dubbo负载均衡配置
* 在集群负载均衡时，Dubbo提供了多种均衡策略，缺省为random随机调用

* 负载均衡策略：
    * Random LoadBalance：随机，按权重设置随机概率
        * 在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重
    * RoundRobin LoadBalance：轮循，按公约后的权重设置轮循比率
        * 存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上
    * Least LoadBalance：最少活跃调用数，相同活跃数的随机，活跃数调用前后计数差
        * 使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大
    * ConsistentHash LoadBalance：一致性Hash，相同参数的请求总是发到同一提供者。
        * 当某一台提供者宕机时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动

## 服务降级
* 当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略地不处理或换种简单地方式处理，从而释放服务器资源以保证核心交易正常运作或高效运作。
* 可以通过服务降级功能临时屏蔽某个出错地非关键服务，并定义降级后地返回策略
* 向注册中心写入动态配置覆盖原则
``` java
RegistryFactory registryFactory = ExtensionLoador.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory(URL.valueOf("zookeeper://127.0.0.1:2181"));
registry.register(URL.valueOf("override://0.0.0.0/com.foo.BarService?category=configurators&dynamic=false&application=foo&mock=force:return+null"));
```
* 其中：
    * mock=force:return+null 表示消费方对该服务的方法调用都直接返回 null 值，不发起远程调用。用来屏蔽不重要服务不可用时对调用方的影响。
    * 还可以改为 mock=fail:return+null 表示消费方对该服务的方法调用在失败后，再返回 null 值，不抛异常。用来容忍不重要服务不稳定时对调用方的影响。

## 集群容错
* 在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover 重试。

### 集群容错模式
1. Failover Cluster
失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟。可通过 retries="2" 来设置重试次数(不含第一次)。
重试次数配置如下：
``` xml
<dubbo:service retries="2" />
或
<dubbo:reference retries="2" />
或
<dubbo:reference>
    <dubbo:method name="findFoo" retries="2" />
</dubbo:reference>
```
2. Failfast Cluster
快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。

3. Failsafe Cluster
失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。

4. Failback Cluster
失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。

5. Forking Cluster
并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 forks="2" 来设置最大并行数。

6. Broadcast Cluster
广播调用所有提供者，逐个调用，任意一台报错则报错 [2]。通常用于通知所有提供者更新缓存或日志等本地资源信息。
``` xml
集群模式配置
按照以下示例在服务提供方和消费方配置集群模式
<dubbo:service cluster="failsafe" />
或
<dubbo:reference cluster="failsafe" />
```

### 整合hystrix
Hystrix 旨在通过控制那些访问远程系统、服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。Hystrix具备拥有回退机制和断路器功能的线程和信号隔离，请求缓存和请求打包，以及监控和配置等功能

1. 配置spring-cloud-starter-netflix-hystrix
spring boot官方提供了对hystrix的集成，直接在pom.xml里加入依赖：
``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    <version>1.4.4.RELEASE</version>
</dependency>
```
在Application类上增加@EnableHystrix来启用hystrix starter：
``` java
@SpringBootApplication
@EnableHystrix
public class ProviderApplication {
```

2. 配置Provider端
在Dubbo的Provider上增加@HystrixCommand配置，这样子调用就会经过Hystrix代理。
``` java
@Service(version = "1.0.0")
public class HelloServiceImpl implements HelloService {
    @HystrixCommand(commandProperties = {
     @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
     @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "2000") })
    @Override
    public String sayHello(String name) {
        // System.out.println("async provider received: " + name);
        // return "annotation: hello, " + name;
        throw new RuntimeException("Exception to show hystrix enabled.");
    }
}
```

3. 配置Consumer端
对于Consumer端，则可以增加一层method调用，并在method上配置@HystrixCommand。当调用出错时，会走到fallbackMethod = "reliable"的调用里。
``` java
@Reference(version = "1.0.0")
private HelloService demoService;

@HystrixCommand(fallbackMethod = "reliable")
public String doSayHello(String name) {
    return demoService.sayHello(name);
}
public String reliable(String name) {
    return "hystrix fallback value";
}
```