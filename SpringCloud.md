# 工程构建

SpringCloud技术栈

* 服务注册中心：~~Eureka~~，Zookeeper，Consul，Nacos
* 服务调用：Ribbon，LoadBalance，~~Feign~~，OpenFeign
* 服务降级：~~Hystrix~~，resilience4j，sentinel
* 服务网关：~~Zuul~~，==Zuul2==，gateway
* 服务配置：~~Config~~，Nacos
* 服务总线：~~Bus~~，Nacos

## 总工程构建

1. 新建Maven工程

    ![](SpringCloud\新建Maven工程.png)

2. 选定字符编码

    ![](SpringCloud\字符编码.png)

3. 注解激活生效

    ![](SpringCloud\注解激活生效.png)

4. 锁定Java编译版本

    ![](SpringCloud\锁定Java版本.png)





## POM文件

1. 指定打包方式为`pom`

    ```xml
    <packaging>pom</packaging>
    ```

2. 统一管理Jar包版本

    ``` xml
      <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>12</maven.compiler.source>
        <maven.compiler.target>12</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <lombok.version>1.18.10</lombok.version>
        <log4j.version>1.2.17</log4j.version>
        <mysql.version>8.0.18</mysql.version>
        <druid.version>1.1.16</druid.version>
        <mybatis.spring.boot.version>2.1.1</mybatis.spring.boot.version>
      </properties>
    ```

3. 管理子模块的依赖

    ``` xml
      <dependencyManagement>
        <dependencies>
        <dependency>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-project-info-reports-plugin</artifactId>
          <version>3.0.0</version>
        </dependency>
        <!--spring boot 2.2.2-->
        <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-dependencies</artifactId>
          <version>2.2.2.RELEASE</version>
          <type>pom</type>
          <scope>import</scope>
        </dependency>
        <!--spring cloud Hoxton.SR1-->
        <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-dependencies</artifactId>
          <version>Hoxton.SR1</version>
          <type>pom</type>
          <scope>import</scope>
        </dependency>
        <dependency>
          <groupId>com.alibaba.cloud</groupId>
          <artifactId>spring-cloud-alibaba-dependencies</artifactId>
          <version>2.1.0.RELEASE</version>
          <type>pom</type>
          <scope>import</scope>
        </dependency>
        <!--mysql-->
        <dependency>
          <groupId>mysql</groupId>
          <artifactId>mysql-connector-java</artifactId>
          <version>${mysql.version}</version>
          <scope>runtime</scope>
        </dependency>
        <!-- druid-->
        <dependency>
          <groupId>com.alibaba</groupId>
          <artifactId>druid</artifactId>
          <version>${druid.version}</version>
        </dependency>
          <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis.spring.boot.version}</version>
          </dependency>
          <!--junit-->
          <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
          </dependency>
          <!--log4j-->
          <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j.version}</version>
          </dependency>
        </dependencies>
      </dependencyManagement>
    ```

    只是声明定义，没有实际引入



## 子工程构建

1. 新建子工程

    ![](SpringCloud\建立子工程.png)

2. POM文件

    ``` xml
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-actuator</artifactId>
            </dependency>
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
            </dependency>
            <!-- https://mvnrepository.com/artifact/com.alibaba/druid-spring-boot-starter -->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid-spring-boot-starter</artifactId>
            </dependency>
    
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-jdbc</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-devtools</artifactId>
                <scope>runtime</scope>
                <optional>true</optional>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <optional>true</optional>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
        </dependencies>
    ```

3. YML文件

    ``` yml
    server:
      port: 8081
    
    spring:
      application:
        name: cloud-payment-service
      datasource:
        url: jdbc:mysql://localhost/table_name?rewriteBatchedStatements=true&serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=UTF-8
        driver-class-name: com.mysql.jdbc.Driver
        username: username
        password: password
        type: com.alibaba.druid.pool.DruidDataSource
        # 连接池配置
        druid:
          initial-size: 10
          min-idle: 5
          max-active: 20
          test-while-idle: true
          test-on-borrow: false
          test-on-return: false
          pool-prepared-statements: true
          max-pool-prepared-statement-per-connection-size: 20
          max-wait: 60000
          time-between-eviction-runs-millis: 60000
          min-evictable-idle-time-millis: 30000
          filters: stat
          async-init: true
          # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
          connection-properties: druid.stat.mergeSql=true;druid.stat.SlowSqlMills=5000
          # 监控后台的配置，如登录账号和密码等
          monitor:
            allow: 127.0.0.1
            loginUsername: admin
            loginPassword: admin
          driver-class-name: com.mysql.jdbc.Driver
    
    # Mybatis相关配置
    mybatis:
      configuration:
        map-underscore-to-camel-case: true
        log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
      mapper-locations: classpath:config/MyBatis/mapper/*.xml
      
    # 日志相关配置
    logging:
      level:
        root:
          info
    ```

4. 主启动类

    ``` java
    @SpringBootApplication
    public class MainApplication {
        public static void main(String[] args) {
            SpringApplication.run(MainApplication.class, args);
        }
    }
    ```



## 热部署

1. 添加devtools

    ``` xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    ```

2. 在总工程添加Maven-plugin插件

    ``` xml
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <fork>true</fork>
            <addResources>true</addResources>
        </configuration>
    </plugin>
    ```

3. 配置IDEA自动构建功能

    ![](SpringCloud\热部署.png)

4. `ctrl + shift + alt + /`

    ![](SpringCloud\热部署2.png)



### 业务逻辑

1. Mapper文件

    ``` xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="">
    
    </mapper>
    ```

2. 生产者代码

    实体类：

    ``` java
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class Payment implements Serializable {
        private Long id;
        private String serial;
    }
    ```

    ``` java
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class CommonResult<T> {
        private Integer code;
        private String message;
        private T data;
    
        public CommonResult(Integer code, String message) {
            this(code, message, null);
        }
    }
    ```

    Mapper

    ``` xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.qiancijun.cloud.dao.PaymentMapper">
    
    <!--    Payment getPaymentById(@Param("id") Long id);-->
        <select id="getPaymentById" parameterType="java.lang.Long" resultType="com.qiancijun.cloud.entities.Payment">
            select * from payment where id = #{id}
        </select>
    <!--    Integer create(Payment payment);-->
        <insert id="create" parameterType="com.qiancijun.cloud.entities.Payment" keyProperty="id">
            insert into payment (serial) values (#{serial})
        </insert>
    </mapper>
    
    ```

    Service

    ``` java
    @Service
    public class PaymentServiceImpl implements PaymentService {
    
        @Resource
        PaymentMapper paymentMapper;
    
        @Override
        public Integer create(Payment payment) {
            return paymentMapper.create(payment);
        }
    
        @Override
        public Payment getPaymentById(Long id) {
            return paymentMapper.getPaymentById(id);
        }
    }
    ```

    Controller

    ``` java
    @Slf4j
    @RestController
    public class PaymentController {
    
        @Autowired
        private PaymentService paymentService;
    
        @PostMapping("/payment/create")
        public CommonResult create(@RequestBody Payment payment) {
            Integer res = paymentService.create(payment);
            log.info("{}", res);
            if (res > 0) {
                return new CommonResult(200, "success", res);
            } else {
                return new CommonResult(200, "fail");
            }
        }
    
        @GetMapping("/payment/get/{id}")
        public CommonResult create(@PathVariable("id") Long id) {
            Payment payment = paymentService.getPaymentById(id);
            log.info("{}", payment);
            if (payment != null) {
                return new CommonResult(200, "success", payment);
            } else {
                return new CommonResult(200, "fail");
            }
        }
    
    }
    ```

3. 消费者代码

    实体类同生产者

    Controller

    ``` java
    @RestController
    @RequestMapping("/consumer")
    @Slf4j
    public class OrderController {
    
        private static final String PAYMENT_URL = "http://localhost:8081";
    
        @Autowired
        private RestTemplate restTemplate;
    
        @GetMapping("/payment/create")
        public CommonResult<Payment> create(Payment payment) {
            return restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
        }
    
        @GetMapping("/payment/get/{id}")
        public CommonResult<Payment> get(@PathVariable("id") Long id) {
            return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
        }
    }
    ```



## 工程重构

项目中，entities部分重复，我们不希望在一个项目下有许多相同重复的代码

![](SpringCloud\重构.png)

1. 新建模块`api-common`

2. POM文件

    引入`hutool`工具包（按需引入，可以不引入）

    ``` xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- https://mvnrepository.com/artifact/cn.hutool/hutool-all -->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.5.9</version>
        </dependency>
    
    </dependencies>
    ```

3. 将通用的实体类迁移到`api-common`模块下

4. 执行maven `clean`和`install`指令，上传到本地库，供其他模块调用

5. 其余模块删除原本的==entities==包

6. 引入自定义的api通用包

    ``` xml
    <dependency>
        <groupId>com.qiancijun</groupId>
        <artifactId>cloud-api-commons</artifactId>
        <version>${project.version}</version>
    </dependency>
    ```





# Eureka（AP）

Spring Cloud封装了Netflix公司开发的Euruka模块来实现==服务治理==

Eureka包含两个组件：

* Eureka Server：提供服务注册服务

    各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点信息可以再在界面中直观看到

* Eureka Client：通过注册中心进行访问

    是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮循负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳（默认周期为30s）。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90s）



## 服务治理

在传统的rpc远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务与服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。



## 服务注册与发现

Eureka采用了CS的设计架构，Eureka Server作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用Eureka的客户端连接到Eureka Serve并维持心跳连接。这样系统的维护人员就可以通过Eureka Serve来监控系统中各个微服务是否正常运行。

在服务注册与发现中，有一个注册中心。当服务器启动的时候，会把当前自己服务器的信息，比如：服务通讯地址以别名的方式注册到注册中心上。另一方（消费者|服务提供者），以该别名的方式去注册中心上获取到实际的服务通讯地址，然后再实现本地RPC调用RPC远程调用框架核心设计思想。

因为使用注册中心管理每个服务与服务之间的一个依赖关系（服务治理概念）。再任何RPC远程框架中，都会有一个注册中心（存放服务地址相关信息）



## 单点Eureka部署

1. 建立Moudle

2. POM文件

    ``` xml
    <dependencies>
    
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!-- boot web actuator -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!-- 一般的通用配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- 自己的common包 -->
        <dependency>
            <groupId>com.qiancijun</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
    ```

3. 编写YML文件

    ``` yml
    server:
      port: 7001
    
    eureka:
      instance:
        hostname: localhost # eureka服务端的实例名称
      client:
        register-with-eureka: false # false表示不向注册中心注册自己
        fetch-registry: false # false表示自己就是注册中心
        service-url:  # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
         defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
    ```

    

4. 主启动类

    ``` java
    @EnableEurekaServer // 表示为服务注册中心
    @SpringBootApplication
    public class EurekaMainApplication {
        public static void main(String[] args) {
            SpringApplication.run(EurekaMainApplication.class, args);
        }
    }
    ```

启动成功后，可以在`localhost:7001`访问Eureka监控页面

![](SpringCloud\Eureka.png)





## 服务注册

修改Provider端

1. POM文件

    ``` xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    ```

2. 需要在yml中指明模块名称，并且配置client

    ``` yml
    spring:
      application:
        name: cloud-payment-service
    
    eureka:
      client:
        register-with-eureka: true # 是否将自己注册进 EurekaServer 默认为 true
        fetch-registry: true # 是否从 EurekaServer 抓取已有的信息，默认为 true 。集群必须设置为true才能配合ribbon使用负载均衡
        service-url:
          defaultZone: http://localhost:7001/eureka
    ```

3. 启动后可以在监控页面看见模块名称

    ![](SpringCloud\服务注册.png)

    >  爆红是由于Eureka有自我保护机制



## 集群

### 原理说明

服务注册：将服务信息注册进注册中心

服务发现：从注册中心上获取服务信息， 实质上k-v键值对，key存服务名，value为服务地址

执行步骤：

1. 先启动Eureka注册中心
2. 启动服务提供者
3. 服务启动后会把自身信息（比如服务地址以别名方式注册进Eureka）
4. 消费者服务在需要调用接口时，使用服务别名去注册中心获取实际的RPC远程调用地址
5. 消费者获得调用地址后，底层是利用HttpClient技术实现远程调用
6. 消费者获得服务地址后会缓存在本地JVM内存中，默认间隔30秒更新一次服务调用地址

注册中心与注册中心之间相互注册，对外只暴露一个整体





### 配置步骤

1. 添加hosts映射

    ```
    127.0.0.1 eureka7001.com
    127.0.0.1 eureka7002.com1
    ```

2. 建Module

3. 写POM文件

4. 改YML

     ``` yml
    server:
      port: 7001
    
    eureka:
      instance:
        hostname: eureka7001.com # eureka服务端的实例名称
      client:
        register-with-eureka: false # false表示不向注册中心注册自己
        fetch-registry: false # false表示自己就是注册中心
        service-url:  # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
          defaultZone: http://eureka7002.com:7002/eureka/ # 注册其余EurekaServer，多个用逗号分割
    ```



### 服务注册

修改服务的YML，将服务注册入Eureka集群中

``` yml
eureka:
  client:
    register-with-eureka: true # 是否将自己注册进 EurekaServer 默认为 true
    fetch-registry: true # 是否从 EurekaServer 抓取已有的信息，默认为 true 。集群必须设置为true才能配合ribbon使用负载均衡
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```





## 服务集群配置

1. 构建多个微服务

2. 修改消费者`restTemplate`发送地址

    ``` java
    @RestController
    @RequestMapping("/consumer")
    @Slf4j
    public class OrderController {
    
        // private static final String PAYMENT_URL = "http://localhost:8081";
        private static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";
    
        @Autowired
        private RestTemplate restTemplate;
    
        @GetMapping("/payment/create")
        public CommonResult<Payment> create(Payment payment) {
            return restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
        }
    
        @GetMapping("/payment/get/{id}")
        public CommonResult<Payment> get(@PathVariable("id") Long id) {
            return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
        }
    }
    ```

3. 修改`restTemplate`支持负载均衡

    ``` java
    @Configuration
    public class ApplicationContextConfig {
    
        @Bean
        @LoadBalanced
        public RestTemplate getRestTemplate() {
            return new RestTemplate();
        }
    }
    ```

    > 这就是Ribbon的负载均衡机制，后续整合不需要手动配置

![](SpringCloud\服务集群.png)



### 微服务信息完善

修改YML文件

``` yml
eureka:
  client:
    register-with-eureka: true # 是否将自己注册进 EurekaServer 默认为 true
    fetch-registry: true # 是否从 EurekaServer 抓取已有的信息，默认为 true 。集群必须设置为true才能配合ribbon使用负载均衡
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    instance-id: payment8082
    prefer-ip-address: true # 访问路径可以显示IP
```



![](SpringCloud\服务信息完善.png)





## 服务发现

对于注册进Eureka的微服务，可以通过服务发现来获取该服务的信息

1. 自动注入组件，并且在主启动上添加`@EnableDiscoveryClient`

    ``` java
    @Autowired
    private DiscoveryClient discoveryClient;
    ```

2. 服务发现代码

    ``` java
    @GetMapping("/payment/discory")
    public Object discory() {
        List<String> services = discoveryClient.getServices();
        for (String service : services) {
            log.info(service);
        }
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance instance : instances) {
            log.info(instance.getInstanceId() + '\t' + instance.getPort() + '\t' + instance.getHost() + '\t' + instance.getUri());
        }
        return discoveryClient;
    }
    ```

    

## 自我保护

保护模式主要用于一组客户端和Eureka Server之间存在网络分区场景下的保护。一旦进入保护模式，EurekaServer将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，也就是不会注销任何微服务

为了防止EurekaClient可以正常运行但是与EurekaServer网络不通情况下，EurekaServer不会立刻将EurekaClient服务剔除。

默认情况下，如果EurekaServer在一定时间内没有接收到某个微服务的实例的心跳，EurekaServer将会注销该实例（默认90s）。但是当网络分区故障发生，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了，因为微服务本身其实是健康的，此时不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题



* 关闭自我保护：

    ```
    enable-self-preservation = false
    eviction-interval-timer-in-ms = 2000
    ```

    服务端

    ```
    lease-renewal-interval-in-seconds = 1 # Eureka客户端向服务端发送心跳的间隔，单位为m秒
    lease-expiration-duration-in-seconds = 2 # Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒，超时将剔除服务
    ```

    





# Zookeeper（CP）

关于[Zookeeper](https://xlich.top/2020/12/15/ZooKeeper/)详细的笔记

## 服务注册

1. 建工程

2. POM文件

    ``` xml
    <!-- SpringCloud整合zookeeper客户端 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
    </dependency>
    ```

3. YML文件，指明Zookeeper服务地址

    ``` yml
    spring:
      cloud:
        zookeeper:
          connect-string: 192.168.74.100:2181
    ```

4. 在主启动上添加`@EnableDiscoveryClient`注解



Zookeeper所创建的节点是临时节点



服务消费注册

1. POM文件同消费者

2. YML

    ``` yml
    server:
      port: 80
    
    spring:
      application:
        name: cloud-order-service
      cloud:
        zookeeper:
          connect-string: 192.168.74.100:2181
    ```

3. Controller

    ``` java
    @RestController
    @RequestMapping("/consumer")
    @Slf4j
    public class OrderController {
        private static final String PAYMENT_URL = "http://cloud-payment-service";
    
        @Autowired
        private RestTemplate restTemplate;
    
        @GetMapping("/payment/zk")
        public String zk(Payment payment) {
            return restTemplate.getForObject(PAYMENT_URL + "/payment/zk", String.class);
        }
    }
    ```



# Consul（CP）

Consul是一套开源的分布式服务发现和配置管理系统，由HashiCorp公司用Go开发

提供了微服务系统中的服务治理、配置中心、控制总线等功能。这些功能中的每一个都可以根据需要单独使用，也可以一起使用以构建全方位的服务网络，总之Consul提供了一种完整的服务网络解决方案



## 启动

* 使用开发模式启动

    ```
    consul agent -dev
    ```

    通过`localhost:8500`访问Consul首页



## 服务注册

1. POM文件

    ``` xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    </dependency>
    ```

2. YML文件

    ``` yml
    server:
      port: 8085
    
    spring:
      application:
        name: consul-provider-payment
      cloud:
        consul:
          host: localhost
          port: 8500
          discovery:
            service-name: ${spring.application.name}
    ```

3. 主启动类添加`@EnableDiscoveryClient`

4. 运行后
    ![](SpringCloud\Consul.png)





# Ribbon

Spring Cloud Ribbon 是基于Netflix Ribbon实现的一套==客户端负载均衡==的工具

Ribbon是Netflix发布的开源项目，主要功能是提供==客户端的软件负载均衡算法和服务调用==。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单得说，就是在配置文件中列出LoadBalancer后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮循，随机连接等）去连接这些机器。我们很容易使用RIbbon实现自定义的负载均衡算法。



## LoadBalance负载均衡

简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA（高可用）。

Nginx是服务器负载均衡，客户端所有请求都会交给Nginx，然后又Nginx实现转发请求。即负载均衡是由服务端实现的。

Ribbon本地负载均衡，在调用微服务接口时候，会在注册中心上获取信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术。



### 集中式LB

即在服务的消费方和提供方之间使用独立的LB设备（可以是硬件，也可以是软件），由该设施负责把访问请求通过某种策略转发至服务的提供方



### 进程内LB

将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。



## 负载规则替换

Ribbon自定义配置类不能放在`@ComponentScan`所扫描的当前包下以及子包下，否则自定义配置类就会被所有的Ribbon客户端所共享，达不到特殊化定制的目的。

1. 新建包

2. 在主启动上添加`@RibbonClient`注解，指明服务名和自定义规则类

    ``` java
    @SpringBootApplication
    @EnableDiscoveryClient
    @RibbonClient(value = "CLOUD-PAYMENT-SERVICE", configuration = MyRibbonRule.class)
    public class OrderMainApplication {
        public static void main(String[] args) {
            SpringApplication.run(OrderMainApplication.class, args);
        }
    }
    
    // 服务名要和注册的相同
    ```

    

3. 自定义规则类

    ``` java
    @Configuration
    public class MyRibbonRule {
        @Bean
        public IRule myRule() {
            return new RandomRule();
        }
    }
    ```

    

## 负载均衡算法

负载均衡算法：rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标

每次服务重启动后rest接口计数从1开始



自定义负载均衡算法：

1. 去掉`@LoadBalance`注解

2. 创建自定义的`IRule`接口

    ``` java
    public interface LoadBalancer {
        ServiceInstance instances(List<ServiceInstance> serviceInstances); // 用于获取现存的服务实例
    }
    ```

3. 实现自定义负载均衡算法

    ``` java
    @Component
    public class MyLoadBalancer implements LoadBalancer {
    
        private AtomicInteger atomicInteger = new AtomicInteger(0);
    
        public final int getAndIncrement() {
            int current;
            int next;
            do {
                current = this.atomicInteger.get();
                next = current >= Integer.MAX_VALUE ? 0 : current + 1;
            } while (!this.atomicInteger.compareAndSet(current, next));
            System.out.println("next:----" + next);
            return next;
        }
    
        @Override
        public ServiceInstance instances(List<ServiceInstance> serviceInstances) {
            int index = getAndIncrement() % serviceInstances.size();
            return serviceInstances.get(index);
        }
    }
    ```

4. Controller

    ``` java
    @GetMapping("/payment/lb")
    public String mylb() {
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        if (instances == null || instances.size() <= 0) {
            return null;
        }
        ServiceInstance serviceInstance = loadBalancer.instances(instances);
        URI uri = serviceInstance.getUri();
    
        return restTemplate.getForObject(uri + "/payment/lb", String.class);
    }
    ```



### 源码

Ribbon自带的负载均衡算法，全来自于一个接口

``` java
public interface IRule {
    Server choose(Object var1);

    void setLoadBalancer(ILoadBalancer var1);

    ILoadBalancer getLoadBalancer();
}
```

主要看默认的轮循算法`RoundRobinRule`类

* 服务选择

    ``` java
    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        } else {
            Server server = null;
            int count = 0;
    
            while(true) {
                if (server == null && count++ < 10) {
                    // 获取所有可达服务
                    List<Server> reachableServers = lb.getReachableServers();
                    List<Server> allServers = lb.getAllServers();
                    int upCount = reachableServers.size();
                    int serverCount = allServers.size();
                    if (upCount != 0 && serverCount != 0) {
                        // 获取下一个应该轮循的服务下标
                        int nextServerIndex = this.incrementAndGetModulo(serverCount);
                        // 获取下一个服务地址
                        server = (Server)allServers.get(nextServerIndex);
                        if (server == null) {
                            Thread.yield();
                        } else {
                            if (server.isAlive() && server.isReadyToServe()) {
                                return server;
                            }
    
                            server = null;
                        }
                        continue;
                    }
    
                    log.warn("No up servers available from load balancer: " + lb);
                    return null;
                }
    
                if (count >= 10) {
                    log.warn("No available alive servers after 10 tries from load balancer: " + lb);
                }
    
                return server;
            }
        }
    }
    ```

* 自增算法

    ``` java
    private int incrementAndGetModulo(int modulo) {
        int current;
        int next;
        // CAS自旋锁，保证高并发下的自增不会出现线程安全问题
        do {
            current = this.nextServerCyclicCounter.get();
            next = (current + 1) % modulo;
        } while(!this.nextServerCyclicCounter.compareAndSet(current, next));
    
        return next;
    }
    ```





# OpenFeign

Feign是一个声明式WebService客户端。使用Feign能让编写Web Service客户端更加简单。

它的使用方法是，定义一个服务接口然后在上面添加注解。Feign也支持可插拔式的编码器和解码器。Spring Cloud对Feign进行了封装，使其支持了Spring MVC标准注解和HttpMessageConverters。Feign可以于Eureka和Ribbon组合使用以支持负载均衡。

在使用Ribbon+RestTemplate时，利用RestTemplate对Http请求的封装处理，形成了一套模板化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，==往往一个接口会被多出调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用==。所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，==我们只需创建一个接口并使用注解的方式来配置它==。即可完成对服务提供方的接口绑定，简化了使用Spring Cloud Ribbon时，自动封装服务调用客户端的开发量。



## 服务调用

1. 建Module

2. POM

    ``` xml
    <dependencies>
    
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
    
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.qiancijun</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
    ```

3. YML

    ``` yml
    server:
      port: 80
    
    eureka:
      client:
        register-with-eureka: true # 是否将自己注册进 EurekaServer 默认为 true
        fetch-registry: true # 是否从 EurekaServer 抓取已有的信息，默认为 true 。集群必须设置为true才能配合ribbon使用负载均衡
        service-url:
          defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
    ```

4. 主启动类添加`@EnableFeignClients`注解

5. 新建接口

    ``` java
    @Component
    @FeignClient(value = "CLOUD-PAYMENT-SERVICE")
    public interface PaymentFeignService {
        @GetMapping("/payment/get/{id}")
        CommonResult get(@PathVariable("id") Long id);
    }
    ```

6. Controller

    ``` java
    @RestController
    @RequestMapping("/consumer")
    @Slf4j
    public class PaymentController {
    
        @Autowired
        private PaymentFeignService paymentFeignService;
    
        @GetMapping("/payment/get/{id}")
        public CommonResult<Payment> get(@PathVariable("id") Long id) {
            return paymentFeignService.get(id);
        }
    }
    ```

    

## 超时控制

OpenFeign默认只等待一秒钟

``` yml
# 设置Feign客户端超时时间
ribbon:
  ReadTimeout: 5000 # 建立连接后读取可用资源的时间
  ConnectTimeout: 5000 # 建立连接所用的时间
```



## 日志增强

Feign提供了日志打印功能，可以通过配置来调整日志级别，从而了解Feign中HTTP请求的细节

日志级别：

* NONE：默认的，不显示任何日志；
* BASIC：仅记录请求方法、URL、响应状态码及执行时间；
* HEADERS：除了BASIC中定义的信息之外，还有请求和响应的头信息；
* FULL：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据





# Hystrix

分布式系统面临的问题：

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免的失败。

* 服务雪崩

    多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的==扇出==。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”

    对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。



Hystrix是一个用于处理分布式系统的==延迟==和==容错==的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，==不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性==

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控，==向调用方返回一个符合预期的、可处理的备选响应，而不是长时间的等待或者抛出调用方无法处理的异常==，这样就保证了服务调用方的线程不会被长时间、不必要的占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。



* 服务降级：不让客户端等待，返回一个备选响应
* 服务熔断：达到最大服务访问后，拒绝访问
* 服务限流



## 构建

1. 建Module

2. POM

    ``` xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
    ```

    

3. YML

    ``` yml
    server:
      port: 8086
    
    
    eureka:
      client:
        register-with-eureka: true # 是否将自己注册进 EurekaServer 默认为 true
        fetch-registry: true # 是否从 EurekaServer 抓取已有的信息，默认为 true 。集群必须设置为true才能配合ribbon使用负载均衡
        service-url:
          defaultZone: http://eureka7001.com:7001/eureka
    spring:
      application:
        name: cloud-provider-hystrix-payment
    ```

4. 主启动

    ``` java
    @SpringBootApplication
    @EnableEurekaClient
    public class PaymentHystrixMainApplication {
        public static void main(String[] args) {
            SpringApplication.run(PaymentHystrixMainApplication.class, args);
        }
    }
    ```

5. 业务类

    Service

    ``` java
    @Service
    public class PaymentService {
        public String PaymentInfo(Long id) {
            return "线程池：" + Thread.currentThread().getName() + " paymentInfo" + "\t" + id;
        }
    
        public String PaymentInfoTimeOut(Long id) {
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "线程池：" + Thread.currentThread().getName() + " PaymentInfoTimeOut" + "\t" + id;
        }
    }
    ```

    Controller

    ``` java
    @RestController
    @Slf4j
    public class PaymentController {
    
        @Autowired
        private PaymentService paymentService;
    
        @Value("${server.port}")
        private String port;
    
        @GetMapping("/payment/hystrix/ok/{id}")
        public String ok(@PathVariable("id") Long id) {
            return paymentService.PaymentInfo(id);
        }
        @GetMapping("/payment/hystrix/timeout/{id}")
        public String timeOut(@PathVariable("id") Long id) {
            return paymentService.PaymentInfoTimeOut(id);
        }
    }
    ```



## 服务降级

1. 提供者自身延长服务有效时间

    ``` java
    @HystrixCommand(fallbackMethod = "fallBackMethod", commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
    })
    public String PaymentInfoTimeOut(Long id) {
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程池：" + Thread.currentThread().getName() + " PaymentInfoTimeOut" + "\t" + id;
    }
    // 服务有效时间为3秒，线程睡眠5秒，模拟服务超时
    ```

2. 出错方法

    ``` java
    public String fallBackMethod(Long id) {
        return "线程池：" + Thread.currentThread().getName() + " PaymentInfoTimeOut服务降级";
    }
    // 注意形参
    ```

3. 主启动添加`@EnableCircuitBreaker`注解

消费者端实现服务降级

1. 主启动添加`@EnableHystrix`注解来支持Hystrix

2. Controller添加自己的服务降级

    ``` java
    @GetMapping("/payment/hystrix/timeout/{id}")
    @HystrixCommand(fallbackMethod = "fallBackMethod", commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "5000")
    })
    public String timeOut(@PathVariable("id") Long id) {
        return paymentService.timeOut(id);
    }
    
    public String fallBackMethod(Long id) {
        return "消费者80";
    }
    ```

    由于消费者用Feign去注册中心调用方法，Feign默认一秒超时，所以需要在配置文件中修改Ribbon的超时时间，否则`@HystrixProperty`设置服务有效时间是无效的



## 全局服务降级

``` java
@Service
@DefaultProperties(defaultFallback = "globalFallbackMethod")
public class PaymentService {
    public String PaymentInfo(Long id) {
        return "线程池：" + Thread.currentThread().getName() + " paymentInfo" + "\t" + id;
    }

//    @HystrixCommand(fallbackMethod = "fallBackMethod", commandProperties = {
//            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "5000")
//    })
    @HystrixCommand
    public String PaymentInfoTimeOut(Long id) {
        try {
            int i = 10 / 0;
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程池：" + Thread.currentThread().getName() + " PaymentInfoTimeOut" + "\t" + id;
    }

    public String fallBackMethod(Long id) {
        return "线程池：" + Thread.currentThread().getName() + " PaymentInfoTimeOut服务降级";
    }


    public String globalFallbackMethod() {
        return "全局服务降级";
    }

}
```

全局服务降级方法不能带有形参



* 在客户端配置服务降级来通知用户远程服务不可用

    yml开启对Hystrix的支持

    ``` yml
    feign:
      hystrix:
        enabled: true
    ```

    Service

    ``` java
    @Component
    @FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT", fallback = PaymentFallbackService.class)
    public interface PaymentService {
    
        @GetMapping("/payment/hystrix/ok/{id}")
        public String ok(@PathVariable("id") Long id);
    
        @GetMapping("/payment/hystrix/timeout/{id}")
        public String timeOut(@PathVariable("id") Long id);
    }
    ```

    实现类

    ``` java
    @Component
    public class PaymentFallbackService implements PaymentService {
        @Override
        public String ok(Long id) {
            return "远程服务挂了";
        }
    
        @Override
        public String timeOut(Long id) {
            return "远程服务挂了";
        }
    }
    ```





## 服务熔断

* 熔断机制概述

    熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的吗某个微服务出错不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。

    当检测到该节点微服务调用响应正常后，恢复调用链路。

    在Spring Cloud框架里，熔断机制通过Hystrix实现。Hystrix会监控微服务调用的状况，当失败的调用到一定阈值，缺省是5秒内20次调用失败，就会启动熔断机制。熔断机制的注解是`@HystrixCommand`

``` java
// 服务熔断
@HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback", commandProperties = {
    @HystrixProperty(name = "circuitBreaker.enabled", value = "true"), // 是否开启断路器
    @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"), // 请求次数
    @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"), // 时间窗口期，尝试恢复服务
    @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60"), // 失败率多少次后跳闸
})
public String paymentCircuitBreaker(@PathVariable("id") Long id) {
    if (id < 0) {
        throw new RuntimeException("id不能负数");
    }
    String serialNumber = UUID.randomUUID().toString();
    return Thread.currentThread().getName() + '\t' + serialNumber;
}

public String paymentCircuitBreaker_fallback(@PathVariable("id") Long id) {
    return "id不能负数，稍后再试";
}
```

* 熔断类型
    * 熔断打开：请求不再进行调用当前服务，内部设置时钟一般为MTTR（平均故障处理时间），当打开时长达到所设时钟则进入半熔断状态
    * 熔断关闭：熔断关闭不会对服务进行熔断
    * 熔断半开：部分请求根据规则调用当前服务，如果请求成功且符合规则则认为当前服务恢复正常，关闭熔断

断路器打开之后：

1. 再有请求调用的时候，将不会调用主逻辑，而是直接调用降级fallback。通过断路器，实现了自动的发现错误并将降级逻辑切换为主逻辑，减少响应延迟的效果。
2. 对主逻辑进行熔断之后，hystrix会启动一个休眠时间窗，在这个时间窗内，降级逻辑是临时的成为主逻辑，当休眠时间窗到期，断路器将进入半开状态，释放一次请求到原来的主逻辑上，如果此次请求正常返回，那么断路器将继续闭合，主逻辑恢复，如果这次请求依然有问题，断路器继续进入打开状态，休眠时间窗重新计时。





## 服务监控

除了隔离依赖服务的调用以外，Hystrix还提供了==准实时的调用监控==，Hystrix会持续的记录所有通过Hystrix发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少请求多少成功，多少失败等。Netflix通过hystrix-metrics-event-stream项目实现了对以上指标的监控。Spring Cloud也提供了Hystrix Dashboard的整合，对监控内容转化成可视化界面。





# GateWay
[Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)

Gateway是在Spring生态系统之上构建的API网关服务，基于Spring 5，Spring Boot 2 和 Project Reactor 等技术。Gateway旨在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能，例如：熔断、限流、重试等



Spring Cloud Gateway具有如下特性:

1. 基于Spring Framework 5，Project Reactor 和Spring Boot 2.0 进行构建
2. 动态路由：能够匹配任何请求属性
3. 可以对路由指定Predicate（断言）和Filter（过滤器）
4. 集成Hystrix的断路器功能
5. 集成Spring Cloud 服务发现功能
6. 易于编写的Predicate（断言）和Filter（过滤器）
7. 请求限流功能
8. 支持路径重写；

SpringCloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是代替Zuul，在Spring Cloud 2.0 以上版本中，没有对新版本的 Zuul 2.0 以上最新高性能版本进行集成，仍然还是使用的 Zuul 1.x 非 Reactor 模式的老版本。而为了提升网关的性能，SpringCloud Gateway 是基于 WebFlux 框架实现的，而 WebFlux 框架底层则使用了高性能的 Reactor 模式通信架构 Netty

## 与Zuul区别
1. Zuul 1.x 是一个基于阻塞 I/O 的 API Gateway
2. Zuul 1.x 基于 Servlet 2.5 使用阻塞架构，它不支持任何长连接，Zuul 的设计模式和 Nginx 较像，每次 I/O 操作都是从工作线程中选择一个执行，请求线程被阻塞到工作线程完成，但是差别是 Nginx 用 C++ 实现，Zuul 用 Java 实现，而 JVM 本身会有第一次加载较慢的情况，使得 Zuul 的性能相对较差
3. Zuul 2.x 理念更先进，想基于 Netty 非阻塞和支持长连接，但 SpringCloud 目前还没有整合。Zuul 2.x 的性能较 Zuul 1.x 有较大提升。在性能方面，根据官方提供的基准测试，Spring Cloud Gateway 的 RPS（每秒请求数）是 Zuul 的1.6倍
4. Spring Cloud Gateway 建立在 Spring Framework 5、Project Reactor 和 Spring Boot 2 之上，使用非阻塞 API
5. Spring Cloud Gateway 还支持 WebSocket，并且与 Spring 紧密集成拥有更好的开发体验


## 工作流程

* Router（路由）
    路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为 true 则匹配该路由

* Predicate（断言）
    参考的是 Java8 的 java.util.function.Predicate，开发人员可以匹配 HTTP 请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由

* Filter（过滤）
    Spring 框架中 GatewayFilter 的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。

web 请求通过一些匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化控制。predicate 就是匹配条件，而 filter 就可以理解为一个无所不能的拦截器。有了这两个元素，再加上目标 uri，就可以实现一个具体的路由了。

核心逻辑：路由转发 + 执行过滤链

## 构建

1. POM
    ``` xml
   <dependencies>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>com.qiancijun</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
   ```

2. YML，使用YML进行路由配置

    ``` yml
    server:
      port: 9527
    spring:
      application:
        name: cloud-gateway
      cloud:
        gateway:
          routes:
            - id: payment_routh # payment_router 路由的Id，没有固定规则但要求唯一，建议配合服务名
              uri: http://localhost:8081 # 匹配后提供服务的路由地址
              predicates:
                - Path=/payment/get/** # 断言，路径相匹配的进行路由
            - id: payment_routh2
              uri: http://localhost:8081
              predicates:
                - Path=/payment/lb/** # 断言，路径相匹配的进行路由
    
    eureka:
      instance:
        hostname: cloud-gateway-service
      client:
        service-url:
          register-with-eureka: true
          fetch-register: true
          defaultZone: http://eureka7001.com:7001/eureka
    ```

3. 主启动类
    ``` java
    @SpringBootApplication
    @EnableEurekaClient
    public class Gateway9527MainApplication {
        public static void main(String[] args) {
            SpringApplication.run(Gateway9527MainApplication.class, args);
        }
    }
    ```



使用编码进行路由配置

``` java
@Configuration
public class GatewayConfig {
    @Bean
    public RouteLocator myRouteLocator(RouteLocatorBuilder builder) {
        RouteLocatorBuilder.Builder routes = builder.routes();
        routes.route("path_route_page", r -> r.path("/MainPage").uri("http://www.cheryl-chun.cn/#/MainPage")).build();
        return routes.build();
    }
}
```

## 动态路由
默认情况下 Gateway 会根据注册中心注册的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能。

改写YML文件
``` yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh # payment_router 路由的Id，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service # 匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/** # 断言，路径相匹配的进行路由
        - id: payment_routh2
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/lb/** # 断言，路径相匹配的进行路由
```

## Predicate
Spring Cloud Gateway 将路由匹配作为 Spring WebFlux HandlerMapping 基础架构的一部分。
Spring Cloud Gateway 包括许多内置的 Route Predicate 工厂。所有这些 Predicate 都与 HTTP 请求的不同属性匹配。多个 Route Predicate 工厂可以结合
Spring Cloud Gateway 创建 Route 对象时，使用 RoutePredicateFactory 创建 Predicate 对象，Predicate 对象可以赋值给 Route。Spring Cloud Gateway 包含许多内置的 Route Predicate Factories。
所有这些谓词都匹配 HTTP 请求的不同属性。多种谓词工厂可以组合，并通过逻辑 and。

* 基于时间的断言
    ``` yml
    spring:
        cloud:
            gateway:
            routes:
            - id: after_route
                uri: https://example.org
                predicates:
                - After=2017-01-20T17:42:47.789-07:00[America/Denver]
    ```
    同样的还有
    ```
    - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
    - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
    ```

* 基于缓存的断言
    Cookie Route Predicate 需要两个参数，一个是 Cookie name， 一个是正则表达式
    路由规则会通过获取对应的 Cookie name 值和正则表达式去匹配，如果匹配上就会执行路由，如果没有匹配上则不执行
    ``` yml
    spring:
        cloud:
            gateway:
            routes:
            - id: cookie_route
                uri: https://example.org
                predicates:
                - Cookie=chocolate, ch.p
    ```

* 基于请求头的断言

    ``` yml
    spring:
        cloud:
            gateway:
            routes:
            - id: header_route
                uri: https://example.org
                predicates:
                - Header=X-Request-Id, \d+ # 请求头含有X-Request-Id请求头，并且值是整数
    ```


## Filter
路由过滤器可用于修改进入的 HTTP 请求和返回的 HTTP 响应，路由过滤器只能指定路由进行使用。
Spring Cloud Gateway 内置了多种路由过滤器，他们都由 GatewayFilter 的工厂类来产生

* 自定义Filter
    ``` java
    @Slf4j
    @Component
    public class MyFilter implements GlobalFilter, Ordered {
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            log.info("自定义过滤器");
            String token = exchange.getRequest().getQueryParams().getFirst("token");
            if (token == null) {
                log.info("没有权限");
                exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
                return exchange.getResponse().setComplete();
            }
            return chain.filter(exchange);
        }

        @Override
        public int getOrder() {
            return 0;
        }
    }
    ```
> 其余更详细的过滤规则可以查看官网或 API 文档


# Config

[Spring Cloud Config](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/)

微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。

Spring Cloud Config 为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为==各个不同微服务应用==的所有环境提供了一个==中心化的外部配置==

Spring Cloud Config 分为服务端和客户端两部分。服务端也称为==分布式配置中心==，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口。

客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息，配置服务器默认采用 git 来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过 git 客户端工具来方便的管理和访问配置内容。

## 特点
1. 集中化管理配置文件
2. 不同环境不同配置，动态化的配置更新，分环境部署，比如 dev/prod/beta/release
3. 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
4. 当配置发生变动时，服务不需要重启即可感知到配置的变化并应用到新的配置
5. 将配置信息以 REST 接口的形式暴露

## 配置中心搭建
1. 在Github上新建一个Repository
2. 获取新建的git地址
3. 在本地硬盘目录上新建git仓库并clone
4. 新建配置中心模块
5. POM
    ``` xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    ```
6. YML
    ``` yml
    server:
        port: 3344

    spring:
        application:
            name: cloud-config-center # 注册进eureka服务器的服务名
    cloud:
        config:
        server:
        git:
            uri: https://github.com/qiancijun/SprintCloudConfig.git # git仓库
            search-paths:
            - SprintCloudConfig # 搜索目录
            username: 769303522@qq.com
            password: 20010206qianci
        label: master # 读取分支

    eureka:
        client:
            service-url:
            defaultZone: http://eureka7001.com:7001/eureka
    ```
    如果使用SSH连接，需要配置密钥

通过`http://localhost:3344/master/config-dev.vml`访问配置文件

* 配置文件命名格式
    ```
    /{application}/{profile}[/{label}]
    /{application}-{profile}.yml
    /{label}/{application}-{profile}.yml
    /{application}-{profile}.properties
    /{label}/{application}-{profile}.properties
    ```

## 客户端

1. POM
    ``` xml
     <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    ```
2. bootstrap.yml
    application.yml 是用户级的资源配置项，bootstrap.yml 是系统级的，优先级更高
    Spring Cloud会创建一个 Bootstrap Context，作为 Spring 应用的 Application Context 的父上下文。初始化的时候，Bootstrap Context 负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的 Environment
    Bootstrap 属性有高优先级，默认情况下，它们不会被本地配置覆盖。Bootstrap Context 和 Application Context 有者不同的约定，所以新增了一个 bootstrap.yml 文件，保证 Bootstrap Context 和Application Context 配置的分离
    ``` yml
    server:
        port: 3355

    spring:
        application:
            name: config-client
    cloud:
        config:
            label: master # 分支名称
            name: config # 配置文件名称
            profile: dev # 读取后缀名称
            uri: http://localhost:3344 # 配置中心地址

    eureka:
        client:
            service-url:
                defaultZone: http://eureka7001.com:7001/eureka
    ```

3. 业务类
    ``` java
    @RestController
    public class ConfigClientController {
        @Value("${config.info}")
        private String info;

        @GetMapping("/configInfo")
        public String getConfigInfo() {
            return info;
        }
    }
    ```

在Github上修改完配置文件，配置中心可以立马获取新的配置，但是客户端必须重启微服务才能获取，这是不现实的

## 动态刷新
避免每次更新配置都要重启客户端微服务

1. POM文件引入`actuator`监控模块
2. 修改YML，暴露监控端口
    ``` yml
    management:
        endpoints:
            web:
                exposure:
                    include: "*"
    ```
3. 在业务类上新增`@RefreshScope`

到这一步，客户端还是不会动态刷新。需要发送POST请求刷新客户端
`curl -X POST "http://localhost:3355/actuator/refresh"`


# Bus
使用消息总线 Bus 可以实现分布式自动刷新配置功能

Bus 支持两种消息代理：RabbitMQ 和 Kafka

Spring Cloud Bus 是用来将分布式系统的节点与轻量级消息系统链接起来的框架，它整合了 Java 的事件处理机制和消息中间件的功能
![](SpringCloud\消息总线Bus.png)

Spring Cloud Bus 能管理和传播分布式系统的消息，就像一个分布式执行器，可用于广播状态更改、事件推送等，也可以当作微服务间的通信通道

* 总线:
    在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个共用的消息主题，并让系统中所有微服务实例都连接上来，由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。在总线上的各个实例，都可以方便的广播一些需要让其他连接在该主题上的实例都知道的消息。

* 基本原理：
    ConfigClient 实例都监听 MQ 中同一个 Topic（默认是 Spring Cloud Bus）。当一个服务刷新数据的时候，它会把这个信息放入到 topic 中，这样其他监听同一个 topic 的服务就能得到通知，然后去更新自身的配置

由于 RabbitMQ 需要 Erlang 环境的支持，这里不做演示
![](SpringCloud\消息总线2.png)

Kafka使用Scala实现，被用作LinkedIn的活动流和运营数据处理的管道，现在也被诸多互联网企业广泛地用作为数据流管道和消息系统
![](SpringCloud\kafka.png)
Kafka是基于消息发布/订阅模式实现的消息系统，其主要设计目标如下：

1. 消息持久化：以时间复杂度为O(1)的方式提供消息持久化能力，即使对TB级以上数据也能保证常数时间复杂度的访问性能。
2. 高吞吐：在廉价的商用机器上也能支持单机每秒100K条以上的吞吐量
3. 分布式：支持消息分区以及分布式消费，并保证分区内的消息顺序
4. 跨平台：支持不同技术平台的客户端（如：Java、PHP、Python等）
5. 实时性：支持实时数据处理和离线数据处理
6. 伸缩性：支持水平扩展

Kafka中涉及的一些基本概念：
1. Broker：Kafka集群包含一个或多个服务器，这些服务器被称为Broker。
2. opic：逻辑上同Rabbit的Queue队列相似，每条发布到Kafka集群的消息都必须有一个Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个Broker上，但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）
3. Partition：Partition是物理概念上的分区，为了提供系统吞吐率，在物理上每个Topic会分成一个或多个Partition，每个Partition对应一个文件夹（存储对应分区的消息内容和索引文件）。
4. Producer：消息生产者，负责生产消息并发送到Kafka Broker。
5. Consumer：消息消费者，向Kafka Broker读取消息并处理的客户端。
6. Consumer Group：每个Consumer属于一个特定的组（可为每个Consumer指定属于一个组，若不指定则属于默认组），组可以用来实现一条消息被组内多个成员消费等功能。

* client 和 Server 端添加
    ``` xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-kafka</artifactId>
    </dependency>
    ```
* kafka 需要 Zookeeper 的支持，先启动 Zookeeper
* 再启动 kafka，需要指定配置文件

在 Config Server 配置中心端添加
``` yml
management:
  endpoints: # 暴露Bus刷新配置的端点
    web:
      exposure:
        include: "bus-refresh"
```

发送 POST 请求来刷新所有的配置
```
curl -X POST http://localhost:配置中心端口号/actuator/bus-refresh
```

指定具体某一个实例生效而不是全部
```
http://localhost:配置中心端口号/actuator/bus-refresh/{destination}
```

# Stream
[Spring Cloud Stream](https://docs.spring.io/spring-cloud-stream/docs/3.1.1/reference/html/)
屏蔽底层消息中间件的差异，降低切换成本，统一消息的编程模型。

Spring Cloud Stream 是一个构建消息驱动微服务的框架
应用程序通过 inputs 或者 outputs 来与 Spring Cloud Stream 中 binder 对象交互。通过配置来绑定，而 Spring Cloud Stream 的 binder 对象负责与消息中间件交互。所以，我们只需要搞清楚如何与 Spring Cloud Stream 交互就可以方便使用消息驱动的方式。
通过使用 Spring Integration 来连接消息代理中间件以实现消息事件驱动。Spring Cloud Stream 为一些供应商的消息中间件产品提供了个性化的自动化配置，引用了发布-订阅、消费组、分区的三个核心概念

中间件的差异性导致了实际项目开发造成的一定困扰，如果用了两个消息队列的其中一种，后面的业务需求，想使用另外一种消息队列进行迁移，这时候无疑就是一个灾难，一大堆东西需要重新推倒重做，因为它跟我们的系统耦合了，这时候 Spring Cloud Stream 给我们提供了一种解耦合的方式。

在没有绑定器的概念的情况下，SpringBoot 应用要直接与消息中间件进行信息交互的时候，由于各消息中间件构建的初衷不同，它们的实现细节上会有较大的差异性，通过定义绑定器作为中间层，完美的实现了==应用程序与消息中间件细节之间的隔离==。通过向应用程序暴露统一的 Channel 通道，使得应用程序不需要再考虑各种不同的消息中间件实现。

## API与常用注解

|组成|说明|
|:-:|:-:|
|Middleware|中间件，目前只支持 RabbitMq 和 Kafka|
|Binder|Binder 是应用与消息中间件之间的封装，目前实行了 Kafka 和 RabbitMQ 的 Binder，通过 Binder 可以很方便的连接中间件，可以动态的改变消息类型（对应 Kafka）的topic，RabbitMQ 的 exchange），这些都可以通过配置文件实现|
|@Input|注解标识输入通道，通过该输入通道接收到的消息进入应用程序|
|@Output|注解标注输出通道，发布的消息将通过该通道离开应用程序|
|StreamListener|监听队列，用于消费者的队列的消息接收|
|@EnableBinding|指信道 channel 和 exchange 绑定在一起|

## 消息驱动-生产者

* POM
    ``` xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-stream-binder-kafka</artifactId>
    </dependency>
    ```
    
* YML
    ``` yml
    server:
        port: 8801

    spring:
        application:
            name: cloud-stream-provider
    cloud:
        stream:
            bindings:
                my_channel_input:
                    destination: study
                my_channel_output:
                    destination: study
            default-binder: kafka
    kafka:
        bootstrap-servers: localhost:9092
        producer:
            key-serializer: org.apache.kafka.common.serialization.ByteArraySerializer
            value-serializer: org.apache.kafka.common.serialization.ByteArraySerializer

    eureka:
        client:
            register-with-eureka: true # 是否将自己注册进 EurekaServer 默认为 true
            fetch-registry: true # 是否从 EurekaServer 抓取已有的信息，默认为 true 。集群必须设置为true才能配合ribbon使用负载均衡
            service-url:
                defaultZone: http://eureka7001.com:7001/eureka
    ```

* 创建 Sink 接口
    ``` java
    public interface Sink {
        String INPUT = "input";
        @Input("input")
        SubscribableChannel input();
    }
    ```

* 创建 Source 接口
    ``` java
    public interface Source {
        String OUTPUT = "output";
        @Output
        MessageChannel output();
    }
    ```

* 创建通道
    ``` java
    public interface MyChannel {
        String MY_CHANNEL_INPUT = "my_channel_input"; // 订阅通道的名称
        String MY_CHANNEL_OUTPUT = "my_channel_output"; // 发消息通道的名称
        @Output(MY_CHANNEL_OUTPUT)
        MessageChannel sendMessage();

        @Input(MY_CHANNEL_INPUT)
        SubscribableChannel receiveMessage();

    }
    ```

* 发布广播
    ``` java
    @EnableBinding(MyChannel.class)
    public class ProviderServiceImpl implements IProviderService {

        @Resource(name = MyChannel.MY_CHANNEL_OUTPUT)
        private MessageChannel messageChannel;

        @Override
        public String send() {
            String uuid = UUID.randomUUID().toString();
            messageChannel.send(MessageBuilder.withPayload(uuid).build());
            System.out.println(uuid);
            return null;
        }
    }
    ```

## 消息驱动-消费者
``` java
@StreamListener(MyChannel.MY_CHANNEL_INPUT)
public void receive(Message<String> message) {
    System.out.println("订阅者：" + message.getPayload());
}
```

## 重复消费
比如在如下场景中，订单系统做集群部署，都会从 RabbitMQ 中获取订单信息，如果一个订单同时被两个服务获取到，那么就会造成数据错误，一定要避免这个情况。可以使用 Stream 中的消息分组来解决。
在 Stream 中处于同一个 group 中的多个消费者是竞争关系，就能够保证消息只会被其中一个应用消费一次。不同组是可以全面消费的，同一组内会发生竞争关系，只有其中一个可以消费

## 分组配置
默认分组 group 是不同的，组流水号不一样，被认为不同组，会导致重复消费

为消费者设置分组
``` yml
spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      bindings:
        my_channel_input:
          destination: study
          group: groupA
      default-binder: kafka
```

# Sleuth
[Spring Cloud Sleuth](https://docs.spring.io/spring-cloud-sleuth/docs/current/reference/html/)
在微服务框架中，一个由客户端发起的请求在后端系统中会经过多个不同的服务节点调用来协同产生最后的请求结果，每一个前端请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延时或错误都会引起整个请求最后的失败。


