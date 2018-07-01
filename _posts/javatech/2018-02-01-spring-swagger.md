---
layout:     post
title:      Spring下的文档自动生成工具--swagger
category: blog
description: swagger, using with spring mvc,spring boot, easily doc generation
---


### SpringMVC Swagger2.0文档生成集成

maven引入：
		<dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.5.0</version>
        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.5.0</version>
        </dependency>
            <dependency>
                <groupId>com.google.guava</groupId>
                <artifactId>guava</artifactId>
                <version>22.0</version>
            </dependency>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-annotations</artifactId>
                <version>2.9.3</version>
            </dependency>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.9.3</version>
            </dependency>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-core</artifactId>
                <version>2.9.3</version>
            </dependency>
配置UI资源webjar映射：
```
 <mvc:resources mapping="swagger-ui.html" location="classpath:/META-INF/resources/"/>
    <mvc:resources mapping="/webjars/**" location="classpath:/META-INF/resources/webjars/"/>
```

增加swaggerConfig配置类(bean)：

```java

@Configuration
@EnableSwagger2
@EnableWebMvc
public class SwaggerConfig {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build().apiInfo(apiInfo());
    }

    private ApiInfo apiInfo() {
        ApiInfo apiInfo = new ApiInfo("Rest API",
                "Rest API ",
                "1",
                "",
                new Contact("", "", ""),
                "Apache License",
                "");
        return apiInfo;
    }
}
```

默认生成的可访问：

1.json格式的接口描述：

http://yourhost:port/v2/api-docs 

2.可视化交互页面
http://yourhost:port/swagger-ui.html