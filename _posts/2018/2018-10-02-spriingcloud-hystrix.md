
---
layout: post
title: Spring-cloud-hystrix实现熔断机制
category: springcloud
no-post-nav: true
tags: [srpingcloud]
---

## Spring-cloud-hystrix
在微服务架构中通常会有多个服务层调用，基础服务的故障可能会导致级联故障，进而造成整个系统不可用的情况，这种现象被称为服务雪崩效应。服务雪崩效应是一种因“服务提供者”的不可用导致“服务消费者”的不可用,并将不可用逐渐放大的过程。

由于服务熔断产生在于服务调用，所以可参考上边spring-cloud-feign进行代码更改

**服务消费者consumer**
* `pom.xml`加入服务调用依赖
``` xml
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-feign</artifactId>
		</dependency>
```
* 启动类
``` java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class HystrixApplication {

	public static void main(String[] args) {
		SpringApplication.run(HystrixApplication.class, args);
	}
}

```
增加了`@EnableFeignClients`注解
* 编写服务调用接口
``` java
@FeignClient(name = "server-producer")
public interface HelloRemote {
    @RequestMapping(value = "/hello")
    String hello(@RequestParam(value = "name") String name);
}

```
`@FeignClient`中的name是调用的服务名，`注意:`想要调用服务对外开发的接口，必须保持服务有同样的参数和请求地址，如上述`hello`，`name`，在`producer`中也要有这个接口和同样的请求参数。
* 启动服务，访问`hello`,可以看到`producer`中的hello被调用，并返回结果。
上述是和远程调用一样，没有做任何变动，下面我们进行简单的变动来实现失败反馈：
``` java
@FeignClient(name = "server-producer",fallback = HelloRemoteImpl.class)
public interface HelloRemote {
    @RequestMapping(value = "/hello")
    String hello(@RequestParam(value = "name") String name);
}
```
在`@FeignClient`增加了`fallback`属性，由fallback可以看出需要新建一个类继承`HolloRemote`.
``` java
@Component
public class HelloRemoteImpl implements HelloRemote {
    @Override
    public String hello(@RequestParam(value = "name") String name) {
        return "hello "+name+",this message send failed";
    }
}
```
* 当访问失败时，执行此方法
**application.properties的变动**
``` yml
server.port=8083
spring.application.name=server-consumer

eureka.client.service-url.defaultZone=http://localhost:8081/eureka/
feign.hystrix.enabled=true
```
增加了`feign.hystrix.enabled=true`

* 启动项目访问`hello`是能正常访问的，但当我们把`producer`服务停止运行时，再次访问，这些会调用fallback方法返回错误日志。

[项目地址](https://github.com/love-mh-forever/spring-cloud-examples/tree/master/spring-cloud-hystrix)