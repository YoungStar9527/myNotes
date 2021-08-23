# 1 zuul基本使用及概念

## 1.1 为啥微服务架构里要搞个网关这个东西

前端工程师，就是写js、html、css之类的代码，专门放在一个工程里，浏览器全部都是请求前端工程的，前端工程也是单独部署在服务器上的，比如说用这个node.js技术。前端工程比如用node.js发起类似ajax的请求，来请求我们的后端工程的api接口，http请求。

前端工程一定会调用大量的，多达几十个，几百个后端的服务

前端工程师要熟悉和维护几十个，甚至几百个后端的服务，在真正的工程开发里，非常的不靠谱和不现实，每个服务部署了几台机器，什么地址，叫什么名字

**设计模式，阶段一，facade门面思想，对于一套复杂的类体系和接口，全部暴露一个统一的门面，门面给别人调用即可**，别人就不用去熟悉你几十个类和几百个接口，我要调用这个接口了，我还得去想想，我得找哪个类

**1、请求路由**

**屏蔽复杂的后台系统的大量的服务，然后让前端工程师调用的时候非常的简单**

**2、统一处理**

**把所有后台服务都需要做的一些通用的事情，挪到网关里面去处理**

**（1）统一安全认证**

**（2）统一限流**

**（3）统一降级**

**（4）统一异常处理**

**（5）统一请求统计**

**（6）统一超时处理**

![image-20210823201752268](Zuul.assets/image-20210823201752268.png)

## 1.2 zuul的demo单独使用

请求过来了，然后就是直接将请求转发给一个url地址

相当于是就单独使用了zuul

pom文件

```xml
    	<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-zuul</artifactId>
		</dependency>
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
		</dependency>
```

yml

```yaml
server:
  port: 9001
zuul:
  routes:
    demo:
      url: http://localhost:9090/
```

入口类

```java
@SpringBootApplication
@EnableZuulProxy
public class ZuulGatewayApplication {

	public static void main(String[] args) {
		SpringApplication.run(ZuulGatewayApplication.class, args);
	}

}
```

根据yml的配置请求:

比如：http://localhost:9001/demo/ServiceB/user/sayHello

那么他会转发给：http://localhost:9090/ServiceB/user/sayHello

**PS:说白了就是将demo后面的替换到对应配置url即可**

## 1.3 spring cloud环境下zuul的demo体使用

我们现在要将zuul和ribbon、feign、hystrix、eureka整合起来使用，是如何使用的呢？

pom

```xml
		<!--单独zuul使用-->
    	<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-zuul</artifactId>
		</dependency>
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
		</dependency>
		<!--结合spring-cloud其他组件-->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
```

yml

```yaml
server:
  port: 9001
spring:
  application:
    name: zuul-gateway
eureka:
  instance:
    hostname: localhost
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
zuul:
  routes:
    ServiceB:
      path: /demo/**
```

上面的配置，就是说，所有针对http://localhost:9001/demo/**的请求，都会转发给服务B的其中一个服务，

比如说转发给http://localhost:9090/ServiceB/user/sayHello

测试请求：http://localhost:9001/demo/ServiceB/user/sayHello

会转发给：http://localhost:9090/ServiceB/user/sayHello

## 1.4 zuul的核心工作原理(基于责任链模式的各种过滤器)

设计模式，其实在各种开源项目里，到处都是设计模式

**数字越小优先级越高**

```
pre过滤器
-3：ServletDetectionFilter
-2：Servlet30WrapperFilter
-1：FromBodyWrapperFilter
1：DebugFilter
5：PreDecorationFilter

routing过滤器
10：RibbonRoutingFilter
100：SimpleHostRoutingFilter
500：SendForwardFilter

post过滤器
1000：SendResponseFilter

#error过滤器
0：SendErrorFilter


```

![image-20210823202854872](Zuul.assets/image-20210823202854872.png)

## 1.5 zuul最主要的功能(各种请求路由规则的配置)

zuul：请求路由，他作为一个网关，接收到一个请求，然后将这个请求转发给其他的服务

**1、简单路由**

spring cloud在zuul的routing阶段，搞了几个过滤器，这几个过滤器会负责将请求转发到后面的服务里去，最基本的就是SimpleHostRoutingFilter，这个大概就是这么配置的：

```yaml
zuul:
  routes:
    demo:
      path: /ServiceB/**
      url: http://localhost:9090/ServiceB
```

zuul.host.maxTotalConnections：这是配置连接到目标主机的最大http连接数，是用来配置http连接池的，默认是200

zuul.host.maxPerRouteConnections：就是每个主机的初始连接数，默认是20

**2、跳转路由**

SendForwardFilter负责进行跳转路由

这个，就是在zuul-gateway工程里，搞一个controller，然后配置一下：

```yaml
zuul:
  routes:
    demo:
      path: /test/**
      url: forward: /gateway/sayHello
```

这个说白了就是自己跳转到自己网关工程里的一个接口

**3、ribbon路由**

这个就是说基于ribbon + eureka，来转发请求到某个服务，使用ribbon来实现负载均衡

```yaml
#serviceId不设置默认就是ServiceB同名
zuul:
  routes:
    ServiceB:
      path: /demo/**
      serviceId: ServiceB

#简化的写法
zuul:
  routes:
    ServiceB:
      path: /demo/**
      
```

**4、自定义路由规则**

```java
@Configuration
public class MyRouteRuleConfig {

    @Bean
    public PatternServiceRouteMapper patternServiceRouteMapper() {
        return new PatternServiceRouteMapper("(zuul)-(?<test>.+)-(service)”, “${test}/**");
    }

}
```

请求：test/**的路径，转发给zuul-test-service

**5、忽略路由**

```yaml
zuul:
  ignoredPatterns: /ServiceB/test
```

## 1.6 zuul的相关常见配置(请求头、路由映射、hystrix、ribbon、超时)

**1、请求头配置**

默认情况下，zuul有些敏感的请求头不会转发给后端的服务

比如说：Cookie、Set-Cookie、Authorization，也可以自己配置敏感请求头

```yaml
zuul:
  sensitiveHeaders: accept-language, cookiei
  routes:
    demo:
      sensitiveHeaders: cookie
```

**2、路由映射信息**

我们在zuul-gateway中引入actuator项目，然后在配置文件中，将management.security.enabled设置为false，就可以访问/routes地址，然后可以看到路由的映射信息

**3、hystrix配置**

与ribbon整合转发时，会使用RibbonRoutingilter，转发会使用hystrix包裹请求，如果请求失败，会执行fallback逻辑

yml

```yaml
zuul:
  routes:
    ServiceB:
      path: /ServiceB/**
```

配置类

```java
public class ServiceBFallbackProvider implements ZuulFallbackProvider {

    public String getRoute() {
    	return “ServiceB”;
    }

    public ClientHttpResponse fallbackResponse() {
    	return new ClientHttpResponse() {
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream(“fallback”.getBytes());
            }

            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.TEXT_PLAIN);
                return headers;
            }

            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            public int getRawStatusCode() throws IOException {
              return 200;
            }

            public String getStatusText() throws IOException {
               return “OK”;
            }

            public void close() {

            }
    	}
    }
}
```

注入配置类

```java
@Configuration
public class FallbackConfig {
    @Bean
    public ZuulFallbackProvider fallbackProvider() {
    	return new ServiceBFallbackProvider();
    }

}
```

上面的代码就定义了ServiceB的降级逻辑

**但是一般不会针对某个服务搞降级，最好是在getRoute()方法中，返回：*，这样子就是做一个全局的降级**

**4、ribbon客户端预加载**

默认情况下，第一次请求zuul才会初始化ribbon客户端，所以可以配置预加载

```yaml
zuul:
  ribbon:
    eager-load:
      enabled: true
```

**5、超时配置**

人zuul也是用的hystrix + ribbon那套东西，所以说，超时这里要考虑hystrix和ribbon的，而且hystrix的超时要考虑ribbon的重试次数和单次超时时间

hystrix的超时时间计算公式如下：

(ribbon.ConnectTimeout + ribbon.ReadTimeout) * (ribbon.MaxAutoRetries + 1) * (ribbon.MaxAutoRetriesNextServer + 1)

```
ribbon:
  ReadTimeout:100
  ConnectTimeout:500
  MaxAutoRetries:1
  MaxAutoRetriesNextServer:1
```

如果不配置ribbon的超时时间，默认的hystrix超时时间是4000ms

## 1.7 zuul的一些高级功能

### 1.7.1 过滤器优先级

**数字越小优先级越高**

```
pre过滤器

-3：ServletDetectionFilter
-2：Servlet30WrapperFilter
-1：FromBodyWrapperFilter
1：DebugFilter
5：PreDecorationFilter

routing过滤器

10：RibbonRoutingFilter
100：SimpleHostRoutingFilter
500：SendForwardFilter

post过滤器

1000：SendResponseFilter

error过滤器

0：SendErrorFilter
```

### 1.7.2 自定义过滤器

配置类

```java
public class MyFilter extends ZuulFilter {

    public boolean shouldFilter() {
 	   // 是否要执行过滤器
  	  return true;
    }

    publici Object run() {
  	  System.out.println(“执行过滤器”);
 	   return null;
    }

    // 在哪个阶段执行
    public String filterType() {
 	   return FilterConstants.ROUTE_TYPE;
    }

    // 这是过滤器的优先级
    public int filterOrder() {
    	return 1;
    }

}
```

加载配置类

```java
@Configuration
public class FilterConfig {

    @Bean
    public MyFilter myFilter() {
        return new MyFilter();
    }

}
```



### 1.7.3 动态加载过滤器

pom

```java
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy-all</artifactId>
    <version>2.4.12</version>
</dependency>
```

yml

```yaml
zuul:
  filter:
    root: “groovy/filters”
    refreshInterval: 5
```

在Application类里添加

```java
    @PostConstruct
    public void zuulInit() {
        FilterLoader.getInstance().setCompiler(new GroovyCompiler());
        String scriptRoot = System.getProperty(“zuul.filter.root”, “groovy/filters”);
        String refreshInterval = System.getProperty(“zuul.filter.refreshInterval”, “5”);
        if(scriptRoot.length() > 0) {
            scriptRoot = scriptRoot + File.separator;
        }
        try {
            FilterFileManager.setFilenameFilter(new GroovyFileFilter());
            FilterFileManager.init(Integer.parseInt(refreshInterval), scriptRoot + “pre“, scriptRoot + “route”, scriptRoot + “post”);
        } catch(Exception e) {
        	throw new RuntimeException(e);
        }

    }
```

在src/main/java/groovy/filters中，放一个MyFilter.groovy

```java
class MyFilter extends ZuulFilter {

    public boolean shouldFilter() {
        return true;
    }

    public Object run() {
        System.out.println(“过滤器”);
        return null;
    }

    public String filterType() {
        return FilterConstants.ROUTE_TYPE;
    }

    public int filterOrder() {
        return 1;
    }

}
```

先启动网关项目，然后将这个过滤器放到指定目录，过几秒钟就会生效

### 1.7.4 禁用过滤器

```yaml
zuul:
  SendForwardFilter:
    route:
      disable: true
```

### 1.7.5 RequestContext

在过滤器中，使用RequestContext.getCurrentContext()，可以获取到serviceId、requestURI等各种东西

### 1.7.6 @EnableZuulServer与@EnableZuulProxy

@EnableZuulProxy简单理解为@EnableZuulServer的增强版，当Zuul与Eureka、Ribbon等组件配合使用时，我们使用@EnableZuulProxy。 

@EnableZuulServer这个的话，就没有PreDecorationFilter、RibbonRoutingFilter、SimpleHostRoutingFilter等过滤器

参考：https://blog.csdn.net/hxpjava1/article/details/78334354

### 1.7.7 error过滤器

在自定义过滤器里搞一个异常抛出来，ZuulException

然后写一个MyErrorController，继承BasicErrorController，统一处理异常，打印一些信息，这就是统一异常处理

统一异常处理

统一认证

统一限流

统一降级

## 1.8 一图了解zuul在spring cloud环境下的的核心原理

spring cloud的一些组件，他的顺序，都是有原因的，eureka、ribbon、feign、hystrix、zuul（eureka、ribbon、hystrix）

**zuul的核心源码，核心就在于说接收到一个url请求之后，如何来解析这个请求，如何根据我们在application.yml配置文件中的配置，来将这个url请求转换为针对某个服务的一个url请求**

依赖ribbon进行负载均衡，依赖eureka进行服务发现，依赖hystrix进行熔断降级 

![image-20210823210141390](Zuul.assets/image-20210823210141390.png)

# 2 zuul源码



