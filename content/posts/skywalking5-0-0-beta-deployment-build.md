+++
title = "skywalking5.0部署搭建"
date = 2018-05-15T16:06:00+08:00
tags = ["java", "apm", "skywalking"]
draft = false
author = "lyraile"
+++

## 介绍 {#介绍}

[skywalking官网](http://skywalking.io/) <br/>

[skywalking-github](https://github.com/apache/incubator-skywalking) <br/>


## 背景 {#背景}

相关文章： <br/>

-   [几种分布式调用链监控组件的实践与比较](https://juejin.im/post/5a274614518825592c07f8b8) <br/>

我们生产使用的spring cloud体系, eureka来做服务发现,feign进行通信,默认使用 hystrix做服务熔断. <br/>

结合上述相关介绍,我们最终采用skywalking <br/>


## 部署 {#部署}


### 下载 {#下载}

点击[下载地址](http://skywalking.io/downloads/),然后选择对应版本进行下载,解压 <br/>


### 解压 {#解压}

下载后解压,文件目录可以分为三部分: <br/>

-   agent: 需要监控的应用的代理 <br/>
-   collector: 应用trace信息收集器(被监控应用内数据会被收集起来) <br/>
-   webapp: 前端页面展示 <br/>

结构如下: <br/>

{{< highlight text >}}
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
{{< /highlight >}}


### 后端部署 {#后端部署}

-   修改config/application.yml,将里面所有host改为物理机ip <br/>
-   修改 **bin/webappService.sh** ,将 **collector.ribbon.listOfServers=127.0.0.1:10800** ,改为 **config/application.yml** 里面 **naming.jetty.host** 与 **port** <br/>
-   执行bin/startup.sh <br/>

需要注意的是, skywalking 数据采用 elasticsearch 存储.需要部署 es ,并且修改 es 的配置文件 **elasticsearch.yml** ,将对应配置改为以下配置: <br/>

{{< highlight text >}}
network.host: 0.0.0.0
thread_pool.bulk.queue_size: 1000
{{< /highlight >}}


### agent部署 {#agent部署}

-   复制skywalking-agent到任意目录 <br/>
-   配置agent/config/agent.config文件 <br/>
    -   将collector.servers填写为collector配置文件(config/application.yml)中naming/jetty/ip:port <br/>
-   添加参数 <br/>
    -   在jvm启动参数添加: `-javaagent:/path/to/skywalking-agent/skywalking-agent.jar` <br/>

假设我们已经将skywalking-agent放到/opt/skywalking/目录下,那么我们的启动命令应该写为: <br/>

> java -javaagent:/opt/skywalking/agent/skywalking-agent.jar -Dskywalking.agent.application_code=gateway -jar gateway.jar <br/>

配置除了通过/config/agent.config文件外,可以通过环境变量和VM参数(-D)来进行设置 <br/>


## 其他 {#其他}

因为现有版本5.0.0-alpha有bug,所以需要使用新发布的5.0.0-beta版,所以需要编译对应版本源码 <br/>

-   拉取skywalking对应版本代码 <br/>
    -   git clone <https://github.com/apache/incubator-skywalking.git> <br/>
    -   git checkout v5.0.0-beta <br/>
-   拉取子模块 <br/>
    -   git submodule init <br/>
    -   git submodule update <br/>
-   修改npm源 <br/>
    -   打开apm-webapp/pom.xml,搜索frontend-maven-plugin <br/>
    -   将install --registry=<https://registry.npmjs.org/> 替换为 install --registry=<https://registry.npm.taobao.org/> <br/>
-   编译 <br/>
    -   mvn clean package -DskipTests <br/>

打包成功后对应的skywalking存放在apm-dist/target目录下,根据上面的部署步骤使用 <br/>


## 注意事项 {#注意事项}

-   agent中的 collector.ip 需要与 collector 中配置文件中的相同 (agent 与 collector 在不同机器时,注意将 ip 改为对应机器的ip) <br/>
-   部署 agent,collector 机器的系统时区必须一致(不同时dashboard中查看不到数据) <br/>
-   编译时替换npm源 <br/>


## 参考链接 {#参考链接}

[skywalking-中文文档](https://github.com/apache/incubator-skywalking/blob/master/docs/README_ZH.md)

