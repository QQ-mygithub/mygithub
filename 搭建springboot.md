# 搭建springboot

## 1.yml多环境配置, 键和值之间一定有空格

```yml
server:
  port: 8082
spring:
  profiles:
    active: dev2
---
server:
  port: 8085
spring:
  profiles: dev2

---
server:
  port: 8086
spring:
  profiles: prod
```

### 1.idea命令方式:

configuration>Program arguments>输入:--spring.profiles.active=prod

### 2.在Windows命令:

```Java
java -jar spring-boot-demo-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
```

## 2.webjars静态资源映射规则

引入jquery-webjars

<https://www.webjars.org/> 提供maven坐标

将坐标复制到pom文件里

启动springboot就可以访问

```Java
http://localhost:8086/webjars/jquery/3.4.1/jquery.js
```

## 3,springmvc相关配置都在WebMvcAutoConfiguration

1. 资源映射在这个方法

```Java
addResourceHandlers(ResourceHandlerRegistry registry)
```

1.所有webjars/**,webjars以jar的方式引入资源;访问资源都去这里找

```Java
"classpath:/META-INF/resources/webjars/"
```

配置缓存时间

```
@ConfigurationProperties(
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties
```

2.第二种映射规则

```
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
```

```
normalized[i] = location.endsWith("/") ? location : location + "/";
```

欢迎页设置

默认源文件下的所有index.html

```
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext) {
    WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(new TemplateAvailabilityProviders(applicationContext), applicationContext, this.getWelcomePage(), this.mvcProperties.getStaticPathPattern());
    welcomePageHandlerMapping.setInterceptors(this.getInterceptors());
    return welcomePageHandlerMapping;
}
```

```
private Optional<Resource> getWelcomePage() {
    String[] locations = WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations());
    return Arrays.stream(locations).map(this::getIndexHtml).filter(this::isReadable).findFirst();
}
```

设置图标

```
@Configuration
@ConditionalOnProperty(
    value = {"spring.mvc.favicon.enabled"},
    matchIfMissing = true
)
public static class FaviconConfiguration implements ResourceLoaderAware
```
图标设置:

所以的**/favicon.ico都是在静态资源文件下找;

设置静态资源路径:   值是数组形式

```yml
spring.resources.static-locations=classpath:/resource1/,classpath:/resource2/
```

## 4.模板引擎

1.