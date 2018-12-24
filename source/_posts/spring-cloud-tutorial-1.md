---
title: spring cloud笔记(一)-springCloud和eureka简介
date: 2018-12-17 11:06:27
tags: 
    - spring cloud
    - eureka
categories: spring cloud
description: a basic tutorial about spring cloud, this one is about spring cloud eureka
---

### 微服务架构基础
- 理论基础: [**康威定律**](http://www.melconway.com/Home/Committees_Paper.html)
- 起源: 来自于martin Fowler的一篇[博文](https://www.martinfowler.com/articles/microservices.html): [这是翻译](https://www.cnblogs.com/liuning8023/p/4493156.html)
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
#### [常用组件](http://spring.io/projects/spring-cloud)
- eureka
- ribbon
- hystrix
- feign
- zuul
- config
- slenth
- [...](https://springcloud.cc/)

#### 服务发现组件--[Eureka](https://github.com/Netflix/eureka)
- eureka和zookeeper的对比
  - CAP原则([解释](http://www.ruanyifeng.com/blog/2018/07/cap.html)), 分布式服务, 优先保证AP, 需要强一致性, 优先保证CP, [两者的对比](http://dockone.io/article/78)
- 服务提供方, 消费方, 服务发现组件的关系
  1. 各个服务启动, 将自己的网络地址等信息注册到服务发现组件
  2. 消费者从服务发现组件查询服务提供者的网络地址, 选择该地址调用服务提供方的接口
  3. 微服务和服务发现组件使用一定的机制通信, 发现组件长时间无法与服务实例通信, 就注销该实例
  4. 服务提供方的网络地址变更, 会重新注册到服务发现组件, 无需人工修改
- 常见的架构:
  - ![微服务架构](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/microservice_construct.jpg)
  - ![eureka](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/spring_cloud_arch.png)
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
      - ![self_preservation](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/eureka_self_preservation.png)
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

### 全部的配置
- eureka.instance
```yml
#服务注册中心实例的主机名
eureka.instance.hostname=localhost
#注册在Eureka服务中的应用组名
eureka.instance.app-group-name=
#注册在的Eureka服务中的应用名称
eureka.instance.appname=
#该实例注册到服务中心的唯一ID
eureka.instance.instance-id=
#该实例的IP地址
eureka.instance.ip-address=
#该实例，相较于hostname是否优先使用IP
eureka.instance.prefer-ip-address=false

#用于AWS平台自动扩展的与此实例关联的组名，
eureka.instance.a-s-g-name=
#部署此实例的数据中心
eureka.instance.data-center-info=
#默认的地址解析顺序
eureka.instance.default-address-resolution-order=
#该实例的环境配置
eureka.instance.environment=
#初始化该实例，注册到服务中心的初始状态
eureka.instance.initial-status=up
#表明是否只要此实例注册到服务中心，立马就进行通信
eureka.instance.instance-enabled-onit=false
#该服务实例的命名空间,用于查找属性
eureka.instance.namespace=eureka
#该服务实例的子定义元数据，可以被服务中心接受到
eureka.instance.metadata-map.test = test

#服务中心删除此服务实例的等待时间(秒为单位),时间间隔为最后一次服务中心接受到的心跳时间
eureka.instance.lease-expiration-duration-in-seconds=90
#该实例给服务中心发送心跳的间隔时间，用于表明该服务实例可用
eureka.instance.lease-renewal-interval-in-seconds=30
#该实例，注册服务中心，默认打开的通信数量
eureka.instance.registry.default-open-for-traffic-count=1
#每分钟续约次数
eureka.instance.registry.expected-number-of-renews-per-min=1

#该实例健康检查url,绝对路径
eureka.instance.health-check-url=
#该实例健康检查url,相对路径
eureka.instance.health-check-url-path=/health
#该实例的主页url,绝对路径
eureka.instance.home-page-url=
#该实例的主页url,相对路径
eureka.instance.home-page-url-path=/
#该实例的安全健康检查url,绝对路径
eureka.instance.secure-health-check-url=
#https通信端口
eureka.instance.secure-port=443
#https通信端口是否启用
eureka.instance.secure-port-enabled=false
#http通信端口
eureka.instance.non-secure-port=80
#http通信端口是否启用
eureka.instance.non-secure-port-enabled=true
#该实例的安全虚拟主机名称(https)
eureka.instance.secure-virtual-host-name=unknown
#该实例的虚拟主机名称(http)
eureka.instance.virtual-host-name=unknown
#该实例的状态呈现url,绝对路径
eureka.instance.status-page-url=
#该实例的状态呈现url,相对路径
eureka.instance.status-page-url-path=/status
```

- eureka.server
```yml
################server 与 client 关联的配置#####################33
#服务端开启自我保护模式。无论什么情况，服务端都会保持一定数量的服务。避免client与server的网络问题，而出现大量的服务被清除。
eureka.server.enable-self-preservation=true
#开启清除无效服务的定时任务，时间间隔。默认1分钟
eureka.server.eviction-interval-timer-in-ms= 60000
#间隔多长时间，清除过期的delta数据
eureka.server.delta-retention-timer-interval-in-ms=0
#过期数据，是否也提供给client
eureka.server.disable-delta=false
#eureka服务端是否记录client的身份header
eureka.server.log-identity-headers=true
#请求频率限制器
eureka.server.rate-limiter-burst-size=10
#是否开启请求频率限制器
eureka.server.rate-limiter-enabled=false
#请求频率的平均值
eureka.server.rate-limiter-full-fetch-average-rate=100
#是否对标准的client进行频率请求限制。如果是false，则只对非标准client进行限制
eureka.server.rate-limiter-throttle-standard-clients=false
#注册服务、拉去服务列表数据的请求频率的平均值
eureka.server.rate-limiter-registry-fetch-average-rate=500
#设置信任的client list
eureka.server.rate-limiter-privileged-clients=
#在设置的时间范围类，期望与client续约的百分比。
eureka.server.renewal-percent-threshold=0.85
#多长时间更新续约的阈值
eureka.server.renewal-threshold-update-interval-ms=0
#对于缓存的注册数据，多长时间过期
eureka.server.response-cache-auto-expiration-in-seconds=180
#多长时间更新一次缓存中的服务注册数据
eureka.server.response-cache-update-interval-ms=0
#缓存增量数据的时间，以便在检索的时候不丢失信息
eureka.server.retention-time-in-m-s-in-delta-queue=0
#当时间戳不一致的时候，是否进行同步
eureka.server.sync-when-timestamp-differs=true
#是否采用只读缓存策略，只读策略对于缓存的数据不会过期。
eureka.server.use-read-only-response-cache=true

################server 自定义实现的配置#####################33
#json的转换的实现类名
eureka.server.json-codec-name=
#PropertyResolver
eureka.server.property-resolver=
#eureka server xml的编解码实现名称
eureka.server.xml-codec-name=

################server node 与 node 之间关联的配置#####################33
#发送复制数据是否在request中，总是压缩
eureka.server.enable-replicated-request-compression=false
#指示群集节点之间的复制是否应批处理以提高网络效率。
eureka.server.batch-replication=false
#允许备份到备份池的最大复制事件数量。而这个备份池负责除状态更新的其他事件。可以根据内存大小，超时和复制流量，来设置此值得大小
eureka.server.max-elements-in-peer-replication-pool=10000
#允许备份到状态备份池的最大复制事件数量
eureka.server.max-elements-in-status-replication-pool=10000
#多个服务中心相互同步信息线程的最大空闲时间
eureka.server.max-idle-thread-age-in-minutes-for-peer-replication=15
#状态同步线程的最大空闲时间
eureka.server.max-idle-thread-in-minutes-age-for-status-replication=15
#服务注册中心各个instance相互复制数据的最大线程数量
eureka.server.max-threads-for-peer-replication=20
#服务注册中心各个instance相互复制状态数据的最大线程数量
eureka.server.max-threads-for-status-replication=1
#instance之间复制数据的通信时长
eureka.server.max-time-for-replication=30000
#正常的对等服务instance最小数量。-1表示服务中心为单节点。
eureka.server.min-available-instances-for-peer-replication=-1
#instance之间相互复制开启的最小线程数量
eureka.server.min-threads-for-peer-replication=5
#instance之间用于状态复制，开启的最小线程数量
eureka.server.min-threads-for-status-replication=1
#instance之间复制数据时可以重试的次数
eureka.server.number-of-replication-retries=5
#eureka节点间间隔多长时间更新一次数据。默认10分钟。
eureka.server.peer-eureka-nodes-update-interval-ms=600000
#eureka服务状态的相互更新的时间间隔。
eureka.server.peer-eureka-status-refresh-time-interval-ms=0
#eureka对等节点间连接超时时间
eureka.server.peer-node-connect-timeout-ms=200
#eureka对等节点连接后的空闲时间
eureka.server.peer-node-connection-idle-timeout-seconds=30
#节点间的读数据连接超时时间
eureka.server.peer-node-read-timeout-ms=200
#eureka server 节点间连接的总共最大数量
eureka.server.peer-node-total-connections=1000
#eureka server 节点间连接的单机最大数量
eureka.server.peer-node-total-connections-per-host=10
#在服务节点启动时，eureka尝试获取注册信息的次数
eureka.server.registry-sync-retries=
#在服务节点启动时，eureka多次尝试获取注册信息的间隔时间
eureka.server.registry-sync-retry-wait-ms=
#当eureka server启动的时候，不能从对等节点获取instance注册信息的情况，应等待多长时间。
eureka.server.wait-time-in-ms-when-sync-empty=0

################server 与 remote 关联的配置#####################33
#过期数据，是否也提供给远程region
eureka.server.disable-delta-for-remote-regions=false
#回退到远程区域中的应用程序的旧行为 (如果已配置) 如果本地区域中没有该应用程序的实例, 则将被禁用。
eureka.server.disable-transparent-fallback-to-other-region=false
#指示在服务器支持的情况下, 是否必须为远程区域压缩从尤里卡服务器获取的内容。
eureka.server.g-zip-content-from-remote-region=true
#连接eureka remote note的连接超时时间
eureka.server.remote-region-connect-timeout-ms=1000
#remote region 应用白名单
eureka.server.remote-region-app-whitelist.
#连接eureka remote note的连接空闲时间
eureka.server.remote-region-connection-idle-timeout-seconds=30
#执行remote region 获取注册信息的请求线程池大小
eureka.server.remote-region-fetch-thread-pool-size=20
#remote region 从对等eureka加点读取数据的超时时间
eureka.server.remote-region-read-timeout-ms=1000
#从remote region 获取注册信息的时间间隔
eureka.server.remote-region-registry-fetch-interval=30
#remote region 连接eureka节点的总连接数量
eureka.server.remote-region-total-connections=1000
#remote region 连接eureka节点的单机连接数量
eureka.server.remote-region-total-connections-per-host=50
#remote region抓取注册信息的存储文件，而这个可靠的存储文件需要全限定名来指定
eureka.server.remote-region-trust-store=
#remote region 储存的文件的密码
eureka.server.remote-region-trust-store-password=
#remote region url.多个逗号隔开
eureka.server.remote-region-urls=
#remote region url.多个逗号隔开
eureka.server.remote-region-urls-with-name.

################server 与 ASG/AWS/EIP/route52 之间关联的配置#####################33
#缓存ASG信息的过期时间。
eureka.server.a-s-g-cache-expiry-timeout-ms=0
#查询ASG信息的超时时间
eureka.server.a-s-g-query-timeout-ms=300
#服务更新ASG信息的频率
eureka.server.a-s-g-update-interval-ms=0
#AWS访问ID
eureka.server.a-w-s-access-id=
#AWS安全密钥
eureka.server.a-w-s-secret-key=
#AWS绑定策略
eureka.server.binding-strategy=eip
#用于从第三方AWS 帐户描述自动扩展分组的角色的名称。
eureka.server.list-auto-scaling-groups-role-name=
#是否应该建立连接引导
eureka.server.prime-aws-replica-connections=true
#服务端尝试绑定候选EIP的次数
eureka.server.e-i-p-bind-rebind-retries=3
#服务端绑定EIP的时间间隔.如果绑定就检查;如果绑定失效就重新绑定。当且仅当已经绑定的情况
eureka.server.e-i-p-binding-retry-interval-ms=10
#服务端绑定EIP的时间间隔.当且仅当服务为绑定的情况
eureka.server.e-i-p-binding-retry-interval-ms-when-unbound=
#服务端尝试绑定route53的次数
eureka.server.route53-bind-rebind-retries=3
#服务端间隔多长时间尝试绑定route53
eureka.server.route53-binding-retry-interval-ms=30
#
eureka.server.route53-domain-t-t-l=10
```

- eureka.client
```yml
#该客户端是否可用
eureka.client.enabled=true
#实例是否在eureka服务器上注册自己的信息以供其他服务发现，默认为true
eureka.client.register-with-eureka=false
#此客户端是否获取eureka服务器注册表上的注册信息，默认为true
eureka.client.fetch-registry=false
#是否过滤掉，非UP的实例。默认为true
eureka.client.filter-only-up-instances=true
#与Eureka注册服务中心的通信zone和url地址
eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/

#client连接Eureka服务端后的空闲等待时间，默认为30 秒
eureka.client.eureka-connection-idle-timeout-seconds=30
#client连接eureka服务端的连接超时时间，默认为5秒
eureka.client.eureka-server-connect-timeout-seconds=5
#client对服务端的读超时时长
eureka.client.eureka-server-read-timeout-seconds=8
#client连接all eureka服务端的总连接数，默认200
eureka.client.eureka-server-total-connections=200
#client连接eureka服务端的单机连接数量，默认50
eureka.client.eureka-server-total-connections-per-host=50
#执行程序指数回退刷新的相关属性，是重试延迟的最大倍数值，默认为10
eureka.client.cache-refresh-executor-exponential-back-off-bound=10
#执行程序缓存刷新线程池的大小，默认为5
eureka.client.cache-refresh-executor-thread-pool-size=2
#心跳执行程序回退相关的属性，是重试延迟的最大倍数值，默认为10
eureka.client.heartbeat-executor-exponential-back-off-bound=10
#心跳执行程序线程池的大小,默认为5
eureka.client.heartbeat-executor-thread-pool-size=5
# 询问Eureka服务url信息变化的频率（s），默认为300秒
eureka.client.eureka-service-url-poll-interval-seconds=300
#最初复制实例信息到eureka服务器所需的时间（s），默认为40秒
eureka.client.initial-instance-info-replication-interval-seconds=40
#间隔多长时间再次复制实例信息到eureka服务器，默认为30秒
eureka.client.instance-info-replication-interval-seconds=30
#从eureka服务器注册表中获取注册信息的时间间隔（s），默认为30秒
eureka.client.registry-fetch-interval-seconds=30

# 获取实例所在的地区。默认为us-east-1
eureka.client.region=us-east-1
#实例是否使用同一zone里的eureka服务器，默认为true，理想状态下，eureka客户端与服务端是在同一zone下
eureka.client.prefer-same-zone-eureka=true
# 获取实例所在的地区下可用性的区域列表，用逗号隔开。（AWS）
eureka.client.availability-zones.china=defaultZone,defaultZone1,defaultZone2
#eureka服务注册表信息里的以逗号隔开的地区名单，如果不这样返回这些地区名单，则客户端启动将会出错。默认为null
eureka.client.fetch-remote-regions-registry=
#服务器是否能够重定向客户端请求到备份服务器。 如果设置为false，服务器将直接处理请求，如果设置为true，它可能发送HTTP重定向到客户端。默认为false
eureka.client.allow-redirects=false
#客户端数据接收
eureka.client.client-data-accept=
#增量信息是否可以提供给客户端看，默认为false
eureka.client.disable-delta=false
#eureka服务器序列化/反序列化的信息中获取“_”符号的的替换字符串。默认为“__“
eureka.client.escape-char-replacement=__
#eureka服务器序列化/反序列化的信息中获取“$”符号的替换字符串。默认为“_-”
eureka.client.dollar-replacement="_-"
#当服务端支持压缩的情况下，是否支持从服务端获取的信息进行压缩。默认为true
eureka.client.g-zip-content=true
#是否记录eureka服务器和客户端之间在注册表的信息方面的差异，默认为false
eureka.client.log-delta-diff=false
# 如果设置为true,客户端的状态更新将会点播更新到远程服务器上，默认为true
eureka.client.on-demand-update-status-change=true
#此客户端只对一个单一的VIP注册表的信息感兴趣。默认为null
eureka.client.registry-refresh-single-vip-address=
#client是否在初始化阶段强行注册到服务中心，默认为false
eureka.client.should-enforce-registration-at-init=false
#client在shutdown的时候是否显示的注销服务从服务中心，默认为true
eureka.client.should-unregister-on-shutdown=true

# 获取eureka服务的代理主机，默认为null
eureka.client.proxy-host=
#获取eureka服务的代理密码，默认为null
eureka.client.proxy-password=
# 获取eureka服务的代理端口, 默认为null
eureka.client.proxy-port=
# 获取eureka服务的代理用户名，默认为null
eureka.client.proxy-user-name=

#属性解释器
eureka.client.property-resolver=
#获取实现了eureka客户端在第一次启动时读取注册表的信息作为回退选项的实现名称
eureka.client.backup-registry-impl=
#这是一个短暂的×××的配置，如果最新的×××是稳定的，则可以去除，默认为null
eureka.client.decoder-name=
#这是一个短暂的编码器的配置，如果最新的编码器是稳定的，则可以去除，默认为null
eureka.client.encoder-name=

#是否使用DNS机制去获取服务列表，然后进行通信。默认为false
eureka.client.use-dns-for-fetching-service-urls=false
#获取要查询的DNS名称来获得eureka服务器，此配置只有在eureka服务器ip地址列表是在DNS中才会用到。默认为null
eureka.client.eureka-server-d-n-s-name=
#获取eureka服务器的端口，此配置只有在eureka服务器ip地址列表是在DNS中才会用到。默认为null
eureka.client.eureka-server-port=
#表示eureka注册中心的路径，如果配置为eureka，则为http://x.x.x.x:x/eureka/，在eureka的配置文件中加入此配置表示eureka作为客户端向注册中心注册，从而构成eureka集群。此配置只有在eureka服务器ip地址列表是在DNS中才会用到，默认为null
eureka.client.eureka-server-u-r-l-context=
```

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