---
layout: post
title: zzzz
category: springcloud
tags: [springcloud]
keywords: Spring Cloud,open source
---

## 简介
这里springconfig是用来解决服务过多，每个服务都有自己的配置文件，查找和管理不是很方便。
`首先要理解config也是一个服务，其他服务通过config来间接的找到自己的配置文件`

## config服务的搭建
引入config-server所必须的依赖
``` xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-config-server</artifactId></dependency>
```
配置文件夹我放在了项目的根目录下，命名为`neo-config`
文件下是`{application}-{profile}`

## config-server是如何找到neo-config

### git方式(常用)
我们在config-server的配置文件加入如下配置:
``` yml
spring:
  application:
    name: spring-cloud-config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/love-dl-forever/spring-cloud-starter/    # 配置git仓库的地址
          search-paths: neo-config,*/application.properties                            # git仓库地址下的相对地址，可以配置多个，用,分割。
          username: love-mh-forever                                        # git仓库的账号
          password: *******  
```
* `uri`是`neo-config`所在项目的git地址
* `search-paths`是我们自定义的文件夹`neo-config`
### native方式
如上述代码,只有git方式就够了，因为springboot默认是使用git方式，如果要改变为native方式需要加入`spring.profiles`配置，具体如下：
``` yml
spring:
  application:
    name: spring-cloud-config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/love-dl-forever/spring-cloud-starter/    # 配置git仓库的地址
          search-paths: neo-config,*/application.properties                            # git仓库地址下的相对地址，可以配置多个，用,分割。
          username: love-dl-forever                                        # git仓库的账号
          password: a810095178                                     # git仓库的密码
          
        native:
          search-locations: /work/neo-config      #neo-config在本地的相对路径
  profiles: native
```
从代码可以看出多加了`native`配置,项目启动时就不会加载git所对应的属性
## config-client
现在把思想回到开始，config-server只是一个注册到注册中心的一个服务，它只是把所有配置文件都放到了一起，并告诉配置文件的路径，但并没告诉我们配置文件夹下的配置文件怎么对应具体的client。
下面看下client的配置文件，根据配置文件来解释会比较清晰明了。
``` yml

spring.application.name=spring-cloud-config-client
server.port=8002
spring.cloud.config.profile=dev
spring.cloud.config.label=master
spring.cloud.config.discovery.enabled=true
spring.cloud.config.discovery.serviceId=spring-cloud-config-server

eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
```
* 通过`serviceId`和config-server建立关联。
* `enabled`表示开启通过服务名来访问config-server。
* `label`表示从git上的哪个分支读取配置文件。
* `dev`指定读取的文件，会在配置文件的书写做详细说明。
## 配置文件的书写

其实在上面neo-config文件夹下的文件也提到过，这里做详细说明。
在项目开发时，我们不可能和`prod`用相同的`database`,`rabbitmq`等，所以一个服务可能会有多个配置文件。所以在配置文件的命令上就有了要求，规定按照`{application-name}-{profile}`来命名;`profile`就表示了不同环境使用的后缀不同，一般都是`prod`、`dev`、`test`这三个。所以上述的dev作用就在于此。
[项目地址](https://github.com/love-mh-forever/spring-cloud-examples/tree/master/spring-cloud-config-eureka)
