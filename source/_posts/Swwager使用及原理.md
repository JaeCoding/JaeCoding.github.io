---
title: Swwager使用及原理 
date: 2018-08-01 17:15:35
tags: [API]
categories: 原理

---

# Swwager使用及原理 

整理一下，Swagger的使用和原理

# Spring Boot 整合Swagger

官方文档与下载
[推荐文档][1]


## 引入依赖
```Java
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
添加Swwager配置类
```Java
@Configuration
@EnableSwagger2
public class Swagger2Config extends WebMvcConfigurationSupport {

    /**
     * 因为 Swagger2的资源文件都是在jar包里面，如果不在此处配置加载静态资源，
     * 会导致请求http://localhost:8081/swagger-ui.html失败
     *  <!--swagger资源配置-->
     *  <mvc:resources location="classpath:/META-INF/resources/" mapping="swagger-ui.html"/>
     *  <mvc:resources location="classpath:/META-INF/resources/webjars/" mapping="/webjars/**"/>
     *
     * @param registry
     */
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");

        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.lx.soilparent.controller"))
                .paths(PathSelectors.any())
                .build()
                //不需要时，或者生产环境可以在此处关闭
                .enable(true);

    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Boot中使用Swagger2构建RESTful APIs")
                .description("描述：测试使用Swagger2！")
                //服务条款网址
                .termsOfServiceUrl("www.aslan007.github.io")
                .contact("Aslan")
                .version("1.0")
                .build();
    }
}
```

相关注释问题：
https://blog.csdn.net/qq_29534483/article/details/81199087#%E4%B8%89%E7%9B%B8%E5%85%B3%E6%B3%A8%E8%A7%A3

# 原理

springfox大致原理
springfox的大致原理就是，在项目启动的过种中，spring上下文在初始化的过程，框架自动跟据**配置加载一些swagger相关的bean到当前的上下文中**，并自动**扫描系统中可能需要生成api文档那些类**，并生成相应的**信息缓存起来**。如果项目MVC控制层用的是**springMvc那么会自动扫描所有Controller类**，跟据这些Controller类中的方法生成相应的api文档。

[官方文档][2]


  [1]: https://legacy.gitbook.com/book/huangwenchao/swagger/details
  [2]: https://swagger.io/specification/