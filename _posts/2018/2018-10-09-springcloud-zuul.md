---
layout: post
tiltle: zuul的使用
no-post-nav:true
categroy: springcloud
tags: [springcloud]
---

## zuul的作用
  在微服务的架构中，由于服务过多，我们不可能多个端口去访问请求地址，这样的话会带来很繁琐的后续操作，如我们最常见的端口分布式问题。而使用网关既能统一请求端口，也能进行权限认证，还可以进行熔断处理。

* 转发请求
* 权限过滤
* 熔断处理
## 自定义网关
实现自定义Filter,需要继承`ZuulFilter`类。
``` java
public class MyFilter extends ZuulFilter {
    @Override
    String filterType() {
        return "pre"; //定义filter的类型，有pre、route、post、error四种
    }

    @Override
    int filterOrder() {
        return 10; //定义filter的顺序，数字越小表示顺序越高，越先执行
    }

    @Override
    boolean shouldFilter() {
        return true; //表示是否需要执行该filter，true表示执行，false表示不执行
    }

    @Override
    Object run() {
        return null; //filter需要执行的具体操作
    }
}
```
这里主要看`run`，因为后续操作都是走这个方法。

#### 转发作用

run方法中不需要添加任何代码，只需启动这个服务即可，如
`http://localhost:8888/spring-cloud-producer/hello?name=zwd`
网关会自动转发到spring-cloud-producer服务，并进行访问hello请求。当然一般不会直接使用这个单独功能。

#### 权限认证

其实就是对请求拦截后，进行一系列加密解密处理，如下验证token处理：
``` java
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();

        logger.info("--->>> TokenFilter {},{}", request.getMethod(), request.getRequestURL().toString());

        String token = request.getParameter("token");// 获取请求的参数

        if (StringUtils.isNotBlank(token)) {
            ctx.setSendZuulResponse(true); //对请求进行路由
            ctx.setResponseStatusCode(200);
            ctx.set("isSuccess", true);
            return null;
        } else {
            ctx.setSendZuulResponse(false); //不对其进行路由
            ctx.setResponseStatusCode(400);
            ctx.setResponseBody("token is empty");
            ctx.set("isSuccess", false);
            return null;
        }
    }
```
这里只是简单的判断请求是否携带token，如果没有就返回`token is empty`.
演示过程：启动eureka springcloud-producer zuul三个服务，访问`http://localhost:8888/spring-cloud-producer/hello?name=zwd`，返回信息为`token is empty`
`http://localhost:8888/spring-cloud-producer/hello?name=zwd&token=zzzzz`，返回信息为`hello zwd，this is two messge`
#### 熔断机制
当服务出现停止运行，或出现连接中断时，我们希望返回给前端一个错误信息，这时就可以使用`FallbackProvider`。

自定义一个类继承FallbackProvider
``` java
public class ProducerFallback implements FallbackProvider {
    private final Logger logger = LoggerFactory.getLogger(FallbackProvider.class);

    //指定要处理的 service。
    @Override
    public String getRoute() {
        return "spring-cloud-producer";
    }

    public ClientHttpResponse fallbackResponse() {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return 200;
            }

            @Override
            public String getStatusText() throws IOException {
                return "OK";
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("The service is unavailable.".getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }

    @Override
    public ClientHttpResponse fallbackResponse(Throwable cause) {
        if (cause != null && cause.getCause() != null) {
            String reason = cause.getCause().getMessage();
            logger.info("Excption {}",reason);
        }
        return fallbackResponse();
    }
```

* `getRoute()`是我们要处理的服务。
* `getBody`是返回的错误内容
演示过程：启动eureka springcloud-producer-2 zuul三个服务，访问`http://localhost:8888/spring-cloud-producer/hello?name=zwd&token=zzzzz`，这时返回给我定义好的错误信息。

[项目地址](https://github.com/love-mh-forever/spring-cloud-examples/tree/master/spring-cloud-zuul)
