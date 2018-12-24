---
title: spring cloud笔记(二)-服务调用(Ribbon和Feign)
date: 2018-12-24 10:25:40
tags:
    - spring cloud
    - ribbon
    - feign
categories: spring cloud
description: an introduction to ribbon and feign
---

## 添加actuator
actuator提供了很多的监控端点, 可以使用http:{ip}:{port}/{endpoint}的形式访问这些端点, 来了解应用程序的运行状况, [官方文档](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#production-ready)<br>
需要的jar:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
actuator的常用的端点如下:

端点 | 描述
---|---
health | 显示应用程序的健康程度, 内容由HealthIndicator的实现类提供
autoconfig | 显示自动配置的信息
beans | 显示上下文中所有的spring bean
configprops | 显示所有的@ConfigurationProperties的配置属性的列表, `endpoints.configprops.enabled=false`可以关闭这个端点
dump | 显示线程活动的快照
env | 显示应用程序的环境变量
info | 显示应用程序的基本信息
metrics | 显示应用的度量标准信息
shutdown | 关闭应用程序(默认不启用, 需要设置`endpoints.shutdown.enabled=true`), POST请求
trace | 显示跟踪信息(默认是最近100个HTTP请求)

- `/metrics`: 返回当前应用的各类度量指标
  - `processors`: 处理器数量
  - `uptime`: 运行时间
  - `systemload.average`: 系统平均负载
  - `mem.*`: 内存概要信息
  - `heap.*`: 堆内存的使用情况
  - `nonheap.*`: 非堆内存的使用情况
  - `threads.*`: 线程使用情况
  - `classes.*`: 应用加载和卸载的类统计
  - `gc.*`: 垃圾收集器的详细信息
    - `gc.ps_scavenge.count`: 垃圾回收次数
    - `gc.ps_scavenge.time`: 垃圾回收消耗的时间
    - `gc.ps_marksweep.count`: 标记-清除算法的次数
    - `gc.ps_marksweep.time`:  标记-清除算法的消耗时间
  - `httpsessions.*`: tomcat会话使用情况, 只有嵌入式的tomcat才会有
  - `gauge.*`: HTTP请求的性能指标
  - `counter.*`: HTTP请求的性能指标
- 自定义统计值
  1. 注入`o.s.b.actuate.metrics.CounterService`或者`o.s.b.actuate.GaugeService`
  2. 调用CounterService的increment或者decrement方法

## [Ribbon 客户端侧负载均衡](https://github.com/Netflix/ribbon)([文档](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.1.0.RC3/single/spring-cloud-netflix.html#spring-cloud-ribbon))
- 服务端的负载均衡: F5, nginx
- 主要区别: 服务清单的维护位置

### **使用**:
- 依赖
  ```xml
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
  </dependency>
  ```
- 为RestTemplate添加`@LoadBalanced`的注解, 就可使其具备负载均衡的能力
- 使用时, 将域名换成服务名就行了

### **ribbon的默认配置**:

<style>
table th:first-of-type {
    width: 110px;
}
</style>

Bean Type | Bean Name | Class Name | description
---|---|---|---
IClientConfig | ribbonClientConfig | DefaultClientConfigImpl | ribbon的客户端配置
IRule | ribbonRule | ZoneAvoidanceRule | ribbon的负载均衡策略
IPing | ribbonPing | DummyPing | ribbon的实例检查策略
ServerList<Server> | ribbonServerList | ConfigurationBasedServerList | 服务实例清单的维护机制
ServerListFilter<Server> | ribbonServerListFilter | ZonePreferenceServerListFilter | 服务实例清单的过滤机制
ILoadBalancer | ribbonLoadBalancer | ZoneAwareLoadBalancer | 负载均衡器
ServerListUpdater | ribbonServerListUpdater | PollingServerListUpdater | 服务实例的更新策略

### **自定义配置**:
  1. 使用配置类
     ```java
     @Configuration
     public class RibbonConfiguration {
        @Bean
        public IRule ribbonRule() {
            // 随机的负载均衡策略
            return new RandomRule();
        }
     }
     ```
     ```java
     @Configuration
     @RibbonClient(name = "USER-GAOTU100-COM", configuration=RibbonConfiguration.class)
     public class UserRibbonConfig {

     }
     ```

  2. 在配置文件中修改默认配置(Camden SR4之后的版本支持)
     ```yml
        <clientName>.ribbon.NFLoadBalancerClassName: Should implement ILoadBalancer
        <clientName>.ribbon.NFLoadBalancerRuleClassName: Should implement IRule
        <clientName>.ribbon.NFLoadBalancerPingClassName: Should implement IPing
        <clientName>.ribbon.NIWSServerListClassName: Should implement ServerList
        <clientName>.ribbon.NIWSServerListFilterClassName: Should implement ServerListFilter
     ```
     ```yml
     USER-GAOTU100-COM:
        ribbon: 
            NFLoadBalancerRuleClassName: com.netflix.loadBanlancer.RandomRule
     ```
     可重写的配置请查看: `org.springframework.cloud.netflix.ribbon.RibbonClientConfiguration`
### **ribbon的全局配置**
  ```yml
  # 常用配置
  # 开启重试机制
  spring.cloud.loadbalancer.retry.enabled = true
  # 断路器的超时时间
  hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds = 10000
  # 请求处理的超时时间
  ribbon.ReadTimeout = 3000
  # 请求链接的超时时间
  ribbon.ribbon.ConnectTimeout = 3000
  # 当前实例的重试次数
  ribbon.MaxAutoRetries = 3
  # 切换实例的重试次数
  ribbon.MaxAutoRetriesNextServer = 2
  # USER-GAOTU100-COM.ribbon.ReadTimeout = 1000
  ```
  针对具体客户端的配置格式: `<client-name>.ribbon.<key>=<value>`<br/>
  所有的配置的位置: `com.netflix.client.config.CommonClientConfigKey`

## [Feign-声明式REST客户端](https://github.com/OpenFeign/feign)([文档](https://cloud.spring.io/spring-cloud-netflix/1.4.x/single/spring-cloud-netflix.html#spring-cloud-feign))
### 使用
- 添加依赖
  ```xml
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
  </dependency>
  ```
- 应用主类添加注解`@EnableFeignClients`
- 创建feign接口
  ```java
  @FeignClient(value = "USER-GAOTU100-COM", configuration = FeignConfig.class, fallback =
        UserFeignFallBackImpl.class)
  public class UserFeignClient() {
      @RequestMapping(
            value = {"user/v2/captcha_pair"},
            method = {RequestMethod.GET}
      )
      ResponseVO<SendPasscodeDTO> getCaptchaPair(@RequestParam("mobile") String var1);

      @RequestMapping(
              value = {"user/v2/validate_captcha"},
              method = {RequestMethod.GET}
      )
      ResponseVO validateCaptcha(ValidateCaptchaRequest var1);

      @RequestMapping(
              value = {"user/v2/send_passcode"},
              method = {RequestMethod.POST},
              headers = {"Content-Type=application/x-www-form-urlencoded"}
      )
      ResponseVO sendPassCode(SendPassCodeRequest var1);

      @RequestMapping(
              value = {"user/v2/validate_passcode"},
              method = {RequestMethod.POST},
              headers = {"Content-Type=application/x-www-form-urlencoded"}
      )
      ResponseVO validatePassCode(ValidatePassCodeRequest var1);

      @RequestMapping(
              value = {"user/v2/register"},
              method = {RequestMethod.POST},
              headers = {"Content-Type=application/x-www-form-urlencoded"}
      )
      ResponseVO<UserInfoVO> registerUser(RegisterUserRequest var1);

      @RequestMapping(
              value = {"user/v2/login"},
              method = {RequestMethod.POST},
              headers = {"Content-Type=application/x-www-form-urlencoded"}
      )
      ResponseVO<UserInfoVO> loginByPassword(LoginByPasswordRequest var1);
      
  ```
  ```java
  @Autowired
  private UserFeignClient userFeignClient;
  ```
- 默认配置下, 当前版本的feign注意事项:
  - 所有的参数必须添加注解,以及绑定参数的名称
  - 传递对象参数, 不加@RequestBody注解, 也会默认是一个RequestBody, 即使是GET请求
  - feign接口上不能加@RequestMapping注解, 要添加统一的前缀, 可以使用@FeignClient注解的path属性
  - GET请求传递参数不能传递对象参数, 只能分开添加@RequestParam, 或者传递一个Map<String, ?> param
  - 不支持类似@GetMapping这种包含请求方法语义的注解

### feign的自定义配置
#### feign的默认配置类: `org.springframework.cloud.netflix.feign.FeignClientsConfiguration`
#### 自定义配置
```java
/**
 * feign自定义配置
 */
@Configuration
public class FeignConfig {

    private final ObjectFactory<HttpMessageConverters> messageConverters;

    @Autowired
    public FeignConfig(ObjectFactory<HttpMessageConverters> messageConverters) {
        this.messageConverters = messageConverters;
    }

    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }

    @Bean
    public Decoder feignDecoder() {
        return new CustomDecoder(new SpringDecoder(messageConverters));
    }

    @Bean
    public Encoder feignEncoder() {
        return new CustomEncoder(messageConverters);
    }
}
```
添加到`@FeignClient(value = "USER-GAOTU100-COM", configuration = FeignConfig.class)`
##### 使用feign原生的注解
详细的用法: <https://github.com/OpenFeign/feign>
```java
/*
* 更换契约
*/
@Bean
public Contract feignContract() {
    return new feign.Contract.Default();
}
```
```java
@RequestLine("GET /{id}")
@Headers("Content-type: application/json")
public User findUserById(@Param("id") Long id)
```
##### 使用JAX-RS的注解
```xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-jaxrs</artifactId>
    <version>9.5.0</version>
</dependency>
<dependency>
    <groupId>javax.ws.rs</groupId>
    <artifactId>jsr311-api</artifactId>
    <version>1.1.1</version>
</dependency>
```
```java
/*
* 更换JAX-RS契约
*/
@Bean
public Contract feignContract() {
    return new JAXRSContract()
}
```
```java
@GET
@Path("/user/{id}")
@Produces({ "application/json"})
public User getUserById(@PathParam("id") Integer id);
```

##### feign对压缩的支持
```yml
feign.compression.request.enabled=true
feign.compression.response.enabled=true
```
更精确的配置
```yml
feign.compression.request.enabled=true
feign.compression.request.mime-types=text/xml,application/xml,application/json
feign.compression.request.min-request-size=2048
```

#### Feign的日志
```java
@Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
```

##### Logger.Level的选择:
1. NONE: 不记录任何日志
2. BASIC: 仅记录请求方法, URL, 响应状态码以及执行时间
3. HEADERS: 在BASIC级别的基础上, 记录请求和响应的header
4. FULL: 记录请求和响应的header, body和元数据
