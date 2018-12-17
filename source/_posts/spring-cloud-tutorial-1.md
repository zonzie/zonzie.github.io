---
title: spring_cloud_tutorial_1
date: 2018-12-17 11:06:27
tags: 
    - spring_cloud
categories: spring_cloud
description: the basic tutorial about spring cloud
---

### 微服务架构基础
- 理论基础: [**康威定律**](http://www.melconway.com/Home/Committees_Paper.html)
- 起源: 来自于martin Fowler的一篇博文: <https://www.cnblogs.com/liuning8023/p/4493156.html>
- 基本概念: 一种系统架构上的设计风格, 主旨是将原本独立的系统拆分成多个小的服务,这些服务各自在独立的进程中运行, 服务之间基于统一的协议进行通信协作, 被拆分的每一个小型服务都围绕着系统中的某一项或者一些耦合程度很高的业务功能进行构建, 每项服务都维护着自身的业务开发, 数据存储等, 不同的服务可以使用不同的语言来编写
- **单体系统的问题**:
  - **复杂性**: 随着项目需求的不断增加, 单体应用会越来越臃肿, 可维护性, 灵活性越来越低, 维护成本越来越高, 模块越来越多, 模块的边界模糊, 依赖关系混乱不清, 修复一个bug可能会引入新的bug
  - **技术债务**: 开发时间紧张, 需求的变更, 人员变更等, 会导致逐渐形成技术债务,并且越积越多. "not broken, don't fix", 但是有技术债务是正常的, 就像金融债务, 只要偿还债务的速度大于债务的增长速度, 就是良性债务
  - **部署风险**: 随着项目越来越大, 每一次部署风险也会越来越大, 因为大的项目涉及到的功能也多, 每次上线都会有大量的功能变更和缺陷修复, 同样会加大出错的概率, 参见[墨菲定律](https://baike.baidu.com/item/%E5%A2%A8%E8%8F%B2%E5%AE%9A%E5%BE%8B/746284?fr=aladdin)
  - **可靠性差**
  - **扩展能力有限**
- **微服务的优点**
  - 基本解决了上述单体应用的缺点
  - 开发简单
  - 技术栈灵活
  - 服务独立无依赖
  - 独立按需扩展
  - 可用性高
- **微服务架构面临的问题**
  - 多服务运维难度
  - 分布式系统的复杂性: 难免会遇到的问题, 网络延迟, 分布式事物, 数据一致性等
  - 接口的一致性, 系统部署依赖: 修改一个接口, 调用了这个接口的服务都需要做调整
  - 服务间通信成本
  - 系统集成测试
  - 重复工作
  - 性能监控
- **微服务的设计原则**:
  - 单一职责原则: 一个单元只应该关注整个系统功能中单独的, 有界限的一部分
  - 服务自治原则: 每个微服务应该具备独立的业务能力, 依赖和运行环境. 服务是一个独立的业务单元, 应该和其他的服务高度解耦, 每个服务从开发, 测试, 构建, 部署都应该可以独立运行,而不依赖其他的服务
  - **通信机制**: spring cloud默认使用REST模式的http协议, dubbo, motan, dubbox基于netty实现自定义的通信协议
  - **微服务的粒度**

### 微服务开发框架--Spring Cloud
#### 常用组件
- eureka
- ribbon
- hystrix
- feign
- zuul
- config
- slenth

#### 服务发现组件--Eureka
- eureka和zookeeper的对比
  - CAP原则, 分布式服务, 优先保证AP, 需要强一致性, 优先保证CP, [两者的对比](http://dockone.io/article/78)
- 服务提供方, 消费方, 服务发现组件的关系
  1. 各个服务启动, 将自己的网络地址等信息注册到服务发现组件
  2. 消费者从服务发现组件查询服务提供者的网络地址, 选择该地址调用服务提供方的接口
  3. 微服务和服务发现组件使用一定的机制通信, 发现组件长时间无法与服务实例通信, 就注销该实例
  4. 服务提供方的网络地址变更, 会重新注册到服务发现组件, 无需人工修改
- 常见的架构:
  - ![微服务架构](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/microservice_construct.jpg)
- 组件:
  - eurka server
  - eureka client

##### eureka服务搭建
###### eureka server端
- 需要的jar
```xml
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

- 需要的注解:
  - 在应用主类上添加注解: `@EnableEurekaServer`
- 需要的最少配置:
```yml
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

- 访问地址:
  - <http://localhost:8080>
- 常用的配置属性
  - `eureka.client.register-with-eureka`: 表示是否把自己注册到eureka
  - `eureka.client.fetch-registry`: 表示是否从注册表上拉取注册信息
  - `eureka.client.serviceUrl.defaultZone`: 表示与eureka server的交互地址,拉取服务列表,注册服务, 都需要这个地址, 多个地址使用`,`分隔
  - `spring.application.name`: 用户指定注册到Eureka Server上的应用名称
  - `eureka.instance.prefer-ip-address`: 表示注册自己的ip到Eureka Server

##### eureka client端
- 需要的jar
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```
- 需要在主类添加的注解: `@EnableEurekaClient` 或者 `@EnableDiscoveryClient`
- 配置:
```yml
spring:
  application:
    name: client
server:
  port: 8082
eureka:
  client:
    service-url:
      defaultZone: http://localhost:1111/eureka/
```

##### 更多的说明
  - 服务续约:
    - `eureka.instance.lease-renewal-interval-in-seconds`: 用于定义服务续约任务的调用间隔时间, 默认是30s
    - `lease-expiration-duration-in-seconds`: 用于定义服务失效的时间, 默认是90s
  - 获取服务:
    - `eureka.client.registry-fetch-interval-seconds`: 缓存清单的更新时间
  - 服务下线:
    - 正常的服务下线或者重启, 会发送REST请求到Eureka Server, 服务端接到请求, 会将服务状态置为下线, 并且把下线事件传播出去
  - 注册中心
    - 失效剔除:
      - 如果服务没有正常下线, eureka server 不会收到下线的请求, Eureka Server会在启动时创建一个定时任务, 每隔一段时间把没有按时续约的服务剔除出去
    - 自我保护:
      - 服务在注册到Eureka Server后, 会维护一个心跳连接, 告诉eureka自己还活着, 如果心跳失败的比例在15分钟内低于85%, eureka server会将实例列表保护起来,不让实例过期, 这时如果实例出现问题, 调用方容易拿到实际不存在的实例,因此, 客户端需要有请求重试机制, 或者断路器
      - 测试环境关闭自我保护机制:
        - `eureka.server.enable-self-preservation`: 默认为true, 改成false
      - 修改阈值: `eureka.server.renewal-percent-threshold`: 默认值为0.85
    - eureka Server 和 client之间通信也是通过REST模式的http协议, [这是文档](https://github.com/Netflix/eureka/wiki/Eureka-REST-operations)
  - `eureka.client`相关配置详细信息: 
    - `org.springframework.cloud.netflix.eureka.EurekaClientConfigBean`
  - `eureka.instance`相关配置详细信息: 
    - `org.springframework.cloud.netflix.eureka.EurekaInstanceConfigBean`
  - `eureka.server`相关配置详细信息: 
    - `org.springframework.cloud.netflix.eureka.server.EurekaServerConfigBean`

### 参考内容
- 微服务: <https://yq.aliyun.com/articles/2764>
- 康威定律: <https://yq.aliyun.com/articles/8611>
- eureka 文档: <https://github.com/Netflix/eureka/wiki>
- eureka rest api: <https://segmentfault.com/a/1190000014750346>
- spring cloud 文档: <http://cloud.spring.io/spring-cloud-static/Finchley.SR2/single/spring-cloud.html>
- 为什么不用zk做服务发现: <http://dockone.io/article/78>
- netflix 开源项目: <https://github.com/Netflix>

### 相关的demo:
  - demo: <https://gitee.com/zonzie/spring-cloud-demo>
  - spring cloud 相关的例子: <https://github.com/spring-cloud-samples/spring-cloud-contract-samples>