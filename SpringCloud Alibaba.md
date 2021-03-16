# Spring Cloud Alibaba

[Spring Cloud Alibaba 中文文档](https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md)

[Spring Cloud Alibaba Ref](https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/en-us/index.html)



# Nacos

Nacos：前四个字母分别为 Naming 和 Configuration 的前两个字母，最后的 s 为 Service

一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

[Nacos Home](https://nacos.io/zh-cn/)

[Github Nacos](https://github.com/alibaba/nacos)



## 启动

Nacos默认以集群方式启动，修改启动的批处理文件，将启动`Mode`从`cluster`改为`standalone`

启动成功后，本地访问`localhost:8848/nacos`访问监控平台，账号密码默认都是`nacos`





## 服务注册与发现

### 服务提供者

1. POM

    ``` xml
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    ```

2. YML

    ``` yml
    server:
      port: 9001
    spring:
      application:
        name: nacos-payment-provider
      cloud:
        nacos:
          discovery:
            server-addr: localhost:8848
    
    management:
      endpoints:
        web:
          exposure:
            include: *
    ```

3. 主启动

    ``` java
    @SpringBootApplication
    @EnableDiscoveryClient
    public class ProviderNacos9001MainApplication {
        public static void main(String[] args) {
            SpringApplication.run(ProviderNacos9001MainApplication.class ,args);
        }
    }
    ```

4. 业务类

    ``` java
    @Slf4j
    @RestController
    public class ProviderController {
        @Value("${server.port}")
        private String port;
    
        @GetMapping("/provider")
        public String provider() {
            return port;
        }
    }
    ```

    

启动后，观察 Nacos 监控面板

![](SpringCloud\Nacos服务注册.png)





### 服务提供者

Nacos自身支持负载均衡

![](SpringCloud\Nacos负载均衡.png)



1. POM

    ``` xml
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    ```

2. YML

    ``` yml
    server:
      port: 80
    spring:
      application:
        name: nacos-order-consumer
      cloud:
        nacos:
          discovery:
            server-addr: localhost:8848
    
    service-url:
      nacos-user-service: http://nacos-payment-provider # 便于后面指定微服务名称
    ```

3. 主启动

    ``` java
    @SpringBootApplication
    @EnableDiscoveryClient
    public class ConsumerNacos80MainApplication {
        public static void main(String[] args) {
            SpringApplication.run(ConsumerNacos80MainApplication.class ,args);
        }
    }
    ```

4. 配置类

    ``` java
    @Configuration
    public class Config {
        @Bean
        @LoadBalanced
        public RestTemplate getRestTemplate() {
            return new RestTemplate();
        }
    }
    ```

5. 业务类

    ``` java
    @RestController
    @Slf4j
    public class ConsumerController {
        @Autowired
        RestTemplate restTemplate;
    
        @Value("${service-url.nacos-user-service}")
        private String serverURL;
    
        @GetMapping("/consumer/provider")
        public String consumer() {
            return restTemplate.getForObject(serverURL + "/provider", String.class);
        }
    }
    ```

启动成功后，Nacos 监控页面可以看见三个微服务

![](SpringCloud\Nacos消费者注册.png)

访问`localhost/consumer/provider`访问测试



## CP与AP切换

C 是所有节点在同一时间看到的数据是一致的，而 A 的定义是所有的请求都会收到响应。

如果不需要存储服务级别的信息且服务实例是通过 nacos-client 注册，并能保持心跳上报，那么就可以选择 AP 模式。当前主流的服务如 Spring Cloud 和 Dubbo 服务，都适用于 AP 模式，AP 模式为了服务的可能性而减弱了一致性， 因此 AP 模式下只支持注册临时实例。

如果需要在服务级别编辑或者存储配置信息，那么 CP 是必须，K8S 服务和 DNS 服务则适用于 CP 模式。CP 模式下则支持注册持久化实例，此时则是以 Raft 协议为集群运行模式，该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误



```
curl -X PUT NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMaode&value=CP 
```



## 配置服务中心

1. POM

    ``` xml
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    ```

2. YML

    bootstrap

    ``` yml
    server:
      port: 3377
    
    spring:
      application:
        name: naocs-config-client
      cloud:
        nacos:
          discovery:
            server-addr: localhost:8848 # Nacos 服务注册中心地址
          config:
            server-addr: localhost:8848 # Nacos 作为配置中心地址
            file-extension: yml # 指定配置文件的格式
    ```

    application

    ``` yml
    spring:
      profiles:
        active: dev # 表示开发环境
    ```

    

3. 主启动

    ``` java
    @SpringBootApplication
    @EnableDiscoveryClient
    public class ConfigClientNacos3377MainApplication {
        public static void main(String[] args) {
            SpringApplication.run(ConfigClientNacos3377MainApplication.class ,args);
        }
    }
    ```



### 配置规则

Nacos 中的 dataid 的组成格式与 SpringBoot 配置文件中的匹配规则

在 Nacos Spring Cloud 中，`dataId` 的完整格式如下：

```plain
${prefix}-${spring.profiles.active}.${file-extension}
```

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 `spring.profiles.active` 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

最后的公式：`${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}`

![](E:\Study\MyNotes\SpringCloud\Nacos新建配置.png)



### 分组

namespace 可以用于区分部署环境，Group 和 DataID 逻辑上区分两个目标对象。

默认情况下：namespace = public，Group = DEFAULT_GROUP，默认 Cluster 是 DEFAULT



## 集群和持久化

默认 Nacos 使用嵌入式数据库实现数据的存储。所以，如果启动多个默认配置下的 Nacos 节点，数据存储是存在一致性问题的。为了解决这个问题， Nacos 采用了集中式存储的方式来支持集群化部署，目前只支持 MySQL 的存储