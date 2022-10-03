---
title: springcloud服务注册-eureka的使用
date: 2020-12-02 15:43:31
cover: /images/eureka.jpg
tags: eureka
category: springcloud
---

本文主要讲解`eureka`的简单使用，由于`eureka`的停更所以目前的学习只作为基础的了解。其中`eureka`的配置主要涉及pom(依赖导入)、application.yaml(配置)、代码注解这三个方面，本文主要着重在这三个方面！

## Eureka服务端

### 服务端pom的引入内容

```xml
<!--引入最新的eureka的server端-->
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
<!--引入springboot的web启动程序-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--引入springboot的监控，一般与web一起使用，且其提供监控api用于第三方监控开发-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 服务端的application.yaml配置

```yaml
server:
  port: 7002


eureka:
  instance:
    #eureka服务端的实例名称
    hostname: eureka7002.com
  client:
    #false表示不向注册中心注册自己
    register-with-eureka: false
    #false表示自己端就是注册中心,职责是维护实例,并不需要检索服务
    fetch-registry: false
    service-url:
      #不是集群就按照下面的内容方式配置
      #设置与Eureka Server 交互的地址查询服务和注册服务都需要依赖这个地址
      #defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      defaultZone: http://eureka7001.com:7001/eureka/  #集群指向其他eureka
  server:
    # 关闭自我保护机制，保证不可用服务被及时剔除.默认开启保护机制 true
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```

### 服务端的启动代码

```java
//必须开启eureka的服务端注册，如果成功就会看到eureka启动成功
@SpringBootApplication
@EnableEurekaServer
public class EurekaMain7002 {
    public static void main(String[] args){
        SpringApplication.run(EurekaMain7002.class,args);
    }
}
```

## Eureka客户端-服务注册

### 客户端pom的引入内容

```xml
 <!--   引入eureka客户端     -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 客户端的application.yaml配置

```yaml
server:
  port: 8001
spring:
  application:
    #此处的name会作为注册到eureka服务的名字，调用的采用此处的名字
    name: cloud-payment-service
eureka:
  client:
    #是否将自己注册到注册中心, 默认true
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息,单机无所谓,集群必须设置为true配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      # defaultZone: http://localhost:7001/eureka  #单机版
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  #集群版
  instance:
    #此处如果指定了名字，那么eureka中的status处会显示此处的名字
    #否则会显示上面的cloud-payment-service，如果多个同样服务就分不清
    instance-id: payment8001
    #访问路径可以显示IP地址
    prefer-ip-address: true
    #Eureka客户端向服务端发送心跳的时间间隔.单位为秒(默认30 秒)
    lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳等待时间上限.单位为秒(默认90 秒),超时将剔除服务
    lease-expiration-duration-in-seconds: 2
```

### 客户端的启动代码与注册代码

```java
//主启动增加EnableEurekaClient
@SpringBootApplication
@EnableEurekaClient
public class PaymentMain8001 {

    public static void main(String[] args){
        SpringApplication.run(PaymentMain8001.class,args);
    }
}

```

后续所有的Controller中的调用接口都会被注册到服务中

> 注：根据本人在实际使用中发现注册端去除掉`@EnableEurekaClient`任然注册到eureka中。

## Eureka客户端-服务调用

### 客户端pom的引入内容

```xml
 <!--   引入eureka客户端     -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 客户端的application.yaml配置

```yaml
server:
  port: 80
spring:
  application:
    name: cloud-order-service

eureka:
  client:
    #是否将自己注册到注册中心, 默认true
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息,单机无所谓,集群必须设置为true配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      #      defaultZone: http://localhost:7001/eureka  #单机版
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka #集群版
```

### 客户端调用的代码

此处的代码调用没采用`ribbon`，只是简单的采用了`RestTemplate`

```java
 public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";

    private RestTemplate restTemplate = new RestTemplate();

    @PostMapping(value = "/consumer/payment/create")
    public CommonResult<Payment> create(@RequestBody Payment payment) {
        return restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
    }
```

## Eureka的一些特殊配置技巧

### 配置注册服务中的服务名字

```yaml
spring:
  application:
    #此处的name会作为注册到eureka服务的名字，调用的采用此处的名字
    name: cloud-payment-service
eureka:
    instance:
        #此处如果指定了名字，那么eureka中的status处会显示此处的名字
        #否则会显示上面的cloud-payment-service，如果多个同样服务就分不清
        instance-id: payment8001
        #访问路径可以显示IP地址
        prefer-ip-address: true
```

### 配置注册服务中服务端的模式

```yaml
 eureka:
     server:
        # 关闭自我保护机制，保证不可用服务被及时剔除.默认开启保护机制 true
        enable-self-preservation: false
        eviction-interval-timer-in-ms: 2000
```

其中模式涉及到CAP原理，如果设置为false为CP，如果设置为true为AP。CAP为C-数据一致性；A-服务可用性；P-服务对网络分区故障的容错性。

## 结语

上诉所有配置与引入只是为了对eureka有一个认识，后续将加上`ribbon`的使用，但是不会对其内部原理继续深入。

根据上诉内容eureka的yaml配置内容实质只有三个部分client、server与instance。

其中instance属于全局概念，client与server都可以被作为instance，因而无论在client还是server中都可以配置instance。

client与server作为一对互斥的概念。当作为server的时候,client的一些注册属性必须消除；当作为client的时候，server不能对其有任何配置。