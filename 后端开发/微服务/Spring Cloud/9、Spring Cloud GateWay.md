# Spring Cloud GateWay

## 1、概述

`GetWay` 是在 `Spring` 生态系统之上构建的 `api` 网关服务，基于 `Spring 5, Spring Boot 2` 和 `Project Reactor` 等技术。

`GetWay` 旨在提供一种简单而有效的方式来对 `api` 进行路由，以及提供一些强大的过滤器功能，例如：熔断、限流、重试等。

`Spring Cloud Gateway` 作为 `Spring Cloud` 生态系统中的网关，目标是替代 `Zuul` ，在 `Spring Cloud 2.0` 以上版本中，没有对新版本 `Zuul 2.0` 以及最新高性能版本进行集成，仍然还是使用 `Zuul 1.x` 非 `Reactor` 模式的老版本。而为了提升网关的性能， `Spring Cloud Gateway` 是基于 `WebFlux` 框架，而 `WebFlux` 框架底层使用了高性能的 `Reactor` 模式通信框架 `Netty` 。

`Spring Cloud Gateway` 的目标提供统一的路由方式且基于 `Filter` 链的方式提供了网关的基本功能，例如：安全，监控/指标和限流。

特性：

- 基于 `Spring Framework5, Project Reactor` 和 `Spring Boot 2.0` 进行构建
- 动态路由：能够匹配任何请求属性
- 可以对路由制定 `Predicate` （断言）和 `Filter` （过滤器）
- 集成 `Hystrix` 的断路器功能
- 集成 `Spring Cloud` 服务发现功能
- 易于编写 `Predicate` （断言）和 `Filter` （过滤器）
- 请求限流功能
- 支持路径重写

## 2、配置使用

### 2.1、基础配置

#### 2.1.1、导包

```xml
<!-- gateway -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

#### 2.1.2、启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class GatewayStartClass9527 {

    public static void main(String[] args) {
        SpringApplication.run(GatewayStartClass9527.class, args);
    }

}
```

### 2.2、`YAML` 配置

#### 2.2.1、`Eureka` 注册中心配置

```yaml
eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      # 是否将自己注册到 Eureka Server
      register-with-eureka: true
      # 是否从 Eureka Server 抓取已有的注册信息，默认为 true。
      fetch-registry: true
      defaultZone: http://localhost:7001/eureka, http://localhost:7002/eureka
```

#### 2.2.2、路由配置

```yaml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_routh # 路由的名称
#          uri: http://localhost:8001 # 直接添加代理的地址
          uri: lb://cloud-payment-service # 使用在注册中心的名称进行动态配置路由
          predicates:
            - Path=/payment/data,/payment/save # 进行代理的路径，可以用逗号分隔
        - id: guonei_routh
          uri: https://news.baidu.com
          predicates:
            - Path=/guonei
            - After=2021-08-06T16:36:17.061+08:00[Asia/Shanghai] # 指定时间之后此路由才会生效（after：之后；before：之前；between：之间，逗号分隔；）
            - Cookie=username, 1234 # 必须携带此cookie才可以允许请求
```

### 2.3、过滤器

```java
@Component
@Slf4j
public class LogGateWayFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.debug("=============全局过滤器=============");
        //             获取请求               获取参数           获取指定参数
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        // 限定参数内容
        if (StringUtils.isBlank(uname)){
            log.warn("=============用户名为空=============");
            // 返回指定状态下的内容
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        // 正常通过过滤器
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        /**
         * 过滤器优先级，数字越小优先级越高
         * int HIGHEST_PRECEDENCE = -2147483648;
         * int LOWEST_PRECEDENCE = 2147483647;
         */
        return 0;
    }
}
```

## 3、测试

![image-20210806170526353](E:\workspace\Notes&Files\markdown\后端开发\微服务\Spring Cloud\Photo\44、gateway测试图1（9）.png)

![image-20210806170645303](E:\workspace\Notes&Files\markdown\后端开发\微服务\Spring Cloud\Photo\45、gateway测试图2（9）.png)

