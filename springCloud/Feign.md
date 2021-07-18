# 1 feign快速开始及样例

## 1.1 基于feign进行crud代码样例

1 首先基于单节点的eureka server,具体样例见eureka笔记

2 改造eureka ribbon结合的serverA serverB项目

首先给serverA加入crud代码样例

```
@RestController
public class ServiceAController implements ServiceAInterface {

	public String sayHello(@PathVariable("id") Long id, 
			@RequestParam("name") String name, 
			@RequestParam("age") Integer age) {     
		System.out.println("打招呼，id=" + id + ", name=" + name + ", age=" + age);   
		return "{'msg': 'hello, " + name + "'}";  
	}
	
	public String createUser(@RequestBody User user) {
		System.out.println("创建用户，" + user);  
		return "{'msg': 'success'}";
	}

	public String updateUser(@PathVariable("id") Long id, @RequestBody User user) {
		System.out.println("更新用户，" + user);  
		return "{'msg': 'success'}";
	}

	public String deleteUser(@PathVariable("id") Long id) {
		System.out.println("删除用户，id=" + id);
		return "{'msg': 'success'}"; 
	}

	public User getById(@PathVariable("id") Long id) {
		System.out.println("查询用户，id=" + id);
		return new User(1L, "张三", 20);
	}

}
```

再基于serverA的curd接口提供一个单独的service-a-api服务，加入私服/本地仓库

![image-20210718222509907](Feign.assets/image-20210718222509907.png)

serverA相关依赖

```xml
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
		    <artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<dependency>
		    <groupId>org.springframework.cloud</groupId>
		    <artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<dependency>
			<groupId>com.zhss.demo</groupId>
			<artifactId>service-a-api</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
	</dependencies>
```

server建立对应ServerClient接口继承service-a-api提供的api接口

```java
// 这个FeignClient里面的名字，就是你要调用的那个服务的名字
// 什么叫做声明式的方式，就是不用写代码，直接接口 +注解就搞定了，直接就可以访问别的服务
// 你用了feign之后，说实话，我提前给你剧透一下，spring cloud直接将feign和ribbon整合在一起了
// feign + ribbon + eureka都是整合在一起的
// feign发起请求的时候，都是会交给ribbon的LoadBalancer去进行负载均衡的
// 我们会感觉到，跟之前研究的ribbon整合eureka的负载均衡的原理还不是太一样，我们后面通过源码来看看
// feign + ribbon + eureka是如何整合在一起的

// 疑问来了，我希望的是说，ServiceB要调用ServiceA
// 我希望的是说，我连这个下面的一堆接口都不用自己定义，ServiceA直接提供给我这一票东东
// 这里其实是有重复性的劳动在里面的，其实每个接口的一堆定义和注解，都是由ServiceA搞定就可以了
// 不需要ServiceB重复的定义一遍

// 我只是说，我要访问ServiceA，但是人家ServiceA里面定义了哪些接口，哪些参数，都不用你管了
// 这里不用ServiceB把接口的定义重新写一遍了，直接用人家jar包里提供的就ok了
@FeignClient("ServiceA")
public interface ServiceAClient extends ServiceAInterface {
//下面的注释是不依赖 service-a-api包编写的额外代码
//没有service-a-api提供对应接口的话,serverB还需要针对serverA接口编写对应代码，比较麻烦
//通过service-a-api以jar包引入的方式，直接使用serverA的接口，是比较优雅的，正常企业级应用基本都是这样操作的

//	@RequestMapping(value = "/user/sayHello/{id}", method = RequestMethod.GET)
//	String sayHello(@PathVariable("id") Long id,
//			@RequestParam("name") String name,
//			@RequestParam("age") Integer age);
//
//	@RequestMapping(value = "/user/", method = RequestMethod.POST)
//	String createUser(@RequestBody User user);
//
//	@RequestMapping(value = "/user/{id}", method = RequestMethod.PUT)
//	String updateUser(@PathVariable("id") Long id, @RequestBody User user);
//
//	@RequestMapping(value = "/user/{id}", method = RequestMethod.DELETE)
//	String deleteUser(@PathVariable("id") Long id);
//
//	@RequestMapping(value = "/user/{id}", method = RequestMethod.GET)
//	User getById(@PathVariable("id") Long id);

}
```

以feign的方式在serverB对应controller调用serverA接口

```java
@RestController
@RequestMapping("/ServiceB/user")  
public class ServiceBController {

	/**
	 * 
	 * 这个东西，直接就是人家jar包里提供的接口和实现类都有了，不需要我们去关注他
	 * 我们就直接使用即可，直接引用一个IService，加上一个@autowired，就可以使用人家的服务了
	 * 
	 * 或者说，我们就在本地定义一个很薄很薄的接口，IServiceA -> 直接基于jar包里提供的一些东西简单配置一些注解就可以
	 * 
	 */
//	@Autowired
//	private IServiceA serviceA;
	
	@Autowired
	private ServiceAClient serviceA;
	
	@RequestMapping(value = "/sayHello/{id}", method = RequestMethod.GET)
	public String greeting(@PathVariable("id") Long id, 
			@RequestParam("name") String name, 
			@RequestParam("age") Integer age) {
//		return serviceA.sayHello(name);
		return serviceA.sayHello(id, name, age);
	}
	
	@RequestMapping(value = "/", method = RequestMethod.POST)
	public String createUser(@RequestBody User user) {
		return serviceA.createUser(user);
	}

	@RequestMapping(value = "/{id}", method = RequestMethod.PUT)
	public String updateUser(@PathVariable("id") Long id, @RequestBody User user) {
		return serviceA.updateUser(id, user); 
	}

	@RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
	public String deleteUser(@PathVariable("id") Long id) {
		return serviceA.deleteUser(id);
	}

	@RequestMapping(value = "/{id}", method = RequestMethod.GET)
	public User getById(@PathVariable("id") Long id) {
		return serviceA.getById(id);
	}
	
}
```

serverB相关依赖

```xml
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
		    <artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<dependency>
		    <groupId>org.springframework.cloud</groupId>
		    <artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<dependency>
		    <groupId>org.springframework.cloud</groupId>
		    <artifactId>spring-cloud-starter-ribbon</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-feign</artifactId>
		</dependency>
		<dependency>
			<groupId>com.zhss.demo</groupId>
			<artifactId>service-a-api</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
	</dependencies>
```

**总结：**微服务架构的话，谁提供什么接口，定义一个单独的xx-service-api的工程，部署到私服，人家依赖你的jar包，基于你提供的接口来开发，就可以了，直接调用你的服务的接口

feign默认就是集成了ribbon实现了负载均衡的

feign负载均衡调用：

![image-20210718223331301](Feign.assets/image-20210718223331301.png)

![image-20210718223337785](Feign.assets/image-20210718223337785.png)

**通过serverB多次调用serverA接口发现，不同请求的调用不是按照ribbon默认的轮询方式负载均衡的，同一接口调用才是安装ribbon默认轮询方式负载均衡的**

## 1.2 feign核心组件概述及spring cloud使用feign自定义bean

### 1.2.1  feign核心组件概述

feign就跟ribbon一样，内部都包含了很多的核心组件

ribbon，三剑客，ILoadBalancer、IRule、IPing，这几个东东是最重要的，ServerList，也属于他的核心组件，也是有Builder的

feign，也是一样的，也有很多核心的组件：

（1）编码器和解码器：Encoder和Decoder，如果调用接口的时候，传递的参数是个对象，feign需要将这个对象进行encode，编码，搞成json序列化，就是把一个对象转成json的格式，Encoder干的事儿；Decoder，解码器，就是说你收到了一个json以后，如何来处理这个东西呢？如何将一个json转换成本地的一个对象

（2）Logger：打印日志的，feign是负责接口调用，发送http请求，所以feign是可以打印这个接口调用请求的日志的

（3）Contract：比如一般来说feign的@FeignClient注解和spring web mvc支持的@PathVariable、@RequestMapping、@RequestParam等注解结合起来使用了。feign本来是没法支持spring web mvc的注解的，但是有一个Contract组件之后，契约组件，这个组件负责解释别人家的注解，让feign可以跟别人家的注解结合起来使用

（4）Feign.Builder：FeignClient的一个实例构造器，各种设计模式的使用，构造器，演示过，就是复杂对象的构造，非常适合，用了构造器模式之后，代码是很优雅的。

（5）FeignClient：就是我们使用feign最最核心的入口，就是要构造一个FeignClient，里面包含了一系列的组件，比如说Encoder、Decoder、Logger、Contract，等等吧

### 1.2.2  sprin cloud对feign的默认组件

Decoder：ResponseEntityDecoder

Encoder：SpringEncoder

Logger：Slf4jLogger

Contract（这个是用来翻译第三方注解的，比如说对feign使用spring mvc的注解）：SpringMvcContract

Feign实例构造器：HystrixFeign.Builder => Hystrix其实也是跟Feign整合在一起使用的，spring cloud几个核心的组件，eureka、ribbon、feign、hystrix、zuul，其实eureka是独立部署server的，但是feign + hystrix + ribbon + eureka client整合在一起使用的。我之前已经给大家说过ribbon和eureka是如何整合的

Feign客户端：LoadBalancerFeignClient => 负载均衡，通过LoadBalancerFeignClient，底层是跟ribbon整合起来使用的

### 1.2.3 自定义bean

自定义拦截器组件

```java
public class MyRequestInterceptor implements RequestInterceptor {

    @Override
    public void apply(RequestTemplate requestTemplate) {
        //在这里实现feign请求的拦截即可
        //在这里可以设置对请求的一些额外的信息
        //可以设置请求头，token,cookie等信息
        System.out.println("拦截请求测试111111111111111111111");
    }
}
```

自定义feign的配置

```java
//这里已经通过FeignClient注解去引用了，所以不用而外再去加@Configuration注解，去标明配置类
//正常来说@Bean注解实例化bean，需要和@Configuration配合
public class MyConfiguration {

	@Bean
	public RequestInterceptor requestInterceptor() {
	return new MyRequestInterceptor();
	}

}
```

引入feign自定义组件的配置

```java
@FeignClient(name = “ServiceA”, configuration = MyConfiguration.class)
public interface ServiceAClient {

}


```

feign的拦截器的使用，就是说我们可以实现对feign的请求进行拦截的拦截器，实现一个接口即可，RequestInterceptor，然后所有的请求发送之前都会被我们给拦截，你看这里不就是拦截器模式，interceptor模式

以自定义feign拦截器为样例，其他feign的组件也可以以此方式来自定义