# 1 配置一个快速上手样例

## 1.1 配置eureka服务

pom spring cloud，Edgware.SR3

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
<version>1.5.13.RELEASE</version>
</parent>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Edgware.SR3</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
</dependencies>
</dependencyManagement>
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
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
</dependencies>
```

主入口

```java
@SpringBootApplicationn
@EnableEurekaServer
public class Server {

	public static void main(String[] args) {
	SpringApplication.run(EurekaServer.class, args);
	}	
}
```

yml文件

```yaml
server:
  port: 8761
eureka:
  client:
    registerWithEureka: false
    fetchRegistry: false
#上面那俩配置，就是说不要把自己注册到Eureka，因为他自己就是Eureka的服务，让别人来注册的；还有就是不要到Eureka服务抓取注册信息，因为他不需要，他自己就是Eureka服务
```

直接运行，就可以启动一个Eureka服务，对外开放的是8761端口

打开浏览器，输入http://localhost:8761/，就可以看到Eureka的控制台

## 1.2 配置service服务

pom 

​	去掉 dependency 的 <artifactId>spring-cloud-starter-eureka-server</artifactId>

主入口

​	EnableEurekaServer改为EnableEurekaClient

​	EnableEurekaClient就是说这是个Eureka客户端应用，需要向Eureka服务端注册自己为一个服务，启动这个就好

yml文件

```yaml
server:
  port: 8080
spring:
  application:
    name: ServiceA
eureka:
  instance:
    hostname: localhost
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka
      #上面的，配置了自己的服务名称，主机地址，还有eureka服务的地址
```

eureka基本原理图：

![image-20210628223217614](Eureka.assets/image-20210628223217614.png)

eureka样例基本原理图：

![image-20210628222946514](Eureka.assets/image-20210628222946514.png)

## 1.3 eureka和service服务改成集群

只需要改yml文件即可

eureka:yml

```yaml
server:
  port: 8762
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka/
      #在本地测试，需改host映射，然后改端口号及,hostname对应映射多次启动即可
      #hosts文件映射样例：127.0.0.1 peer1
```

​	这个就是自己对外开放8762端口，但是自己向8761端口的eureka注册中心来注册自己

​	说白了，就是启动两个eureka服务，互相注册，组成一个集群

service:yml

```yaml
server:
  port: 8088
spring:
  application:
    name: ServiceA
eureka:
  instance:
    hostname: localhost
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka,http://peer2:8762/eureka
      #defaultZone 这个也是说让服务调用者感知到eureka集群，可以从eureka集群获取注册信息
      #多次启动需要改端口号
```

集群启动成功后样例图：

![image-20210628221906557](Eureka.assets/image-20210628221906557.png)



![image-20210628221856486](Eureka.assets/image-20210628221856486.png)

## 1.4 负载均衡测试

ServiceB 接口

```java
@RestController
@Configuration
public class ServiceBController {

	/**
	 * RestTemplate，本来就是访问单个http接口的，但是现在加了@LoadBalanced以后，就可以通过Ribbon的支持，
	 * 实现负载均衡了，假如ServiceA部署了几台机器，那么可以自动负载均衡，轮询调用每一个实例
	 * @return
	 */
	@Bean
	@LoadBalanced
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}

	/**
	 * 服务B，就会去调用服务A的sayHello接口
	 * @param name
	 * @return
	 */
	@RequestMapping(value = "/greeting/{name}", method = RequestMethod.GET)
	public String greeting(@PathVariable("name") String name) {
		RestTemplate restTemplate = getRestTemplate();
		return restTemplate.getForObject("http://ServiceA/sayHello/" + name, String.class);
	}
	
}
```

一共访问了服务B的接口11次，会负载均衡到服务A的两个实例上去，一个实例是5次调用，一个实例是6次调用

![image-20210628222317591](Eureka.assets/image-20210628222317591.png)

![image-20210628222331530](Eureka.assets/image-20210628222331530.png)



集群环境eureka调用图：

![image-20210628223055871](Eureka.assets/image-20210628223055871.png)

## 1.5 eureka服务健康的自检机制

​	**1默认情况下:**

​	你的所有的服务，比如服务A和服务B，都会自动给eureka注册中心同步心跳，续约，每隔一段时间发送心跳，如果说某个服务实例挂了，那么注册中心一段时间内没有感知到那个服务的心跳，就会把那个服务给他下线

​	**2 如果你要自己来实现一个服务的健康检查的机制:**

​	自己来检查服务是否宕机，比如说，如果底层依赖的MQ、数据库挂了，你就宣布自己挂了，通知注册中心

在服务中加入以下依赖：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-actuator</artifactId>
	<version>1.5.13.RELEASE</version>
</dependency>
```

​	http://localhost:8080/health(服务地址前缀/health)，可以看到服务的健康状态

**3 可以自己实现一个健康检查器**

 	正常情况下就这样就可以了，但是有一个问题，就是可能这个服务依赖的其他基础设施，比如redis、mysql、mq，都挂掉了，或者底层的基础服务挂掉了，此时这个服务已经不可用了，那么这个服务就可以认定自己是不可用了

​	所以可以自己实现一个健康检查器，就是自己检查自己依赖的基础设施，或者是基础服务，是否挂掉了，来决定自己是否还是健康的

```java
@Component
public class ServiceAHealthIndicator implements HealthIndicator {

@Override
public Health health() {
	// 这里可以通过返回UP或者DOWN来指示服务的状态
	return new Health.Builder(Status.UP).build();
}

@Component
public class ServiceAHealthCheckHandler implements HealthCheckHandler {

@Autowired
private ServiceAHealthIndicator indicator;
    /**
     * eureka client里面会有一个定时器，不断调用那个HealthCheckHandler的getStatus()方法，然后检查当前这个服	   * 务实例的状态，
     * 如果状态变化了，就会通知eureka注册中心。如果服务实例挂掉了，那么eureka注册中心就会感知到，然后下线这个服      * 务实例。
     * @param currentStatus
     * @return
     */	
public InstanceStatus getStatus(InstanceStatus currentStatus) {
	Status status = indicator.health().getStatus();
	// 根据这个status，可以决定这里返回什么
	return InstanceStatus.UP;
}

}	
```

自己实现的情况较少，一般有个Health接口判断服务是否挂掉了就可以了。

## 1.6 心跳检测

```properties
eureka.instance.leaseRenewallIntervalInSeconds
#服务续约，eureka客户端，默认会每隔30秒发送一次心跳的eureka注册中心，这个参数可以修改这个心跳间隔时间
eureka.instance.leaseExpirationDurationInSeconds
#到期时间，如果在90秒内没收到一个eureka客户端的心跳，那么就摘除这个服务实例，别人就访问不到这个服务实例了，通过这个参数可以修改这个90秒的值

########
#但是一般这俩参数建议不要修改。
#另外这个心跳检测的机制其实叫做renew机制，看参数配置就知道了，也可以叫做服务续约
#如果一个服务被关闭了，那么会走cancel机制，就是类似是服务下线吧
```

​	如果90秒内没收到一个client的服务续约，也就是心跳吧，但是他这里叫做服务续约，那么就会走eviction，将服务实例从注册表里给摘除掉

## 1.7 注册表抓取

```properties

eureka.client.registryFetchIntervalSeconds
#默认情况下，客户端每隔30秒去服务器抓取最新的注册表，然后缓存在本地，通过该参数可以修改。

```

## 1.8 自定义元数据(少用)

```yaml
eureka:
	instance:
		hostname: localhosto
		metadata-map:
			company-name: zhss
```

可以通过上面的metadata-map定义服务的元数据，反正就是你自己需要的一些东西，不过一般挺少使用的

## 1.9 保护模式

如果在eureka控制台看到下面的东西：

#### <font color="red">**EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.**</font>

​	这就是eureka进入了自我保护模式，如果客户端的心跳失败了超过一定的比例，或者说在一定时间内（15分钟）接收到的服务续约低于85%，那么就会认为是自己网络故障了，导致人家client无法发送心跳。这个时候eureka注册中心会先给保护起来，不会立即把失效的服务实例摘除，在测试的时候一般都会关闭这个自我保护模式：

```
eureka.server.enable-self-preservation: false
#关闭保护模式
```

​	在生产环境里面，他怕自己因为自己有网络问题，导致别人没法给自己发心跳，就不想胡乱把别人给摘除，他就进入保护模式，不再摘除任何实例，等到自己网络环境恢复。

## 1.10 扩展

eureka server的启动，相当于是注册中心的启动

eureka client的启动，相当于是服务的启动

**eureka运行的核心的流程，eureka client往eureka server注册的过程**，

​	1 服务注册；服务发现，eureka client从eureka server获取注册表的过程；

​	2 服务心跳，eureka client定时往eureka server发送续约通知（心跳）；

​	3 服务实例摘除；通信，限流，自我保护，server集群

eureka server怎么启动的？

​	**eureka server是依赖eureka client的**，为啥呢？eureka server也是一个eureka client，因为后面我们讲到eureka server集群模式的时候，eureka server也要扮演eureka client的角色，往其他的eureka server上去注册。

​	**eureka core**，扮演了核心的注册中心的角色，接收别人的服务注册请求，提供服务发现的功能，保持心跳（续约请求），摘除故障服务实例。**eureka server依赖eureka core的，基于eureka core的功能对外暴露接口**，提供注册中心的功能

# 2 Eureka Server启动源码

## 2.1 Eureka Server的web工程结构分析以及web.xml解读

### 2.1.1 gradle主要引用

**jersey框架(国内基本没人用):**

​	1 eureka server依赖jersey框架，你可以认为jersey框架类比于spring web mvc框架，支持mvc模式，支持restful http请求

​	2 eureka client和eureka server之间进行通信，都是基于jersey框架实现http restful接口请求和调用的

​	3 eureka-client-jersey2，eureka-core-jersey2,这两个工程是eureka为了方便自己，对jersey框架的一个封装，提供更多的功能，方便自己使用

**mockito：**

​	1 mock测试框架，mock就是用的mockito框架

​	2 在eureka框架里面，他每个工程都是有src/test/java的，里面都写了针对自己本工程的单元测试

**jetty：**	

​	1 方便你测试的，测试的时候，是会基于jetty直接将eureka server作为一个web应用给跑起来

​	2 jersey对外暴露了一些restful接口，然后测试类里，就可以基于jersey的客户端，发送http请求，调用eureka server暴露的restful接口，测试比如说：服务注册、心跳、服务实例摘除，等等功能。

```groovy
apply plugin: 'war'

dependencies {
    compile project(':eureka-client')
    compile project(':eureka-core')
    runtime 'xerces:xercesImpl:2.4.0'
    compile "com.sun.jersey:jersey-server:$jerseyVersion"
    compile "com.sun.jersey:jersey-servlet:$jerseyVersion"
    compile 'org.slf4j:slf4j-log4j12:1.6.1'
    runtime 'org.codehaus.jettison:jettison:1.2' 
    providedCompile "javax.servlet:servlet-api:$servletVersion"

    testCompile project(':eureka-test-utils')
    testCompile "org.mockito:mockito-core:${mockitoVersion}"
    testCompile "org.eclipse.jetty:jetty-server:$jetty_version"
    testCompile "org.eclipse.jetty:jetty-webapp:$jetty_version"
}

task copyLibs(type: Copy) {
    into 'testlibs/WEB-INF/libs'
    from configurations.runtime	
}

//将eureka-resources下面的那些jsp、js、css给搞到这个war包里面去，然后就可以跑起来，提供一个index页面
war {
    from (project(':eureka-resources').file('build/resources/main'))
}

// Integration test loads eureka war, so it must be ready prior to running tests
test.dependsOn war

```



### 2.1.2 web.xml解读

web应用最最核心的就是web.xml。

​	最最重要的就是listener，listener是在web应用启动的时候就会执行的，负责对这个web应用进行初始化的事儿，我们如果自己写个web应用，也经常会写一个listener，在里面搞一堆初始化的代码，比如说，启动一些后台线程，加载个配置文件。

```xml
  <listener>
    <listener-class>com.netflix.eureka.EurekaBootStrap</listener-class>
  </listener>
<!--在eureka-core里，就是负责eureka-server的初始化的-->
```

有连着4个Filter，任何一个请求都会经过这些filter，这些filter会对每个请求都进行处理，这个4个filter都在eureka-core里面

```xml
  <filter>
    <filter-name>statusFilter</filter-name>
    <filter-class>com.netflix.eureka.StatusFilter</filter-class>
  </filter>
<!--StatusFilter：负责状态相关的处理逻辑-->
  <filter>
    <filter-name>requestAuthFilter</filter-name>
    <filter-class>com.netflix.eureka.ServerRequestAuthFilter</filter-class>
  </filter>
  <!--ServerRequestAuthFilter：一看就是，对请求进行授权认证的处理的-->
  <filter>
    <filter-name>rateLimitingFilter</filter-name>
    <filter-class>com.netflix.eureka.RateLimitingFilter</filter-class>
  </filter>
  <!--RateLimitingFilter：负责限流相关的逻辑的（很有可能成为eureka-server里面的一个技术亮点，看看人家eureka-server作为一个注册中心，是怎么做限流的，先留意算法是什么，留到后面去看）-->
  <filter>
    <filter-name>gzipEncodingEnforcingFilter</filter-name>
    <filter-class>com.netflix.eureka.GzipEncodingEnforcingFilter</filter-class>
  </filter>
  <!--GzipEncodingEnforcingFilter：gzip，压缩相关的；encoding，编码相关的-->
```

jersey框架的一个ServletContainer的一个filter

​	1 类似于每个mvc框架，比如说struts2和spring web mvc，都会搞一个自己的核心filter，或者是核心servlet，配置在web.xml里

​	2 相当于就是将web请求的处理入口交给框架了，框架会根据你的配置，自动帮你干很多事儿，最后调用你的一些处理逻辑。

​	3 jersey也是一样的，这里的这个ServletContainer就是一个核心filter，接收所有的请求，作为请求的入口，处理之后来调用你写的代码逻辑。

```xml
  <filter>
    <filter-name>jersey</filter-name>
    <filter-class>com.sun.jersey.spi.container.servlet.ServletContainer</filter-class>
    <init-param>
      <param-name>com.sun.jersey.config.property.WebPageContentRegex</param-name>
      <param-value>/(flex|images|js|css|jsp)/.*</param-value>
    </init-param>
    <init-param>
      <param-name>com.sun.jersey.config.property.packages</param-name>
      <param-value>com.sun.jersey;com.netflix</param-value>
    </init-param>

    <!-- GZIP content encoding/decoding -->
    <init-param>
      <param-name>com.sun.jersey.spi.container.ContainerRequestFilters</param-name>
      <param-value>com.sun.jersey.api.container.filter.GZIPContentEncodingFilter</param-value>
    </init-param>
    <init-param>
      <param-name>com.sun.jersey.spi.container.ContainerResponseFilters</param-name>
      <param-value>com.sun.jersey.api.container.filter.GZIPContentEncodingFilter</param-value>
    </init-param>
  </filter>
```

filter-mapping

```xml
  <filter-mapping>
    <filter-name>statusFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <filter-mapping>
    <filter-name>requestAuthFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
<!--StatusFilter和RequestAuthFilter，一看就是通用的处理逻辑，是对所有的请求都开放的-->
  <!-- Uncomment this to enable rate limiter filter.
  <filter-mapping>
    <filter-name>rateLimitingFilter</filter-name>
    <url-pattern>/v2/apps</url-pattern>
    <url-pattern>/v2/apps/*</url-pattern>
  </filter-mapping>
  -->
<!--RateLimitingFilter，默认是不开启的，如果你要打开eureka-server内置的限流功能，你需要自己把RateLimitingFilter的<filter-mapping>的注释打开，让这个filter生效-->

  <filter-mapping>
    <filter-name>gzipEncodingEnforcingFilter</filter-name>
    <url-pattern>/v2/apps</url-pattern>
    <url-pattern>/v2/apps/*</url-pattern>
  </filter-mapping>
<!-/v2/apps相关的请求，会走这里，仅仅对部分特殊的请求生效-->

  <filter-mapping>
    <filter-name>jersey</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  <!--jersey核心filter，是拦截所有的请求的-->

```

welcome-file-list，是配置了status.jsp是欢迎页面，首页，eureka-server的控制台页面，展示注册服务的信息

```xml
  <welcome-file-list>
    <welcome-file>jsp/status.jsp</welcome-file>
  </welcome-file-list>
```

### 2.1.3 总结

​	如果要启动eureka-server，就打成war包，找一个web容器，比如说tomcat，就可以启动了 ==> 测试类里，是基于jetty代码层面来启动jetty web容器和eureka-server，方便测试发送http restful接口的调用请求

​	1 eureka-server -> build.gradle中的依赖和构建的配置

​	2 eureka-server -> web应用 -> war包 -> tomcat就可以启动

​	3 web.xml -> listener -> 4个filter -> jersy filter -> filter mapping -> welcome file

##  2.2 Eureka Server启动之环境EurekaBootStrap监听器

​	web容器（tomcat还是jetty）启动的时候，把eureka-server作为一个web应用给带起来的时候，eureka-server的初始化的逻辑，监听器，EurekaBootStrap。（eureka-core里面）

**EurekaBootStrap部分核心代码：**

```java
public class EurekaBootStrap implements ServletContextListener {

    /**
     * Initializes Eureka, including syncing up with other Eureka peers and publishing the registry.
     * 监听器的执行初始化的方法，是contextInitialized()方法，这个方法就是整个eureka-server启动初始化的一个入
     * 口。
     */
 	@Override
    public void contextInitialized(ServletContextEvent event) {
        try {
            //2.2.1 initEurekaEnvironment(初始化eureka环境)
            initEurekaEnvironment();
            //2.2.2 配置文件加载
            initEurekaServerContext();

            ServletContext sc = event.getServletContext();
            sc.setAttribute(EurekaServerContext.class.getName(), serverContext);
        } catch (Throwable e) {
            logger.error("Cannot bootstrap eureka server :", e);
            throw new RuntimeException("Cannot bootstrap eureka server :", e);
        }
    }

}
```



### 2.2.1 initEurekaEnvironment(初始化eureka环境 )

#### 2.2.1.1 流程

**EurekaBootStrap部分核心代码：**

```java
package com.netflix.eureka;
public class EurekaBootStrap implements ServletContextListener {
    
    /** 
     * Users can override to initialize the environment themselves.
     * 初始化eureka-server的环境
     * ConfigurationManager.getConfigInstance()方法，
     * 这个方法，其实就是初始化ConfigurationManager的实例，也就是一个配置管理器的初始化的这么一个过程
     */
    protected void initEurekaEnvironment() throws Exception {
        logger.info("Setting the eureka configuration..");
        
        //初始化数据中心的配置
        String dataCenter = ConfigurationManager.getConfigInstance().getString(EUREKA_DATACENTER);
        if (dataCenter == null) {
            logger.info("Eureka data center value eureka.datacenter is not set, defaulting to default");
            //初始化数据中心的配置，如果没有配置的话，就是默认(DEFAULT)
            ConfigurationManager.getConfigInstance().setProperty(ARCHAIUS_DEPLOYMENT_DATACENTER, DEFAULT);
        } else {
            //初始化数据中心的配置,有的话就将配置设置一下
            ConfigurationManager.getConfigInstance().setProperty(ARCHAIUS_DEPLOYMENT_DATACENTER, dataCenter);
        }
        String environment = ConfigurationManager.getConfigInstance().getString(EUREKA_ENVIRONMENT);
        if (environment == null) {
            //初始化eurueka运行的环境，如果你没有配置的话，默认就给你设置为test环境
            ConfigurationManager.getConfigInstance().setProperty(ARCHAIUS_DEPLOYMENT_ENVIRONMENT, TEST);
            logger.info("Eureka environment value eureka.environment is not set, defaulting to test");
        }
    }
}
```

   ConfigurationManager是什么呢？看字面意思都猜的出来，配置管理器， 管理eureka自己的所有的配置，读取配置文件里的配置到内存里，供后续的eureka-server运行来使用。

**ConfigurationManager 部分核心代码（ConfigurationManager本身不是属于eureka的源码，是属于netflix config项目的源码）：**

```java
package com.netflix.config;


public class ConfigurationManager {    
    
    static volatile AbstractConfiguration instance = null;
    
    /**
    * double check + volatile 实现线程安全的单例模式
    */
	public static AbstractConfiguration getConfigInstance() {
        if (instance == null) {
            synchronized (ConfigurationManager.class) {
                if (instance == null) {
                    //调用重载有参方法
                    instance = getConfigInstance(Boolean.getBoolean(DynamicPropertyFactory.DISABLE_DEFAULT_CONFIG));
                }
            }
        }
        return instance;
    }
    
    /**
    * getConfigInstance()重载
    */
    private static AbstractConfiguration getConfigInstance(boolean defaultConfigDisabled) {
        if (instance == null && !defaultConfigDisabled) {
            //createDefaultConfigInstance 获取AbstractConfiguration实例 
            instance = createDefaultConfigInstance();
            registerConfigBean();
        }
        return instance;        
    }
    
    /**
    * ConcurrentCompositeConfiguration为AbstractConfiguration的实例，是AbstractConfiguration抽象类的实现
    */
    private static AbstractConfiguration createDefaultConfigInstance() {
        //无参构造调用了clear()方法进行一些配置的初始化
        ConcurrentCompositeConfiguration config = new ConcurrentCompositeConfiguration();  
        try {
            DynamicURLConfiguration defaultURLConfig = new DynamicURLConfiguration();
            config.addConfiguration(defaultURLConfig, URL_CONFIG_NAME);
        } catch (Throwable e) {
            logger.warn("Failed to create default dynamic configuration", e);
        }
        if (!Boolean.getBoolean(DISABLE_DEFAULT_SYS_CONFIG)) {
            SystemConfiguration sysConfig = new SystemConfiguration();
            config.addConfiguration(sysConfig, SYS_CONFIG_NAME);
        }
        if (!Boolean.getBoolean(DISABLE_DEFAULT_ENV_CONFIG)) {
            EnvironmentConfiguration envConfig = new EnvironmentConfiguration();
            config.addConfiguration(envConfig, ENV_CONFIG_NAME);
        }
        ConcurrentCompositeConfiguration appOverrideConfig = new ConcurrentCompositeConfiguration();
        config.addConfiguration(appOverrideConfig, APPLICATION_PROPERTIES);
        config.setContainerConfigurationIndex(config.getIndexOfConfiguration(appOverrideConfig));
        return config;
    }
}
```

​	**ConcurrentCompositeConfiguration实例，这个东西，其实就是代表了所谓的配置，包括了eureka需要的所有的配置。在初始化这个实例的时候，调用了坑爹的clear()方法，fireEvent()发布了一个事件（EVENT_CLEAR），fireEvent()这个方法其实是父类的方法，牵扯比较复杂的另外一个项目**

​	**ConcurrentCompositeConfiguration部分核心代码：**

```java
public class ConcurrentCompositeConfiguration extends ConcurrentMapConfiguration 
        implements AggregatedConfiguration, ConfigurationListener, Cloneable {
    
    /**
    * 有参构造调用clear()初始化配置
    */
    public ConcurrentCompositeConfiguration()
    {
        clear();
    }
    
    /**
    * 不属于eureka源码，无须追踪太多
    */
    @Override
    public final void clear()
    {	
        //父类的方法，加入事件到对应监听中(观察者模式)
        fireEvent(EVENT_CLEAR, null, null, true);
        configList.clear();
        namedConfigurations.clear();
        // recreate the in memory configuration
        containerConfiguration = new ConcurrentMapConfiguration();
        containerConfiguration.setThrowExceptionOnMissing(isThrowExceptionOnMissing());
        containerConfiguration.setListDelimiter(getListDelimiter());
        containerConfiguration.setDelimiterParsingDisabled(isDelimiterParsingDisabled());
        containerConfiguration.addConfigurationListener(eventPropagater);
        configList.add(containerConfiguration);
        
        overrideProperties = new ConcurrentMapConfiguration();
        overrideProperties.setThrowExceptionOnMissing(isThrowExceptionOnMissing());
        overrideProperties.setListDelimiter(getListDelimiter());
        overrideProperties.setDelimiterParsingDisabled(isDelimiterParsingDisabled());
        overrideProperties.addConfigurationListener(eventPropagater);
        //父类的方法，加入事件到对应监听中(观察者模式)
        fireEvent(EVENT_CLEAR, null, null, false);
        containerConfigurationChanged = false;
        invalidate();
    }

}
```

#### 2.2.1.2 总结

（1）创建一个ConcurrentCompositeConfiguration实例，这个东西，其实就是代表了所谓的配置，包括了eureka需要的所有的配置。在初始化这个实例的时候，调用了clear()方法来初始化配置，fireEvent()发布了一个事件（EVENT_CLEAR）（fireEvent()这个方法其实是父类的方法，牵扯比较复杂的另外一个项目（ConfigurationManager本身不是属于eureka的源码，是属于netflix config项目的源码）

（2）就是往上面的那个ConcurrentCompositeConfiguration实例加入了一堆别的config，然后搞完了以后，就直接返回了这个实例，就是作为所谓的那个配置的单例

（3）初始化数据中心的配置，如果没有配置的话，就是DEFAULT（默认配置），有配置则加入ConfigurationManager

（4）初始化eurueka运行的环境，如果你没有配置的话，默认就给你设置为test环境

（5）initEurekaEnvironment的初始化环境的逻辑，就结束了

### 2.2.2 initEurekaServerContext(配置文件加载)

```java
public class EurekaBootStrap implements ServletContextListener {
	/**
     * init hook for server context. Override for custom logic.
     */
    protected void initEurekaServerContext() throws Exception {
        //第一步，加载eureka-server.properties文件中的配置
        //在springCloud集成中就是对应yml的一些eureka配置
        EurekaServerConfig eurekaServerConfig = new DefaultEurekaServerConfig();

        // For backward compatibility
        JsonXStream.getInstance().registerConverter(new V1AwareInstanceInfoConverter(), XStream.PRIORITY_VERY_HIGH);
        XmlXStream.getInstance().registerConverter(new V1AwareInstanceInfoConverter(), XStream.PRIORITY_VERY_HIGH);

        logger.info("Initializing the eureka client...");
        logger.info(eurekaServerConfig.getJsonCodecName());
        ServerCodecs serverCodecs = new DefaultServerCodecs(eurekaServerConfig);

       
        ApplicationInfoManager applicationInfoManager = null;
	    //第二步，初始化eureka-server内部的一个eureka-client(用来跟其他的eureka-server节点进行注册和通信的)
        if (eurekaClient == null) {
            EurekaInstanceConfig instanceConfig = isCloud(ConfigurationManager.getDeploymentContext())
                    ? new CloudInstanceConfig()
                    : new MyDataCenterInstanceConfig();
            
            applicationInfoManager = new ApplicationInfoManager(
                    instanceConfig, new EurekaConfigBasedInstanceInfoProvider(instanceConfig).get());
            
            EurekaClientConfig eurekaClientConfig = new DefaultEurekaClientConfig();
            eurekaClient = new DiscoveryClient(applicationInfoManager, eurekaClientConfig);
        } else {
            applicationInfoManager = eurekaClient.getApplicationInfoManager();
        }

        //第三步，处理注册相关的事情
        PeerAwareInstanceRegistry registry;
        if (isAws(applicationInfoManager.getInfo())) {
            registry = new AwsInstanceRegistry(
                    eurekaServerConfig,
                    eurekaClient.getEurekaClientConfig(),
                    serverCodecs,
                    eurekaClient
            );
            awsBinder = new AwsBinderDelegate(eurekaServerConfig, eurekaClient.getEurekaClientConfig(), registry, applicationInfoManager);
            awsBinder.start();
        } else {
            registry = new PeerAwareInstanceRegistryImpl(
                    eurekaServerConfig,
                    eurekaClient.getEurekaClientConfig(),
                    serverCodecs,
                    eurekaClient
            );
        }

        //第四步，处理peer节点相关的事情
        PeerEurekaNodes peerEurekaNodes = getPeerEurekaNodes(
                registry,
                eurekaServerConfig,
                eurekaClient.getEurekaClientConfig(),
                serverCodecs,
                applicationInfoManager
        );

        //第五步，完成eureka上下文(context)的构建
        serverContext = new DefaultEurekaServerContext(
                eurekaServerConfig,
                serverCodecs,
                registry,
                peerEurekaNodes,
                applicationInfoManager
        );

        EurekaServerContextHolder.initialize(serverContext);

        serverContext.initialize();
        logger.info("Initialized server context");

        // Copy registry from neighboring eureka node
        //第六步，处理一点善后的事情，从相邻的eureka节点拷贝注册信息
        int registryCount = registry.syncUp();
        registry.openForTraffic(applicationInfoManager, registryCount);

        // Register all monitoring statistics.
        //第七步，处理一点善后的事情，处理所有的监控统计项
        EurekaMonitors.registerAllStats();
    }
}
```

#### 2.2.2.1 加载eureka-server.properties文件中的配置

##### 2.2.2.1.1 流程

EurekaServerConfig，这是个接口，这里面有一堆getXXX()的方法，包含了eureka server需要使用的所有的配置，都可以通过这个接口来获取

```java
@Singleton
public class DefaultEurekaServerConfig implements EurekaServerConfig {
    
    private static final DynamicStringProperty EUREKA_PROPS_FILE = DynamicPropertyFactory
            .getInstance().getStringProperty("eureka.server.props",
                    "eureka-server");
    
    /**
    * 可以认为DynamicPropertyFactory是从ConfigurationManager那儿来的，因为ConfigurationManager中都包含了加载出来的配置了，所以DynamicPropertyFactory里，也可以获取到所有的配置项
    */
    private static final DynamicPropertyFactory configInstance = com.netflix.config.DynamicPropertyFactory
            .getInstance();
    
    /**
    * 在DefaultEurekaServerConfig的各种获取配置项的方法中，配置项的名字是在各个方法中硬编码的
    * getEIPBindRebindRetries是n个get方法中的其中之一，写法都一样，都是硬编码获取值，没有则给个默认值
    */
    @Override
    public int getEIPBindRebindRetries() {
        //configInstance从一个DynamicPropertyFactory里面去获取的
        return configInstance.getIntProperty(
                namespace + "eipBindRebindRetries", 3).get();

    }
    
    public DefaultEurekaServerConfig() {
        init();
    }
    
    /**
    * DefaultEurekaServerConfig.init()方法中，会将eureka-server.properties文件中的配置加载出来，都放到ConfdigurationManager中去
    */
    private void init() {
        String env = ConfigurationManager.getConfigInstance().getString(
                EUREKA_ENVIRONMENT, TEST);
        ConfigurationManager.getConfigInstance().setProperty(
                ARCHAIUS_DEPLOYMENT_ENVIRONMENT, env);
        //eurekaPropsFile，对应的就是eureka-server(配置文件名)
        String eurekaPropsFile = EUREKA_PROPS_FILE.get();
        try {
            // ConfigurationManager
            // .loadPropertiesFromResources(eurekaPropsFile);
            //将加载出来的Properties中的配置项都放到ConfigurationManager中去，由这个ConfigurationManager来管理
            ConfigurationManager
                    .loadCascadedPropertiesFromResources(eurekaPropsFile);
        } catch (IOException e) {
            logger.warn(
                    "Cannot find the properties specified : {}. This may be okay if there are other environment "
                            + "specific properties or the configuration is installed with a different mechanism.",
                    eurekaPropsFile);
        }
    }
}
```

​	ConfigurationManager，是个单例，负责管理所有的配置的，ConfigurationManager是属于netfilx config开源项目的，不是属于eureka项目的源码，所以我们大概看一下就可以了，不要去深究了。eureka-server跟.properties给拼接起来了，拼接成一个eureka-server.properties，代表了eureka server的配置文件的名称。

​	比如说eureka-server那个工程里，就有一个src/main/resources/eureka-server.properties文件，只不过里面是空的，全部都用了默认的配置

```java
public class ConfigurationManager {
    
    /**
    * 将eureka-sesrver.properties中的配置，加载到了Properties对象中去；然后会加载eureka-server-环境.properties中的配置，加载到另外一个Properties中，覆盖之前那个老的Properties中的属性。
    */
    public static void loadCascadedPropertiesFromResources(String configName) throws IOException {
        //将eureka-sesrver.properties中的配置，加载到了Properties对象中去
        Properties props = loadCascadedProperties(configName);
        //将加载出来的Properties中的配置项都放到ConfigurationManager中去，由这个ConfigurationManager来管理
        if (instance instanceof AggregatedConfiguration) {
            ConcurrentMapConfiguration config = new ConcurrentMapConfiguration();
            config.loadProperties(props);
            //addConfiguration方式
            ((AggregatedConfiguration) instance).addConfiguration(config, configName);
        } else {
            //loadProperties方式
            ConfigurationUtils.loadProperties(props, instance);
        }
    }
}
```

##### 2.2.2.1.2 总结

（1）创建了一个DefaultEurekaServerConfig对象

（2）创建DefaultEurekaServerConfig对象的时候，在里面会有一个init方法

（3）先是将eureka-server.properties中的配置加载到了一个Properties对象中，然后将Properties对象中的配置放到ConfigurationManager中去，此时ConfigurationManager中去就有了所有的配置了

（4）然后DefaultEurekaServerConfig提供的获取配置项的各个方法，都是通过硬编码的配置项名称，从DynamicPropertyFactory中获取配置项的值，DynamicPropertyFactory是从ConfigurationManager那儿来的，所以也包含了所有配置项的值

（5）在获取配置项的时候，如果没有配置，那么就会有默认的值，全部属性都是有默认值的
