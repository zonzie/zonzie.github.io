---
title: kafka的基本使用
date: 2018-04-18 12:50:22
tags: kafka
categories: kafka
description: kafka是用于构建实时数据管道和流应用程序。本文是它的基本操作方法,整理翻译自http://kafka.apache.org/
---
#### 一些基本信息
- Apache Kafka: 消息中间件的一种, 一般的消息中间件会有消息的生产者,消息的消费者,主题等概念,kafka也有,除此之外,kafka是一个分布式的流处理平台
- 作为一个流处理平台,主要有以下三点能力:
    1. 从流的记录中进行发布订阅,类似于企业消息系统中的消息队列
    2. 以一种高可用的方式存储记录流
    3. 实时的处理记录流
- 应用场景:
    1. 构建实时的数据流管道,在系统和应用之间可靠的获取数据
    2. 构建实时的流式处理应用,转变或者反应数据流
- 为了理解kafka做的事情,可以从以下了解到kafka的能力
    - 首先是几个概念
        1. kafka以一个集群的方式运行在一个或者多个服务器上而且可以跨越多个数据中心
        2. kafka集群在叫做topic的目录中存储流式的记录
        3. 每条记录包括一个key,一个value,一个timestamp
    - kafka包括以下四个核心API
        - producer API 
        - consumer API
        - Streams API
        - Connector API
#### 快速开始
- 下载kafka,地址:http://kafka.apache.org/downloads
- 下载二进制包,解压
```bash
tar -xzf kafka_2.11-1.1.0.tgz
cd kafka_2.11-1.1.0
```
- 启动服务,kafka需要使用zookeeper,如果没有下载单独的zookeeper,kafka提供了一个
    1. 启动kafka自带的zookeeper,可以使用nohup命令后台启动
    ```
    bin/zookeeper-server-start.sh config/zookeeper.properties
    ```
    2. 再启动kafka (我在启动时有报错,xxx:找不到名称或者服务,只需要再/etc/hosts文件中加入xxx到本地ip对应的名称中,再启动就可以了)
    ```
    bin/kafka-server-start.sh config/server.properties
    ```
    3. 创建一个topic
        - 创建一个叫做test的topic,并且只有一个副本,一个分区
        ```shell
        bin/kafka-topic.sh --create --zookeeper localhost:2181 --replication-factor 1 --partition 1 --topic test
        ```
        - 我们可以查看topic列表
        ```bash
        bin/kafka-topics.sh --list --zookeeper locahost:2181
        ```
        - 也可以通过配置broker在不存在topic时自动创建topic,以替代手动创建topic
    4. 发送一些消息
        1. kafka提供了命令行客户端可以从文件中或者标准输入中向kafka集群中发送消息
        ```
        bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
        ```
        然后就可以在标准输入中写消息了
        2. 开启命令行客户端的消费者
        ```bash
        bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
        ```
        然后之前输入的消息就会打印在控制台上
        3. 设置一个有多个broker的集群
            1. 复制server.properties文件<br/>
            `cp config/server/properties config/server-1.properties`<br>
            `cp config/server/properties config/server-2.properties`
            2. 修改其中的配置
            ```
            config/server-1.properties:
                broker.id=1
                listeners=PLAINTEXT://:9093
                log.dir=/tmp/kafka-logs-1
             
            config/server-2.properties:
                broker.id=2
                listeners=PLAINTEXT://:9094
                log.dir=/tmp/kafka-logs-2
            ```
            3. broker.id表示集群中每一个节点的唯一的,永久的名称,修改端口号和log目录是因为我们将多个broker运行在一台机器上,不修改将会导致多个broker的数据相互覆盖
            4. 此时我们可以将配置好的两个新的节点启动<br/>
            `bin/kafka-server-start.sh config/server-1.properties &` <br/>
            `bin/kafka-server-start.sh config/server-2.properties &`
            5. 现在我们可以创建一个新的topic,它拥有3个副本
            ```bash
            bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic myreplicated-topic
            ```
            6. 可以运行"describe topics"命令查看topic信息
            ```bash
            bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
            ```
            
