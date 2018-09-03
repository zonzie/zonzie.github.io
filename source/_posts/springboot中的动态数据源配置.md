---
title: springboot中的动态数据源配置
date: 2018-09-03 17:32:58
tags:
	- springboot
	- mysql
categories: springboot
description: 在spring项目中使用多数据源配置的基本方法
---
#### 在项目中使用多数据源
- 使用多数据源一般有两种解决办法
    - 在数据库之上加一层,项目在访问数据时,直接访问中间层,中间层解析sql,确定访问哪个数据库实例,或者访问多个实例,然后再汇总数据
    - 如果是简单的配置多数据源,或者配置读写分离,可以采用spring的动态数据源配置

##### 这里只说使用动态数据源的方法
- spring提供的动态数据源配置非常简单
- 基本思路如下:
    1. 分别初始化每个数据源
    2. 将所有的数据源全部放入spring提供的动态数据源对象中
    3. 将dao层使用的原本的数据源切换为动态数据源对象
    4. 在执行dao层的方法时,确定具体要访问的数据源
- 下面的例子基于springboot项目

##### 初始化数据源
###### 这里只添加两个数据源,做一个简单的读写分离配置
**1 首先是application.yml的配置**
```yml
# 数据源使用druid
druidconfig: &druidconfig
  type: com.alibaba.druid.pool.DruidDataSource
  driver-class-name: com.mysql.jdbc.Driver
  initialSize: 5
  minIdle: 5
  maxActive: 21
  maxWait: 60001
  timeBetweenEvictionRunsMillis: 60001
  minEvictableIdleTimeMillis: 300001
  validationQuery: SELECT 1 FROM DUAL
  testWhileIdle: true
  testOnBorrow: false
  testOnReturn: false
  poolPreparedStatements: true
  maxPoolPreparedStatementPerConnectionSize: 20
  filters: stat,wall,log4j
  connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
  useGlobalDataSourceStat: true
spring:
  datasource:
    url: jdbc:mysql://192.168.198.128:3306/operation?characterEncoding=utf8&useSSL=false
    username: root
    password: root
    <<: *druidconfig
  readSource:
    url: jdbc:mysql://192.168.198.128:3306/operation?characterEncoding=utf8&useSSL=false
    username: root
    password: root
    <<: *druidconfig
```
**2 然后是初始化数据源配置**
```java
import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import javax.sql.DataSource;

/**
 * @author itw_yaobq
 * @date 2018/2/28 17:12
 */
@Configuration
public class LoadDataSource {

    /**
     * 写库
     */
    @Primary
    @Bean(name = "writeDataSource")
    @ConfigurationProperties(prefix = "spring.dataSource")
    public DataSource writeDataSource() {
        return new DruidDataSource();
    }

    /**
     * 读库
     */
    @Bean(name = "readDataSource")
    @ConfigurationProperties(prefix = "spring.readSource")
    public DataSource readDataSource() {
        return new DruidDataSource();
    }
}
```
**3 jpa数据源配置**
```java
import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;

import javax.annotation.Resource;
import javax.persistence.EntityManagerFactory;
import javax.sql.DataSource;

/**
 * 修改jpa的自动配置,添加动态数据源
 *
 * @author itw_yaobq
 * @date 2018/2/28 17:22
 */
@Configuration
@EnableJpaRepositories(value = "com.taikang.operation.*.repository")
public class InitialDataSource {
    /**
     * 加载yml中的配置
     */
    @Resource
    private JpaProperties jpaProperties;

    /**
     * 这里的DataSource,注入动态数据源
     */
    @Resource(name = "routingDataSource")
    private DataSource dataSource;

    /**
     * 设置动态数据源,属性配置,扫描entity的位置
     */ 
    @Bean(name = "entityManagerFactoryBean")
    @Primary
    public LocalContainerEntityManagerFactoryBean entityManagerFactoryBean(EntityManagerFactoryBuilder builder) {
        return builder.dataSource(dataSource).properties(jpaProperties.getProperties()).packages("com.taikang.operation.*.repository.entity")
                //                .persistenceUnit("")
                .build();
    }

    /**
     * 配置实体工厂
     */
    @Bean(name = "entityManagerFactory")
    public EntityManagerFactory entityManagerFactory(EntityManagerFactoryBuilder builder) {
        return this.entityManagerFactoryBean(builder).getObject();
    }

    /**
     * 事务配置
     */
    @Bean(name = "transactionManager")
    public PlatformTransactionManager transactionManager(EntityManagerFactoryBuilder builder) {
        MyJpaTransactionManager myJpaTransactionManager = new MyJpaTransactionManager();
        myJpaTransactionManager.setEntityManagerFactory(this.entityManagerFactory(builder));
        return myJpaTransactionManager;
    }
}
```
**4 事务管理类**
```java
import com.taikang.operation.core.common.constant.DataSourceType;
import com.taikang.operation.core.config.readwriteseparationconfig.dynamicdatasourceconfig.DynamicDataSourceHolder;
import lombok.extern.slf4j.Slf4j;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.support.DefaultTransactionStatus;

/**
 * @author itw_yaobq
 * @date 2018/2/28 17:16
 */
@Slf4j
public class MyJpaTransactionManager extends JpaTransactionManager {

    @Override
    protected void doBegin(Object transaction, TransactionDefinition definition) {
        boolean readOnly = definition.isReadOnly();
        if (!readOnly) {
            DynamicDataSourceHolder.setDataSource(DataSourceType.write.getType());
        }
        if (log.isDebugEnabled()) {
            log.info("transaction-readOnly?: {}", readOnly);
        }
        super.doBegin(transaction, definition);
    }

    @Override
    protected void doCommit(DefaultTransactionStatus status) {
        String dataSource = DynamicDataSourceHolder.getDataSource();
        if (log.isDebugEnabled()) {
            log.info("dataSource: {}", dataSource);
        }
        super.doCommit(status);
    }
}
```

###### 配置动态数据源
**1 先创建一个读写库的标识**
```java
/**
 * @author itw_yaobq
 * @date 2018/4/1 13:44
 */
public enum DataSourceType {
    /**
     * 读写分离标识
     */
    read("read", "读库"), write("write", "写库");

    private String type;
    private String name;

    DataSourceType(String type, String name) {
        this.type = type;
        this.name = name;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
**2 创建动态数据源,并且设置数据源的切换策略**
```java
import com.taikang.operation.core.common.constant.DataSourceType;
import lombok.extern.slf4j.Slf4j;
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
import org.springframework.util.StringUtils;

/**
 * @author itw_yaobq
 * @date 2018/2/28 16:49
 */
@Slf4j
public class DynamicDataSource extends AbstractRoutingDataSource {

    /**
     * 切换数据源的名称标识
     */
    @Override
    protected Object determineCurrentLookupKey() {
        String dataSource = DynamicDataSourceHolder.getDataSource();
        if (StringUtils.isEmpty(dataSource)) {
            dataSource = DataSourceType.write.getType();
        }
        log.info("dataSource: " + dataSource);
		DynamicDataSourceHolder.clearDataSource();
        return dataSource;
    }
}
```
**3 创建动态数据源**
```java
import com.taikang.operation.core.common.constant.DataSourceType;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;

import javax.sql.DataSource;
import java.util.HashMap;

/**
 * 配置动态数据源,将所有的数据源都放入DynamicDataSource中
 * @author itw_yaobq
 * @date 2018/2/28 16:49
 */
@Configuration
public class DataSourceRouting {

    @Bean("routingDataSource")
    public AbstractRoutingDataSource routingDataSource(@Qualifier("readDataSource") DataSource readDataSource, @Qualifier("writeDataSource") DataSource writeDataSource) {
        DynamicDataSource dynamicDataSource = new DynamicDataSource();
        HashMap<Object, Object> map = new HashMap<>(2);
        map.put(DataSourceType.read.getType(), readDataSource);
        map.put(DataSourceType.write.getType(), writeDataSource);

        dynamicDataSource.setTargetDataSources(map);
        // 设置默认的数据源
        dynamicDataSource.setDefaultTargetDataSource(writeDataSource);
        return dynamicDataSource;
    }
}
```
**4 将读写库的标识存到ThreadLocal中**
```java
/**
 * 绑定当前线程和特定的数据源
 *
 * @author itw_yaobq
 * @date 2018/2/28 16:49
 */
public class DynamicDataSourceHolder {

    private static final ThreadLocal<String> DATASOURCE = new ThreadLocal<>();

    public static String getDataSource() {
        return DATASOURCE.get();
    }

    public static void setDataSource(String dataSourceName) {
        DATASOURCE.set(dataSourceName);
    }

    public static void clearDataSource() {
        DATASOURCE.remove();
    }
}
```
**5 在切面中根据方法名称判断要切换的数据源**
```java
import com.taikang.operation.core.common.constant.DataSourceType;
import com.taikang.operation.core.config.readwriteseparationconfig.dynamicdatasourceconfig.DynamicDataSourceHolder;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

/**
 * @author itw_yaobq
 * @date 2018/2/28 16:47
 */
@Aspect
@Component
@Slf4j
public class DynamicDataSourceAspect {

    @Around("execution(* com.taikang.operation.*.service.impl.*.select*(..)) || execution(* com.taikang.operation.*.service.impl.*.find*(..)) " +
            "|| execution(* com.taikang.operation.*.service.impl.*.get*(..)) || execution(* com.taikang.operation.*.service.impl.*.query*(..))")
    public Object setReadDataSourceType(ProceedingJoinPoint pjp) throws Throwable {
        MethodSignature signature = (MethodSignature) pjp.getSignature();
        Class declaringType = signature.getDeclaringType();
        boolean annotationPresent = declaringType.isAnnotationPresent(Transactional.class);
        if (!annotationPresent) {
            DynamicDataSourceHolder.setDataSource(DataSourceType.read.getType());
        }
        if (log.isDebugEnabled()) {
            log.info("数据源切面: {}", "read");
        }
        return pjp.proceed();
    }

    @Around("execution(* com.taikang.operation.*.service.impl.*.delete*(..)) || execution(* com.taikang.operation.*.service.impl.*.update*(..)) " + "|| execution(* com.taikang.operation.*.service.impl.*.insert*(..))")
    public Object setWriteDataSourceType(ProceedingJoinPoint pjp) throws Throwable {
        //        MethodSignature signature = (MethodSignature) pjp.getSignature();
        DynamicDataSourceHolder.setDataSource(DataSourceType.write.getType());
        if (log.isDebugEnabled()) {
            log.info("数据源切面: {}", "write");
        }
        return pjp.proceed();
    }
}
```
这里只是给出了两个数据源读写分离的配置,数据库的主从复制需要另外处理,除此之外,配置mybatis的动态数据源与此类似,还可以将代码改为支持多数据源配置的方法,只需要添加新的数据源, 至于具体选择哪个数据源,可以将根据方法名切换数据源的方式修改为根据自定义注解的方式切换数据源,在此不再赘述