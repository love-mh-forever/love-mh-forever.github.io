---
layout: post
title: springboot2.0 整合swagger2
category: springboot
tags: [springboot]
keywords: Spring Cloud,open source
---

### springboot2.0 整合swagger2
#### 依赖引入

``` xml
    <parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.6.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
    </parent>
    <!--swagger2-->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.8.0</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.8.0</version>
    </dependency>    
```

#### 配置类

``` java
@Configuration
@EnableSwagger2
public class Swagger2 {
    /**
     * 通过 createRestApi函数来构建一个DocketBean
     * 函数名,可以随意命名,喜欢什么命名就什么命名
     */
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())//调用apiInfo方法,创建一个ApiInfo实例,里面是展示在文档页面信息内容
                .select()
                //控制暴露出去的路径下的实例
                //如果某个接口不想暴露,可以使用以下注解
                //@ApiIgnore 这样,该接口就不会暴露在 swagger2 的页面下
                .apis(RequestHandlerSelectors.basePackage("cc.huluwa.electronic.contract.sign.group.controller"))
                .paths(PathSelectors.any())
                .build();
    }
    //构建 api文档的详细信息函数
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("Spring Boot Swagger2 构建RESTful API")
                //条款地址
                .termsOfServiceUrl("http://hwangfantasy.github.io/")
                .contact("zwd")
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
```

`@EnableSwagger2` 此注解有时引入失败，请删除`m2/repository/io/springfox/springfox-swagger2`中对应的版本包，我这里是2.8.0

#### 控制层使用
``` java
    @ApiOperation(value = "测试post请求",notes="注意事项",httpMethod = "POST")
    @ApiImplicitParam(dataType = "Department4Info",name = "department4Info",value = "添加部门",required = true)
    @RequestMapping(value = "/adddepartment")
    public String add_department(@RequestBody Department4Info department4Info) {

         RespInfo respInfo = ZwdUtil.ParamIsNull(department4Info,"companyid,userid");
         if (respInfo.getStatus()== InfoCode.ERROR) {
             return JSON.toJSONString(respInfo);
         }
         respInfo = department4Service.adddepartment(department4Info);
        return JSON.toJSONString(respInfo);
    }
```

`swagger是自动检测扫描包下面的所有方法，如果该方法没有使用@ApiIgnore，则表示此方法默认生成swagger`

* @ApiOperation中的`httpMethod`可以决定生成的是何类型的请求，默认是全部类型都生成
* @ApiImplicitParam 是对传递的参数进行描述和填充，可以多重使用
* @ApiIgnore 使用此注解表示该方法不对外暴露
* @JsonIgnore 是用来忽略在参数生成时，屏蔽掉我们不想生成的参数使用。用在实体类的参数上