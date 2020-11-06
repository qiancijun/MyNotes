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