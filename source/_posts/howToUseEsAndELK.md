---
title: elasticsearch和ELK的安装使用
date: 2018-09-01 14:41:38
tags: 
	- elasticSearch
	- logstash
	- kibana
	- ELK
categories: es
description: elasticsearch以及ELK的安装配置及使用
---
#### es安装
- 下载es
    - 下载es,地址: <https://www.elastic.co/cn/downloads/elasticsearch>,可以下载tarball或者zip文件
    - es不能在root目录下运行,需要一个普通账户
    - 解压后进入bin目录,`./elasticsearch`,就可以启动es
- 安装ik分词器
    - 下载地址: <https://github.com/medcl/elasticsearch-analysis-ik/releases>
    - 可以使用plugin直接安装,但是需要安装对应的版本,我这里下载的是6.3.2: `./elasticsearch-plugin install https://oiw0skz2u.qnssl.com/o_1cl0qgokl4aq1gof9ii1qch1r5k7.zip?attname=elasticsearch-analysis-ik-6.3.2.zip`
    - 我这里无法直接下载安装,只能手动安装: 
        - 下载zip文件: `wget https://oiw0skz2u.qnssl.com/o_1cl0qgokl4aq1gof9ii1qch1r5k7.zip?attname=elasticsearch-analysis-ik-6.3.2.zip`
        - 解压文件,指定目录: `unzip -d ./ikanalyzer elasticsearch-analysis-ik-6.3.2.zip`
        - 进入目录,打包,需要maven环境: `mvn clean package`
        - 打包完毕后,找到`target/release/`目录下的zip文件,复制到es的plugin目录下
        - 解压这个文件,指定目录: `unzip -d ./ikanalyzer elasticsearch-analysis-ik-6.3.2.zip`
        - 删除zip文件
        - 再次启动es,ikAnalyzer会自动被加载

- es没有界面,可以使用head插件,head插件需要node环境
    - 安装node,去官网下载node
    - 解压
    - 配置环境变量: `vim /etc/profile`
    ```bash
    export NODE=/opt/node/
    export PATH=$NODE/bin:$PATH
    ```
    - 切换npm源: `npm config set registry https://registry.npm.taobao.org`
    - 下载head插件, 地址: <https://github.com/mobz/elasticsearch-head.git>
    - 使用yum安装git,使用git clone: `git clone https://github.com/mobz/elasticsearch-head.git`
    - 拉下代码后, 进入目录,运行`npm install`
    - 因为没有phantomJs,会报错,再执行`npm install phantomjs-prebuilt@2.1.16 --ignore-scripts`
    - 可以启动了: `npm run start`
    - 但是打开浏览器,还是无法连接es,需要在config/elasticsearch.yml中添加配置: 
    ```yml
    http.cors.enabled: true
    http.cors.allow-origin: "*"
    ```
- es集群配置
    - 需要修改`/config/elasticsearch.yml`
    ```YAML
    # 需要修改的配置
    # 集群名称
    cluster.name: es-cluster
    # 节点名称,每个节点要不一样
    node.name: node-1
    # 是否可以作为master
    node.master: true
    # 是否可以作为数据节点
    node.data: true
    # 本机的ip地址
    network.host: 192.168.198.88
    # 要监听的端口
    http.port: 9201
    # 集群的端口
    transport.tcp.port: 9301
    # 集群中的其他节点的host,因为都在一台机器上,集群端口不一样,这里配置两个实例的单机集群
    discovery.zen.ping.unicast.hosts: ["192.168.198.88:9301","192.168.198.88:9302"]
    # 最小集群节点数,为了避免集群脑裂, 集群中的节点数应该为 半数+1
    discovery.zen.minimum_master_nodes: 2
    ```
    - 依次启动各个节点即可组成集群

#### ELK的基本安装整合使用,以及和logback整合使用
##### 简介
- ELK是当前流行的日志分析系统,是由三个部分组成ElasticSearch, LogStash, Kibana组成, 三个部分都是elastic的开源产品,Logstash是一个用来收集分析过滤日志的工具,它可以从各种来源收集日志,输出到各种其他的组件之中,Kibana是一个基于web的图形化界面, 主要用于数据的可视化展示, ELK三者基本都做到了零配置,开箱即用
- 官网上有各个部分的使用文档: <https://www.elastic.co/cn/products>
- ELK三者的版本需要保持一致

##### 配置
###### logstash的配置:
- logstash依赖jdk1.8,使用之前需要配置好jdk环境
- logstash通过管道进行运作,管道有两个必须的元素,输入和输出,还有一个可选的元素,过滤器
- 输出部分从数据源获取数据,过滤器根据用户指定的数据格式修改数据,输出插件则将数据写入到目的地,如图:<img height="50" src="https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/basic_logstash_pipeline.png" alt="logstash"/>
- 一个简单的例子,启动logstash
    ```bash
    ./bin/logstash -e 'input { stdin {}} output { stdout {}}'
    ```
    启动后,键入的输入都会作为logstash的输入并且返回结果
    这里添加一个新的配置文件logstash-logback.conf
    ```conf
    input {
        tcp {
            host => "192.168.198.128"
            port => 9760
            mode => "server"
            tags => ["tags"]
            codec => json_lines
        }
    }
    
    output {
        stdout {
            codec => rubydebug
        }
        elasticsearch {
            hosts => "192.168.198.88:9200"
        }
    }
    ```
    上面的input内容会接收发送到`host:port`的内容,host是本机的ip,output中有两个输出,一个输出到标准控制台,另一个输出到es
- 带配置文件启动: `./bin/logstash -f ./config/logstash-logback.conf`

###### Kibana的配置
- 解压下载包后, 找到config目录下的kibana.yml
- 开启配置
```yml
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.url: "http://192.168.198.88:9200"
kibana.index: ".kibana"
```
- 直接启动bin目录下的kibana.bat,就可以直接使用了
- 访问kibana的页面: `localhost:5601`
- 由于logstash默认的索引是以`logstash-`开头, 后面为当前的日期,因此create index pattern 添加`logstash-*`

###### 项目中的日志配置
- pom文件中添加jar
```xml
<dependency>
	<groupId>net.logstash.logback</groupId>
	<artifactId>logstash-logback-encoder</artifactId>
	<version>4.11</version>
</dependency>
```
- 在日志配置的文件logback.xml中添加配置
```xml
<appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
    <destination>192.168.198.128:9760</destination>
    <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
</appender>

<root level="INFO">
    <appender-ref ref="LOGSTASH"/>
</root>
```
至此,开启ELK,可以在kibana的界面上看到项目的日志了,可以根据自己想看的内容添加filter,过滤日志内容

---
> 参考内容: <br/> https://www.cnblogs.com/yuhuLin/p/7018858.html <br> https://blog.csdn.net/y_y_y_k_k_k_k/article/details/72772223 <br/> https://www.cnblogs.com/moonlightL/p/7760512.html
