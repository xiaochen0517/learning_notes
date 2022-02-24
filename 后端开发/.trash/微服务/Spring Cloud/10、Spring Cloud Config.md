# 1、概述

## 1.1、简介

Spring Cloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。Spring Cloud Config分为服务端和客户端两部分。服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口。客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息。配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。

# 2、配置

## 2.1、服务端配置
### 2.1.1、代码配置

#### 2.1.1.1、`POM` 配置

```xml
<!-- spring-cloud-config-server -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

#### 2.1.1.2、启动类

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigStartClass3344 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigStartClass3344.class, args);
    }
}
```

#### 2.1.1.3、`YAML` 配置

```yaml
server:
  port: 3344
spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          # 使用用户名密码连接远程库
          uri: https://gitee.com/xxxxxx/xxxxxx.git
          username: xxxxxxx
          password: xxxxxxx
          default-label: main # 默认分支
          # 使用ssh连接远程库
#          uri: git@gitserver.com:team/repo1.git
#          ignoreLocalSshSettings: true
#          hostKey: someHostKey
#          hostKeyAlgorithm: ssh-rsa
#          privateKey: |
#            -----BEGIN RSA PRIVATE KEY-----
#            -----END RSA PRIVATE KEY-----
      label: main
eureka: # 将配置中心注册到eureka中心
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
```

### 2.1.2、访问测试

#### 2.1.2.1、访问格式

```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

#### 2.1.2.2、访问测试

![config服务端测试](E:\workspace\Notes&Files\markdown\后端开发\微服务\Spring Cloud\Photo\46、config服务端测试（10）.png)

## 2.2、客户端配置

### 2.2.1、代码配置

#### 2.2.1.1、`POM` 配置

```xml
<!-- spring-cloud-starter-config -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<!-- 检测远程仓库配置文件修改 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

#### 2.2.1.2、`YAML` 配置

新建 `bootstrap.yml` 文件

```yaml
server:
  port: 3355
spring:
  application:
    name: cloud-config-client
  cloud:
    # 客户端配置
    config:
      label: main # 分支名称
      name: config # 配置文件名称
      profile: dev # 配置文件后缀
      uri: http://localhost:3344 # 配置中心地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
# 设置监控端口
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

#### 2.2.1.3、主启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class ConfigClientStartClass3355 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientStartClass3355.class,args);
    }
}
```

#### 2.2.1.4、`Controller` 控制类

使用 `restful` 格式将配置文件的内容向外暴露

```java
@RestController
@RequestMapping("/client")
@RefreshScope
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("configInfo")
    public String getConfigInfo(){
        return configInfo;
    }

}
```

### 2.2.2、测试运行

#### 2.2.2.1、直接访问

![config客户端测试](E:\workspace\Notes&Files\markdown\后端开发\微服务\Spring Cloud\Photo\46、config客户端测试（10）.png)

#### 2.2.2.2、远程仓库和服务端刷新配置后刷新客户端

运行以下命令，对客户端此接口进行 `post` 请求。

```shell
curl -X POST "http://localhost:3355/actuator/refresh"
```

