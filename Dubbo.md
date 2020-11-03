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