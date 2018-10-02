
---
layout: post
title: Spring-cloud-feign实现服务消费
category: springcloud
no-post-nav: true
tags: [srpingcloud]
---

## Spring-cloud-feign实现服务消费
在微服务架构中，服务互相消费很平常的事情，下面简单的演示服务消费demo.
项目要使用一个`注册中心`二个`注册服务`

**注册中心eureka**
和平常的注册中心没有区别还是一样的配置
``` java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServserApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServserApplication.class, args);
	}
}

```
**服务提供者producer**

服务提供者也是正常的服务，向外提供服务接口，如下：
``` java
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String index(String name) {
        return "hello "+ name+", fegin is success";
    }
}
```

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

[项目地址](https://github.com/love-mh-forever/spring-cloud-examples/tree/master/spring-cloud-feign)