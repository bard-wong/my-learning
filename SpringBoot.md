# JSR303校验依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

在类上使用`@Validated`注解，在属性上使用`@Email`等校验数据

# 多环境配置

```
spring.profiles.active=dev
```

后面跟配置文件后缀，配置文件以application-XXX命名

# 集成swagger2

Maven依赖

```
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

SwaggerConfig

```java
@Configuration //配置类
@EnableOpenApi// 开启Swagger2的自动配置
public class SwaggerConfig {  
	@Bean
    public Docket docket(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .enable(true)
                .groupName("开发")
                .select()
                //RequestHandlerSelectors，配置要扫描接口的方式
                //basePackage：指定要扫描的包
                //any()：扫描全部
                //none()：不扫描
                //withClassAnnotation(GetMapping.class)：扫描类上的注解
                //withMethodAnnotation(RestController.class)：扫描方法上的注解
                .apis(RequestHandlerSelectors.basePackage("com.wang"))
                //过滤路径
                /*.paths(PathSelectors.ant("/wang/**"))*/
                .build();
    }
    
    /**配置Swagger信息=apiInfo*/
    private ApiInfo apiInfo() {
        Contact bard = new Contact("Bard", "wangwei_1013@outlook.com", "wangwei_1013@outlook.com");
        return new ApiInfo("Api文档",
                "Swagger2",
                "1.0", "urn:tos",
                bard,
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList());
    }
}
```

# 异步任务

`@EnableAsync` 开启异步注解功能，在方法上使用注解`@Async`开启异步

# 定时任务

`@EnableScheduling` 开启基于注解的定时任务

`@Scheduled(cron = "0 * * * * 0-7")`


cron表达式
//秒   分   时     日   月   周几
//0 * * * * MON-FRI
//注意cron表达式的用法；

