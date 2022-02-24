# 1、SpringCloud Bus 消息总线

### 1.1、概述

Spring Cloud Bus 是用来将分布式系统的节点与轻量级消息系统链接起来的框架，它整合了Java的时间处理机制和消息中间件的功能。Spring Cloud Bus目前支持 RabbitMQ和Kafka

Spring Cloud Bus 能管理和传播分布式系统之间的消息，就像一个分布式执行器，可用于广播状态的更改、事件推送等，也可以作为微服务间的通信通道。

#### 1.1.1、什么是总线

在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个共用的消息主题，并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被实例监听和消费，所以称它为消息总线。在总线上的各个实例，都可以方便地广播一些需要让其他连接在该主题上的实例都知道的消息。

#### 1.1.2、基本原理

ConfigClient实例都监听MQ中同一个topic（默认是SpringCloudBus）。当一个服务刷新数据的时候，它会把这个信息放入到Topic中，这样其他监听同一个Topic的服务就能得到通知，并去更新自身的配置。

### 1.2、配置环境

#### 1.2.1、下载 Erlang

[官网下载地址](https://www.erlang.org/downloads) 

#### 1.2.2、下载 RabbitMQ

[Installing on Windows — RabbitMQ](https://www.rabbitmq.com/install-windows.html) 

启动完成，访问地址 `http://localhost:15672` ，即可查看 RabbitMQ 控制面板

默认用户名密码为： guest

#### 1.2.3、代码配置

- 配置maven

```xml
<!-- 消息总线支持 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```



- 配置yaml

```yaml
spring:
  # 配置rabbitmq地址
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
eureka: # 将配置中心注册到eureka中心
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
# 暴露刷新配置端点-此处配置在配置中心服务端需要-用户刷新git储存库中的配置文件
management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'
```



#### 1.2.4、全部刷新及部分刷新

- 整体刷新

```shell
curl -X POST "http://localhost:3344/actuator/bus-refresh"
```

- 部分刷新

```shell
curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"
```



