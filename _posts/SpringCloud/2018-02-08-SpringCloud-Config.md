---
layout: post
title: Spring Cloud Config配置中心
category: SpringCloud
tags: Spring Cloud
keywords: Spring,Cloud，config
---
### 1.简介
在分布式系统中，服务数量多，配置繁杂，而spring cloud config就为我们提供了这样一个分布式配置中心的服务。
它可以提供配置统一管理，客户端（config-client）不重启刷新配置，以及消息推送同时刷新多个客户端配置

### 2.准备工作

Spring Cloud Config包含config-server和config-client

* config-server: 配置中心，提供配置获取以及实时刷新配置。

* config-client: 客户端，获取配置。

* rabbitMq: 消息推送，要使用spring cloud bus需启动rabbitmq（安装方法请自行百度）。
### 3.Pom文件依赖添加
#### 3.1 config-server Pom引入config-server和bus消息总线模块

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```
#### 3.2 config-client Pom引入config和bus消息总线模块

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```
### 4.application文件配置
#### 4.1 config-server添加spring cloud config配置
application.yml添加以下配置

```
spring:
  application:
    name: config
    
  #使用本地配置文件（也可以使用git，svn等仓库）
  profiles:
     active: native
  cloud:
    config:
      enabled: true
      server:
        native:
          search-locations: classpath:/config  #配置文件存放路径
          
  #springcloud bus支持（也可以用其他的替代，这里使用rabbitmq）
  rabbitmq:
      host: localhost
      port: 5672
      username: guest
      password: guest
```
#### 4.2 config-client添加spring cloud config配置
resources目录下创建bootstrap.properties

```
#spring cloud config
spring.application.name=product-service
spring.cloud.config.label=master
spring.cloud.config.profile=dev
spring.cloud.config.uri=http://localhost:8888/
spring.cloud.config.username=user
spring.cloud.config.password=123456
spring.cloud.config.failFast=true

spring.cloud.bus.refresh.enabled=true
endpoints.bus.enabled=true
spring.cloud.bus.enabled=true
#rabbitMq
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```
#### 4.3 配置文件命名匹配
如4.1配置本地文件config目录下则需将文件存放在resources/config目录下，命名规则如下：

```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```
如有一个product-service微服务，环境是dev环境，则文件命名可以使用product-service-dev.properties
### 5.application注解添加
#### 5.1 config-server注解添加
添加`@EnableConfigServer`注解

```
@SpringBootApplication
@EnableConfigServer
@EnableDiscoveryClient
public class ConfigApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigApplication.class, args);
	}
}

```
#### 5.2 config-client使用配置刷新
添加`@RefreshScope`以支持实时刷新配置

```
@RefreshScope
@RestController
@RequestMapping("/client")
public class ConfigClientController {

	private static Logger logger = LoggerFactory.getLogger(ProductController.class);

	@Value("${members}")
	private String members;
	....
```

### 6.配置刷新
#### 6.1刷新单个微服务配置
Post请求http://ip:port/refresh
其中ip和port为此微服务的ip和端口
#### 6.2刷新所有或指定微服务配置（可以使用通配符）
Post请求http://ip:port/monitor?path=**
其中ip和port为config-server服务器的ip和端口，path可以放在body里面发送
#### 6.3后续将继续探索使用服务名刷新？？


