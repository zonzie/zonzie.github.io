title: rabbitMq基本用法
tags:
  - rabbitMq
categories:
  - mq
description: rabbitMq的基本用法,参考自:&laquo;RabbitMQ实战指南&raquo;
author: zonzie
date: 2018-08-01 10:17:00
---

## 简介
#### MQ简介
- MQ是啥?
    - MQ是Message Queue消息队列的缩写,是一种应用程序对应用程序的通信方法、应用程序通过写和检索入列队的针对应用程序的数据来进行通信，而不需要专用连接来链接它们。队列的使用除去了接收和发送应用程序同时执行的要求。
- 概况
    - 是分布式应用之间交换信息的一种技术,消息队列可以驻留在内存或者磁盘中,直到被程序读走,通过消息队列,应用程序可以独立执行,不需要消息收发者彼此的位置
- 基本概念
    1. 消息message
        - 消息是mq中最小的概念,本质上是一段数据,能被一个或者多个应用程序所理解,是传递信息的载体
    2. 队列Queue
        - 初始化队列: 用作消息的触发功能
        - 传输队列: 暂存待传消息,条件许可的情况下,通过管道将消息传送到其他队列
        - 目标队列: 是消息的目的地,可以长期存放
        - 死信队列: 当消息不能送到目标队列,也不再路由出去,则自动放入死信队列
- 应用场景:
    - 所有可以异步操作的功能都可以用mq

#### rabbitMq简介
- 简介: rabbitMq是一个由Erlang语言开发的AMQP的开源实现
- AMQP: Advanced Message Queuing Protocol, 高级消息队列协议,它是应用层协议的一个开放的标准,以解决众多的消息中间件的需求和拓扑结构问题,并不受产品,语言等条件的限制

## 安装
#### 安装
- 先安装erlang语言环境
    - 可以使用yum工具安装
    - 这里下载官方的tar包安装,下载源码包otp_src_21.0.tar.gz
        1. 先解压 `tar -zxvf otp_src_21.0.tar.gz`
        2. 进入目录后,设置安装目录 `./configure --prefix=/opt/erlang/`
        3. 如果出现报错信息如下,可以使用yum搜索相关的组件,并且安装即可: ![报错信息](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20180801153222.png)
        4. 然后`make && make install`
        5. 在`/etc/profile`配置环境变量即可
        6. 输入`erl`看是否安装成功
- 再安装rabbitMq
    - 官网下载`rabbitmq-server-generic-unix-3.7.7.tar`,下载的rabbitMq的版本要和erlang适配,具体适配的版本机见rabbitMq官网每个版本的说明
    - 解压后配置环境变量即可
    - 运行: `rabbitmq-server`
- 如果有docker环境,可以直接拉取rabbitMQ的镜像,直接启动
	- 搜索相关的镜像: `docker search rabbitmq`
	- 拉取镜像: `docker pull rabbitmq`
	- 启动镜像,指定hostname,映射相关的端口,映射rabbitmq的数据库文件到宿主机: `docker run -tid --hostname rabbitmq_2 --name my_rabbitmq_2 -p 5672:5672 -p 15672:15672 -v /root/rabbitmq/lib/:/var/lib/rabbitmq/ rabbitmq:latest /bin/bash`
	- 进入镜像启动rabbitmq: `rabbitmq-server -detached`
	- 添加新的rabbitmq用户
	- 添加管理界面的插件 `rabbitmq-plugins enable rabbitmq_management`

## 基本操作
#### 基本概念
1. 生产者和消费者
    - 生产者: 消息投递的一方,投递的消息一般包含两部分:消息体(payload)和标签(tag),消息一般是带哦有业务逻辑结构的数据,标签用来描述这条消息,比如一个交换器的名称和一个路由键
    - 消费者: 接收消息的一方,消费者消费消息时,消费的只是消息体payload,在消息路由的过程中,消息的标签会被丢弃掉,存入队列的只有消息体,消费者也不会知道消息的生产者是谁
    - broker: 服务节点,一个broker可以看作一个简单的服务节点,
    ![topic](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/rabbitmq-structure.png)
<center>rabbitmq整体架构</center>
2. 队列
    - rabbitMq的内部对象,用来存储消息
3. 交换器,路由,绑定
    - Exchange: 交换器,决定最终投递到哪个队列的路由的一部分,消息被发送到Exchange,交换器将消息路由到一个或者多个队列中
        - fanout: 会将所有发送到交换器的消息路由到所有与交换器绑定的队列中
        - direct: 会把消息路由到所有的BindingKey和RoutingKey完全匹配的队列中
        ![direct](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/rabbitmq-direct.png)
<center>交换器direct</center>
        - topic: 将消息路由到RoutingKey和BindingKey满足一定的匹配规则的队列中
            - RoutingKey为点号`.`分隔的一些单词组成
            - BindingKey也是点号`.`分隔的字符串
            - BindingKey中存在两种特殊的字符`*`,`#`,用于模糊匹配,`*`用于匹配一个单词,`#`用于匹配多个单词
                - 例如: routingKey为`com.rabbitMq.client`的消息会路由到bindingKey为`*.rabbitMq.*`和`*.*.client`和`com.#`的队列
             ![topic](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/rabbitmq-topic.png)
<center>交换器topic</center>
        - headers: 根据发送内容中的headers属性进行匹配
    - RoutingKey: 路由键 生产者将消息发送到交换器时,一般会指定一个RoutingKey,它和交换器类型还有绑定键联合使用才能最终生效
    - BindingKey: 绑定键 在交换器的类型为direct或者topic时,消息将发送到routingKey和bindingKey匹配的队列中
	
#### 添加用户
- 添加一个root用户,密码是root`rabbitmqctl add_user root root`
- 设置所有权限: `rabbitmqctl set_permissions -p / root ".*" ".*" ".*"`
- 设置管理员角色: `rabbitmqctl set_user_tags root administrator`

#### 开启管理界面
- 使用命令 `rabbitmq-plugins enable rabbitmq_management`
- 重启rabbitMq `rabbitmqctl stop`, `rabbitmq-server -detached`
- 如果需要在其他机器上访问,需要放开15672端口,centos7使用firewall而不是iptables,需要注意

#### 消息的流转过程
- **生产者**:
	1. 生产者连接到RabbitMQ Broker,建立一个连接(Connection),开启一个信道(Channel)
	2. 生产者声明一个交换器,并且设置相关的属性,比如交换机类型,是否持久化等
	3. 生产者声明一个队列并且设置相关的属性,比如是否排他,是否持久化,是否自动删除等
	4. 生产者通过路由键将交换器和队列绑定起来
	5. 生产者发送消息到RabbitMq Broker,其中包含路由键,交换器等信息(tag和payload两部分)
	6. 相应的交换器根据接收到的路由键查找相匹配的队列
	7. 如果找到,将从生产者发过来的消息存入到相应的队列
	8. 如果没有找到,则根据生产者配置的属性选择丢弃还是回退给生产者
	9. 关闭信道
	10. 关闭连接
- **消费者**:
	1. 消费者连接到RabbitMQ Broker, 建立一个连接(Connection),开启一个信道(Channel)
	2. 消费者向RabbitMQ Broker 请求消费相应的队列中的消息,可能会设置相应的回调函数,以及做一些准备工作
	3. 等待RabbitMQ Broker 回应并投递相应的队列中的消息,消费者接收消息
	4. 消费者确认(ack)接收到的消息
	5. RabbitMQ从队列中删除相应的已经被确认的消息
	6. 关闭信道
	7. 关闭连接

## 实践
#### 示例代码

- **首先需要导入依赖**

```xml
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>4.2.1</version>
    </dependency>
```

- **生产者代码:** 

```java
import com.rabbitmq.client.*;
import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * rabbitMq 生产者
 * @author zonzie
 * @date 2018/6/30 15:43
 */
public class RabbitProducer {
    private static final String EXCHANGE_NAME = "exchange_demo";
    private static final String ROUTING_KEY = "routingkey_demo";
    private static final String QUEUE_NAME = "queue_demo";
    private static final String IP_ADDRESS = "192.168.198.128";
    private static final int PORT = 5672;// rabbitmq 服务端默认的端口

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(IP_ADDRESS);
        factory.setPort(PORT);
        factory.setUsername("root");
        factory.setPassword("root");
        // 创建连接
        Connection connection = factory.newConnection();
        // 创建信道
        Channel channel = connection.createChannel();
        // 创建一个type="direct",持久化的,非自动删除的交换器
        AMQP.Exchange.DeclareOk direct = channel.exchangeDeclare(EXCHANGE_NAME, "direct", true, false, null);
        // 创建一个持久化的,非排他的,非自动删除的队列
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        // 将交换器与队列通过路由键绑定,因为交换器类型为direct,所以ROUTING_KEY和BINDING_KEY保持一致,这里使用ROUTING_KEY就行
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);
        // 发送一条持久化的消息:hello world!
        String message = "hello world";
        channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
        // 关闭资源
        channel.close();
        connection.close();
    }
}
```

- **消费者代码:**

```java
import com.rabbitmq.client.*;
import java.io.IOException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

/**
 * rabbitMq 消费者
 * @author zonzie
 * @date 2018/6/30 16:30
 */
public class RabbitConsumer {
    private static final String QUEUE_NAME = "queue_demo";
    private static final String IP_ADDRESS = "192.168.198.128";
    private static final int PORT = 5672;

    public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
        Address[] addresses = {new Address(IP_ADDRESS, PORT)};
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUsername("root");
        factory.setPassword("root");
        // 创建连接
        Connection connection = factory.newConnection(addresses);
        // 创建信道
        final Channel channel = connection.createChannel();
        // 设置客户端最多接收未被ACK的消息的个数
        channel.basicQos(64);
        DefaultConsumer defaultConsumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("recv message: " + new String(body));
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        };
        channel.basicConsume(QUEUE_NAME, defaultConsumer);
        // 等待回调函数执行完毕, 关闭资源
        TimeUnit.SECONDS.sleep(20);
        channel.close();
        connection.close();
    }
}
```

#### 在spring中的使用-rabbitTemplate

- **相关的spring配置:** 

```yml
spring:
  application:
    name: spring-boot-rabbitmq
  rabbitmq:
    host: 192.168.198.128
    port: 5672
    username: root
    password: root
    # 设置手动确认
    publisher-confirms: true
```

- **生产者: 配置队列和交换器,还有绑定关系**

```java
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;


/**
 * @author zonzie
 * @date 2018/7/12 18:08
 */
@Configuration
public class AmqpConfig {

    /**
     * 创建一个队列 -> "hello"
     */
    @Bean
    public Queue queue() {
        HashMap<String, Object> map = new HashMap<String, Object>();
        // 设置超时时间
        map.put("x-message-ttl", 1000);
        // 设置死信交换器
        map.put("x-dead-letter-exchange", "dead-letter-exchange");
        // 设置死信路由键
        map.put("x-dead-letter-routing-key", "dead-letter-routing");
        return new Queue("hello",false, false, false, map);
    }

    /**
     * 创建一个队列 -> "object"
     */
    @Bean
    public Queue objectQueue() {
        return new Queue("object");
    }

    /**
     * 创建一个队列 -> "test"
     */
    @Bean
    public Queue testQueue() {
        return new Queue("test");
    }

    /**
     * 死信队列
     */
    @Bean
    public Queue deadQueue() {
        return new Queue("deadqueue");
    }

    /**
     * 死信交换器
     */
    @Bean
    public DirectExchange deadExchange() {
        return new DirectExchange("dead-letter-exchange");
    }

    /**
     * 绑定死信队列和死信交换器
     */
    @Bean
    public Binding deadBinding() {
        return BindingBuilder.bind(deadQueue()).to(deadExchange()).with("dead-letter-routing");
    }

    /**
     * 创建一个direct交换器
     */
    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange("com.zonzie.directtest");
    }

    /**
     * 通过bindingKey -> "hellotest", 绑定queue->"hello"和上面的交换器
     */
    @Bean
    public Binding bindingKey() {
        return BindingBuilder.bind(queue()).to(directExchange()).with("hellotest");
    }

    /**
     * 创建topic交换器
     */
    @Bean
    public TopicExchange topicExchange() {
        return new TopicExchange("com.zonzie.topictest");
    }

    /**
     * 通过bindingKey -> "com.*.test" 绑定queue->"test"和上面的topicExchange
     */
    @Bean
    public Binding topicBindKey() {
        return BindingBuilder.bind(testQueue()).to(topicExchange()).with("com.*.test");
    }

    /**
     * 通过bindingKey -> "com.#" 绑定 queue->"hello"和topicExchange
     */
    @Bean
    public Binding topicBindKey2() {
        return BindingBuilder.bind(queue()).to(topicExchange()).with("com.#");
    }
}
```

- **生产者: 设置rabbitTemplate**

```java
/**
 * @author zonzie
 * @date 2018/7/13 17:48
 */
@Configuration
@Slf4j
public class AmqpTemplateConfig{

    @Bean(name = "myTemplate")
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        // 设置messageConverter,确定消息传递时序列化的方式
        rabbitTemplate.setMessageConverter(messageConverter());
        // 设置重试次数
        RetryTemplate retryTemplate = new RetryTemplate();
        ExponentialBackOffPolicy exponentialBackOffPolicy = new ExponentialBackOffPolicy();
        exponentialBackOffPolicy.setInitialInterval(500);
        exponentialBackOffPolicy.setMultiplier(10.0);
        exponentialBackOffPolicy.setMaxInterval(10000);
        retryTemplate.setBackOffPolicy(exponentialBackOffPolicy);
        rabbitTemplate.setRetryTemplate(retryTemplate);
        // 设置回调方法
        rabbitTemplate.setConfirmCallback(new ConfirmCallbackDemo());
        // 失败后return回调
        rabbitTemplate.setReturnCallback(new ReturnCallBackDemo());
        // return 回调需要设置,不然不会生效
        rabbitTemplate.setMandatory(true);
        // 事务
//        rabbitTemplate.setChannelTransacted(true);
//        rabbitTemplate.setExchange();
        return rabbitTemplate;
    }

    private Jackson2JsonMessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}

// ---------------------
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.support.CorrelationData;

/**
 * 手动确认回调的方法
 * @author zonzie
 * @date 2018/7/14 15:31
 */
@Slf4j
public class ConfirmCallbackDemo implements RabbitTemplate.ConfirmCallback {
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        log.info(" 消息id:" + correlationData);
        if (ack) {
            log.info("消息发送确认成功");
        } else {
            log.info("消息发送确认失败:" + cause);
        }
    }
}

//---------------------
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.core.RabbitTemplate;

/**
 * 设置消息消费失败后回调的方法
 * @author zonzie
 * @date 2018/7/15 13:07
 */
@Slf4j
public class ReturnCallBackDemo implements RabbitTemplate.ReturnCallback {
    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
        System.out.println("return--message:"+new String(message.getBody())+",replyCode:"+replyCode+",replyText:"+replyText+",exchange:"+exchange+",routingKey:"+routingKey);
    }
}
```

- **用到的实体**

```java
import lombok.Data;

/**
 * @author zonzie
 * @date 2018/7/13 15:08
 */
@Data
public class User {
    private String username;
    private String password;
}
```

- **生产者: 发送消息**

```java
import com.zonzie.domian.User;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.support.CorrelationData;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import java.util.UUID;

/**
 * @author zonzie
 * @date 2018/7/13 14:22
 */
@RestController
@RequestMapping(value = "sender", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
@Slf4j
public class RabbitSender {

    @Resource(name = "myTemplate")
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/send")
    public String send(String context) {
        System.out.println(context);
        CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
//        rabbitTemplate.convertAndSend("hello", context, correlationData);
//        rabbitTemplate.convertAndSend("test","helloTest");
//        rabbitTemplate.convertAndSend("com.zonzie.directtest","hellotest","helloTEST");
        rabbitTemplate.convertAndSend("com.zonzie.topictest", "com.zonzie.test",context);
//        rabbitTemplate.convertAndSend("hellotest","helloTEST");

        return "success";
    }

    @GetMapping("/user")
    public String sendUser() {
        User user = new User();
        user.setUsername("ybq");
        user.setPassword("123");
        rabbitTemplate.convertAndSend("object",user);
        return "bingo!";
    }
}
```

- **消费者: 消费者配置**

```java
/**
 * @author zonzie
 * @date 2018/7/13 17:48
 */
@Configuration
@Slf4j
public class AmqpConsumeConfig{

    @Bean(name="myListenContainer")
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(ConnectionFactory connectionFactory, @Qualifier("rabbitTransactionManager") PlatformTransactionManager manager) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        // 设置messageConverter,需要同生产者保持一致
        factory.setMessageConverter(messageConverter());
        // 设置事务管理,事务管理和手动确认的方式只能使用一种
//        factory.setChannelTransacted(true);
//        factory.setTransactionManager(manager);
        factory.setConnectionFactory(connectionFactory);
        // 设置手动应答模式
        factory.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        return factory;
    }

    @Bean(name = "rabbitTransactionManager")
    public RabbitTransactionManager getTransactionManager(ConnectionFactory connectionFactory) {
        return new RabbitTransactionManager(connectionFactory);
    }

//    另一种配置消费者的方式
//    @Bean
//    public SimpleMessageListenerContainer messageContainer(ConnectionFactory connectionFactory) {
//        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
//        container.setMessageConverter(messageConverter());
//        container.setQueues(new Queue("hello"), new Queue("object"));
//        container.setExposeListenerChannel(true);
//        container.setMaxConcurrentConsumers(1);
//        container.setConcurrentConsumers(1);
//        container.setAcknowledgeMode(AcknowledgeMode.MANUAL);
//        container.setMessageListener(new ChannelAwareMessageListener() {
//
//            @Override
//            public void onMessage(Message message, com.rabbitmq.client.Channel channel) throws Exception {
//                byte[] body = message.getBody();
//                log.info("消费端接收到消息 : " + new String(body));
//                  // 手动确认消息已被消费
//                channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
//            }
//        });
//        return container;
//    }

    private Jackson2JsonMessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
```

- **消息监听并且消费**

```java
import com.google.common.base.Objects;
import com.rabbitmq.client.Channel;
import com.zonzie.domian.User;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.amqp.support.AmqpHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.stereotype.Component;

import java.io.IOException;

/**
 * @author zonzie
 * @date 2018/7/13 15:27
 */
@Slf4j
@Component
public class ObjectReceiver {

     /**
        消息的标识，false只确认当前一个消息收到，true确认所有consumer获得的消息
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        ack返回false，并重新回到队列，api里面解释得很清楚
        channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
        拒绝消息
        channel.basicReject(message.getMessageProperties().getDeliveryTag(), true);
        如果消息没有到exchange,则confirm回调,ack=false
        如果消息到达exchange,则confirm回调,ack=true
        exchange到queue成功,则不回调return
        exchange到queue失败,则回调return(需设置mandatory=true,否则不回回调,消息就丢了)
    */
    @RabbitHandler
    @RabbitListener(containerFactory = "myListenContainer", queues = "object")
    public void process(User user, @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag, Channel channel) throws IOException {
        System.out.println(user);
        // 确认消息收到
        channel.basicAck(deliveryTag,false);
        System.out.print("这里是接收者1答应消息： ");
        System.out.println("SYS_TOPIC_ORDER_CALCULATE_ZZ_FEE process1  : " + user);
    }

    @RabbitHandler
    @RabbitListener(containerFactory = "myListenContainer",queues = {"hello","test"})
    public void process(String hello, Channel channel, @Header(AmqpHeaders.DELIVERY_TAG) long deliverTag) throws IOException {
        System.out.println("Receiver: " + hello);
        if(Objects.equal(hello, "tttt")) {
            // 对于业务中遇到的一些不满足条件的消息,使用channel.reject(),或者channel.Nack(),会触发生产者的returnCallBack配置并且被处理或者会被发送到死信队列
            channel.basicReject(deliverTag, false);
        }
        channel.basicAck(deliverTag, false);
    }

    @RabbitHandler
    @RabbitListener(containerFactory = "myListenContainer",queues = {"test"})
    public void process2(String hello, Channel channel, @Header(AmqpHeaders.DELIVERY_TAG) long deliverTag) throws IOException {
        System.out.println("Receiver222: " + hello);
        channel.basicAck(deliverTag, false);
    }

}
```

----------

## 集群配置
#### 单机多节点集群配置
- 一台机器部署多个rabbitMQ服务节点,需要确保每个节点都有独立的名称,数据存储位置,端口号(包括插件的端口号),我们在主机名为node1的机器上创建一个由rabbit1@node1,rabbit2@node1和rabbit3@node1这三个节点组成的RabbitMQ集群
- 没有安装任何插件的情况,依次启动三个节点:
    1. `RABBITMQ_NODE_PORT=5672 RABBITMQ_NODENAME=rabbit1 rabbitmq-server -detached`
    2. `RABBITMQ_NODE_PORT=5673 RABBITMQ_NODENAME=rabbit2 rabbitmq-server -detached`
    3. `RABBITMQ_NODE_PORT=5674 RABBITMQ_NODENAME=rabbit3 rabbitmq-server -detached`
- 如果安装了managemnet插件,还需要配置插件的端口号,不然按照上面的启动方式会报错
    1. `RABBITMQ_NODE_PORT=5672 RABBITMQ_NODENAME=rabbit1 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15672}]" rabbitmq-server -detached`
    2. `RABBITMQ_NODE_PORT=5673 RABBITMQ_NODENAME=rabbit2 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15673}]" rabbitmq-server -detached`
    3. `RABBITMQ_NODE_PORT=5674 RABBITMQ_NODENAME=rabbit3 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15674}]" rabbitmq-server -detached`
- 3个节点都启动了之后, 将rabbit2@node1节点加入到rabbit1@node1的集群之中,并且按照同样的方法,将rabbit3加入到集群之中
    1. `rabbitmqctl -n rabbit2@node1 stop_app`
    2. `rabbitmqctl -n rabbit2@node1 reset`
    3. `rabbitmqctl -n rabbit2@node1 join_cluster rabbit1@node1`
    4. `rabbitmqctl -n rabbit2@node1 start_app`
- 集群状态查看
    - `rabbitmqctl -n rabbit1@node1 cluster_status`

#### 多机多节点配置
1. 为了让每个节点能够识别其他节点,首先需要修改/etc/hosts文件,添加ip和节点名称的映射文件
```
192.168.0.1 node1
192.168.0.2 node2
192.168.0.3 node3
```
2. 编辑RabitMQ的cookie文件,确保每个节点的cookie文件使用的是同一个值,可以读取一个节点的cookie的值,将其复制到node2和node3节点中.cookie文件默认的路径为`/var/lib/rabbitmq/.erlang.cookie`或者`$HOME/.erlang.cookie`, cookie相当于密钥令牌,集群中的RabbitMQ节点需要通过交换密钥令牌以获得相互认证,不然在配置节点时会报错
3. 启动三个节点的RabbitMQ服务
    - 分别在三个节点下执行: `rabbitmq-server -detached`
4. 以node1为基准,分别将node2和node3加入到node1节点的集群中
    1. `rabbitmqctl stop_app`
    2. `rabbitmqctl reset`
    3. `rabbitmqctl join_cluster rabbit@node1`
    4. `rabbitmqctl start_app`
5. 如果关闭了集群中的所有的节点,则需要确保在启动的时候最后关闭的节点是第一个启动的,如果第一个启动的不是最后关闭的节点,那这个节点会等待最后关闭的节点启动,这个等待时间是30s,如果没有等到,那么这个节点的启动也会失败,最新的版本中会默认重试10次,每次30s
6. 如果最后一个节点因为某些异常而无法启动,可以通过`rabbitmqctl forget_cluster_node`命令将此节点剔除出集群
7. 如果集群中所有节点都因为某些原因非正常关闭,比如断电,那么集群中的节点都会认为还有节点在它后面关闭,此时需要调用`rabbitmqctl force_boot`强制启动某一个节点,其他节点才会正常启动

#### 剔除单个节点
- 有多种方法可以将一个节点剔除出集群
    1. 如果由于启动顺序的原因不得不剔除一个节点,有两种情况
        1. node2节点已经不再运行rabbitMQ了, 可以在node1或者node3中将其剔除: `rabbitmqctl forget_cluster_node rabbit@node2`
        2. 如果此时没有节点处于启动状态,需要剔除掉node1节点,可以在node2或者node3执行命令: `rabbitmqctl forget_cluster_node rabbit@node1 --offline`,这里如果不加"--offline",则需要保证执行命令的节点处于运行状态,如果node2和node3无法先行启动,可以加这个参数,在没有启动rabbitmq的情况下将node1剔除出节点
    2. 正常情况下,剔除一个节点的方法
        1. `rabbitmqctl stop_app`
        2. `rabbitmqctl reset`
        3. `rabbitmqctl start_app`

> **以上代码地址**: <https://github.com/zonzie/rabbitMQ_demo>