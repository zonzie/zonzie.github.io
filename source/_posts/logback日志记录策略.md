---
title: logback日志记录策略
date: 2018-04-22 16:48:05
tags: logback
categories: log
description: logback日志框架,将不同级别的日志,以及特定包下的日志分开存储
---
###### 近期需要将日志文件分开存储,因为流量监控部分数据采集接口的日志文件过多,需要将其单独存放
- 以前的日志打印策略很简单,采用logback日志框架,LOG_FILE的值来自application.yml文件中的logging.file的值,具体配置如下:
	```xml
	<?xml version="1.0" encoding="UTF-8" ?>
	<configuration>
	
	    <!--<property name="LOG_FILE" value="/u02/tomcat/flowsystem/logs/fs"/>-->
	    
	    <!-- 控制台 -->
	    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
	        <encoder>
	            <pattern>%d{'yyyy-MM-dd HH:mm:ss,SSS'} %highlight(%5level) - %boldYellow([%-21thread]) %boldGreen(%-50logger{50}) : %m%n</pattern>
	        </encoder>
	    </appender>
	    <!-- 通用日志 appender  -->
	    <appender name="COMMON" class="ch.qos.logback.core.rolling.RollingFileAppender">
	    
	        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
	            <fileNamePattern>${LOG_FILE}-%d{yyyy-MM-dd}.log</fileNamePattern>
	            <!-- 日志最大的历史 90天 -->
	            <maxHistory>90</maxHistory>
	        </rollingPolicy>
	        <encoder>
	            <pattern>%d{'yyyy-MM-dd HH:mm:ss,SSS'} %5level - [%-21thread] %-50logger{50} : %m%n</pattern>
	        </encoder>
	    </appender>
	    <root level="INFO">
	        <appender-ref ref="STDOUT" />
	        <appender-ref ref="COMMON" />
	    </root>
	</configuration>
	```
- 这里我们将其改造一下,将数据采集接口的日志以及所有ERROR级别的文件单独存放
	```xml
	<?xml version="1.0" encoding="UTF-8" ?>
	<configuration>
	
	    <!-- 控制台 -->
	    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
	        <encoder>
	            <pattern>%d{'yyyy-MM-dd HH:mm:ss,SSS'} %highlight(%5level) - %boldYellow([%-21thread]) %boldGreen(%-50logger{50}) : %m%n</pattern>
	        </encoder>
	    </appender>
	
	    <!-- 通用日志 appender  -->
	    <appender name="COMMON" class="ch.qos.logback.core.rolling.RollingFileAppender">
	        <!--<file>${LOG_FILE}.log</file>-->
	        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
	            <fileNamePattern>${LOG_FILE}-%d{yyyy-MM-dd}.log</fileNamePattern>
	            <!-- 日志最大的历史 90天 -->
	            <maxHistory>90</maxHistory>
	        </rollingPolicy>
	        <encoder>
	            <pattern>%d{'yyyy-MM-dd HH:mm:ss,SSS'} %5level - [%-21thread] %-50logger{50} : %m%n</pattern>
	        </encoder>
	        <!--添加过滤器,ERROR级别的日志不会被记录-->
	        <filter class="ch.qos.logback.classic.filter.LevelFilter">
	            <level>ERROR</level>
	            <onMatch>DENY</onMatch>
	            <onMismatch>ACCEPT</onMismatch>
	        </filter>
	    </appender>
	
	    <!--数据采集接口的 appender-->
	    <appender name="COLLECT_APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
	        
	        <!--<file>${LOG_FILE}.log</file>-->
	        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
	            <fileNamePattern>${LOG_FILE}-COLLECT-%d{yyyy-MM-dd}.log</fileNamePattern>
	            <!-- 日志最大的历史 90天 -->
	            <maxHistory>90</maxHistory>
	        </rollingPolicy>
	        <encoder>
	            <pattern>%d{'yyyy-MM-dd HH:mm:ss,SSS'} %5level - [%-21thread] %-50logger{50} : %m%n</pattern>
	        </encoder>
	        <!--添加过滤器,ERROR级别的日志不会被记录-->
	        <filter class="ch.qos.logback.classic.filter.LevelFilter">
	            <level>ERROR</level>
	            <onMatch>DENY</onMatch>
	            <onMismatch>ACCEPT</onMismatch>
	        </filter>
	    </appender>
	
	    <!--ERROR级别的 appender-->
	    <appender name="ERROR_APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
	        <!--<file>${LOG_FILE}.log</file>-->
	        <!--基于时间的日志记录策略-->
	        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
	            <!--每天一个日志文件-->
	            <fileNamePattern>${LOG_FILE}-ERROR-%d{yyyy-MM-dd}.log</fileNamePattern>
	            <!-- 日志最大的历史 90天 -->
	            <maxHistory>90</maxHistory>
	        </rollingPolicy>
	        <encoder>
	            <pattern>%d{'yyyy-MM-dd HH:mm:ss,SSS'} %5level - [%-21thread] %-50logger{50} : %m%n</pattern>
	        </encoder>
	        <!--添加过滤器,只记录ERROR级别的日志-->
	        <filter class="ch.qos.logback.classic.filter.LevelFilter">
	            <level>ERROR</level>
	            <onMatch>ACCEPT</onMatch>
	            <onMismatch>DENY</onMismatch>
	        </filter>
	    </appender>
	
	    <!--对应数据采集的包下的日志文件配置,additivity设置为false,避免将root中的配置再记录一次-->
	    <logger name="com.taikang.flowsystem.datacollection" level="INFO" additivity="false">
	        <appender-ref ref="STDOUT" />
	        <appender-ref ref="COLLECT_APPENDER"/>
	        <appender-ref ref="ERROR_APPENDER"/>
	    </logger>
	
	    <!--root配置,所有的包遵循的日志打印方式-->
	    <root level="INFO">
	        <appender-ref ref="STDOUT" />
	        <appender-ref ref="COMMON" />
	        <appender-ref ref="ERROR_APPENDER"/>
	    </root>
	    
	</configuration>
	```
这样datacollection包下的日志和ERROR级别的日志就会分别存放在${LOG_FILE}-COLLECT-{yyyy-MM-dd}.log和${LOG_FILE}-ERROR-{yyyy-MM-dd}.log文件中了.