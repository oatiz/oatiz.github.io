---
title: skywalking5.0部署搭建
date: 2018-05-15 10:41:45
tags: 
  - apm
  - skywalking
---
## 介绍
[skywalking官网](http://skywalking.io/)

[skywalking-github](https://github.com/apache/incubator-skywalking)

## 背景
基于之前我用过zipkin与pinpoint,可以做下简单的对比(基于我自己的理解):

- 部署简易度(由低到高)
    - zipkin --> skywalking --> pinpoint
- 性能方面(由低到高)
    - pinpoint --> zipnkin --> skywaling 

相关文章:
[几种分布式调用链监控组件的实践与比较](https://juejin.im/post/5a274614518825592c07f8b8)

我们生产使用的spring cloud体系, eureka来做服务发现,feign进行通信,默认使用 hystrix做服务熔断.

结合上述相关介绍,我们最终采用skywalking

## 部署
### 下载

点击[下载地址](http://skywalking.io/downloads/),然后选择对应版本进行下载,解压

### 解压

下载后解压,文件目录可以分为三部分:

- agent: 需要监控的应用的代理
- collector: 应用trace信息收集器(被监控应用内数据会被收集起来)
- webapp: 前端页面展示

结构如下:

```shell
apache-skywalking-apm-incubating
|-- agent
|   |-- activations
|   |   |-- apm-toolkit-log4j-1.x-activation-5.0.0-beta-SNAPSHOT.jar
|   |   |-- apm-toolkit-log4j-2.x-activation-5.0.0-beta-SNAPSHOT.jar
|   |   |-- apm-toolkit-logback-1.x-activation-5.0.0-beta-SNAPSHOT.jar
|   |   |-- apm-toolkit-opentracing-activation-5.0.0-beta-SNAPSHOT.jar
|   |   `-- apm-toolkit-trace-activation-5.0.0-beta-SNAPSHOT.jar
|   |-- config
|   |   `-- agent.config
|   |-- logs
|   |-- optional-plugins
|   |   `-- apm-spring-annotation-plugin-5.0.0-beta-SNAPSHOT.jar
|   |-- plugins
|   |   |-- apm-dubbo-plugin-5.0.0-beta-SNAPSHOT.jar
|   |   |-- apm-elastic-job-2.x-plugin-5.0.0-beta-SNAPSHOT.jar
|   |   |-- ......
|   |   `-- tomcat-7.x-8.x-plugin-5.0.0-beta-SNAPSHOT.jar
|   `-- skywalking-agent.jar
|-- bin
|   |-- collectorService.bat
|   |-- collectorService.sh
|   |-- startup.bat
|   |-- startup.sh
|   |-- webappService.bat
|   `-- webappService.sh
|-- collector-libs
|   |-- agent-grpc-define-5.0.0-beta-SNAPSHOT.jar
|   |-- agent-grpc-provider-5.0.0-beta-SNAPSHOT.jar
|   |-- agent-jetty-define-5.0.0-beta-SNAPSHOT.jar
|   |-- ......
|   `-- zookeeper-3.4.10.jar
|-- config
|   |-- application.yml
|   |-- component-libraries.yml
|   `-- log4j2.xml
|-- DISCLAIMER
|-- LICENSE
|-- licenses
|   |-- LICENSE-annotations.txt
|   |-- LICENSE-antlr4-runtime.txt
|   |-- ......
|   |-- LICENSE-zuul.txt
|   `-- ui-licenses
|       |-- LICENSE-add-dom-event-listener
|       |-- LICENSE-antd
|       |-- ......
|       `-- LICENSE-whatwg-fetch
|-- logs
|   |-- collector.log
|   |-- skywalking-collector-server.log
|   |-- webapp-console.log
|   `-- webapp.log
|-- NOTICE
|-- README.txt
`-- webapp
    `-- skywalking-webapp.jar
```

### 后端部署

- 修改config/application.yml,将里面所有host改为物理机ip
- 修改bin/webappService.sh,将collector.ribbon.listOfServers=127.0.0.1:10800,改为config/application.yml里面naming.jetty.host与port
- 执行bin/startup.sh

需要注意的是,skywalking数据采用elasticsearch 存储.需要部署es,并且修改es的配置文件 elasticsearch.yml,将对应配置改为以下配置:

```yaml
network.host: 0.0.0.0
thread_pool.bulk.queue_size: 1000
```

### agent部署
- 复制skywalking-agent到任意目录
- 配置agent/config/agent.config文件
    - 将collector.servers填写为collector配置文件(config/application.yml)中naming/jetty/ip:port
- 添加参数
    - 在jvm启动参数添加:**-javaagent:/path/to/skywalking-agent/skywalking-agent.jar**

假设我们已经将skywalking-agent放到/opt/skywalking/目录下,那么我们的启动命令应该写为:

>java -javaagent:/opt/skywalking/agent/skywalking-agent.jar -Dskywalking.agent.application_code=gateway -jar gateway.jar

配置除了通过/config/agent.config文件外,可以通过环境变量和VM参数(-D)来进行设置

## 其他
因为现有版本5.0.0-alpha有bug,所以需要使用新发布的5.0.0-beta版,所以需要编译对应版本源码

- 拉取skywalking对应版本代码
    - git clone https://github.com/apache/incubator-skywalking.git
    - git checkout v5.0.0-beta
- 拉取子模块
    - git submodule init
    - git submodule update
- 修改npm源
    - 打开apm-webapp/pom.xml,搜索frontend-maven-plugin
    - 将install \-\-registry=https://registry.npmjs.org/ 替换为 install \-\-registry=https://registry.npm.taobao.org/
- 编译
    - mvn clean package -DskipTests

打包成功后对应的skywalking存放在apm-dist/target目录下,根据上面的部署步骤使用

## 注意事项

- ES版本问题:5.3.x有问题,请使用5.5.x或5.6.x
- agent中的collector.ip需要与collector 中配置文件中的相同(agent与collector在不同机器时,注意将ip改为对应机器的ip)
- 部署agent,collector机器的系统时区必须一致(不同时dashboard中查看不到数据)
- 编译时替换npm源

## 参考链接
[skywalking-中文文档](https://github.com/apache/incubator-skywalking/blob/master/docs/README_ZH.md)
