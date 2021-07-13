# 1 快速上手样例及基础知识

## 1.1 前言

**eureka:服务注册和发现**

1 分布式系统的环境里，特别拆分为了很多个服务，服务之间是互相调用的，可能一个请求过来，要多个服务互相调用，完成一个工作

2 就是说，服务A调用服务B，必须是得知道服务B部署在了哪些机器上，每台机器上接收请求的是哪个端口号

3 服务注册: 每个服务注册过去，eureka里就包含了每个服务部署在哪些机器上,每台机器上有哪个端口可以接受请求

4 服务发现呢: 你要调用一个服务，你就从eureka那边去拉取服务注册表，看看你要调用的那个服务部署在哪些机器上,监听的是哪个端口号

**Ribbon: 调用时的负载均衡**

1 spring cloud里面另外一个组件，ribbon也是netflix公司搞出来的

2 ribbon -> load balancer，负载均衡

3 ribbon会尽量确保说将所有的请求，均匀的分配到请求服务A的各台机器上去

**微服务!=分布式**

1 分布式不一定是微服务，可能是复杂系统的相互依赖，在不同的机器上，所以是分布式

2 微服务肯定是分布式，不同的微服务在分布式在不同的机器上，一个请求过来依赖于不同的微服务协作处理这个请求

## 1.2 基于netflix ribbon组件来一个原生的负载均衡调用demo

直接基于netflix的原生的ribbon API来做一把负载均衡的调用

```java
public class Application {

	public static void main(String[] args) throws Exception {
		// 基于ribbon的api来开发一个可以负载均衡调用一个服务的代码

		// 首先使用代码的方式对ribbon进行一下配置，配置一下ribbon要调用的那个服务的server list
		ConfigurationManager.getConfigInstance().setProperty(
				"greeting-service.ribbon.listOfServers", "localhost:8080,localhost:8088");

		// 获取指定服务的RestClient，用于请求某个服务的client
		RestClient restClient = (RestClient) ClientFactory.getNamedClient("greeting-service");

		// 你要请求哪个接口，构造一个对应的HttpRequest
		HttpRequest request = HttpRequest.newBuilder()
				.uri("/greeting/sayHello/leo")
				.build();

		// 模拟请求一个接口10次
		for(int i = 0; i < 10; i++) {
			HttpResponse response = restClient.executeWithLoadBalancer(request);
			String result = response.getEntity(String.class);
			System.out.println(result);
		}
	}

}
```

运行结果

![image-20210712204148083](Ribbon.assets/image-20210712204148083.png)

web服务

```java
@RestController
@RequestMapping("/greeting")
public class GreetingController {
	
	@GetMapping("/sayHello/{name}") 
	public String sayHello(@PathVariable("name") String name) {
		System.out.println("接收到了一次请求调用");   
		return "hello, " + name;
	}
	
}
```

web服务被调用结果：

![image-20210712204305925](Ribbon.assets/image-20210712204305925.png)

![image-20210712204315263](Ribbon.assets/image-20210712204315263.png)

​	实验结果是什么呢？ribbon很棒，就是帮我们完成一个服务部署多个实例的时候，负载均衡的活儿，没问题，可以干到。10次请求，均匀分布在了两个服务实例上，每个服务实例承载了5次请求。

## 1.3 netflix ribbon的负载均衡器的原生接口以及内置的规则(rule)

​	默认使用round robin轮询策略，直接从服务器列表里轮询

RestClient内部，底层，就是基于默认的BaseLoadBalancer来选择一个server

```java
    public static void ruleTest(){
        BaseLoadBalancer balancer = new BaseLoadBalancer();
        balancer.setRule(new MyRule(balancer));

        List<Server> servers = new ArrayList<Server>();
        servers.add(new Server("localhost", 8080));
        servers.add(new Server("lolcalhost", 8088));
        balancer.addServers(servers);

        for(int i = 0; i < 10; i++) {
            Server server = balancer.chooseServer(null);
            System.out.println(server);
        }
    }	
```

ILoadBalancer负载均衡器，底层是基于IRule，负载均衡算法，规则，来从一堆服务器list中选择一个server出来

​	负载均衡器是基于一个IRule接口指定的负载均衡规则，来从服务器列表里获取每次要请求的服务器的，所以可以自定义负载均衡规则

**自定义负载均衡规则**

```java
//**自定义负载均衡规则**
public class MyRule implements IRule {

    ILoadBalancer balancer;

    public MyRule() {

    }

    public MyRule(ILoadBalancer balancer) {
        this.balancer = balancer;
    }

    //重写choose方法，实现负载均衡算法
    public Server choose(Object key) {
        List<Server> servers = balancer.getAllServers();
        //样例，总是选择第一个
        return servers.get(0);
    }

    @Override
    public void setLoadBalancer(ILoadBalancer iLoadBalancer) {
        this.balancer = iLoadBalancer;
    }

    @Override
    public ILoadBalancer getLoadBalancer() {
        return balancer;
    }

}
```

调用结果(总是选择第一个)：

![image-20210712220733384](Ribbon.assets/image-20210712220733384.png)

**总结：**

1 较少需要自己定制负载均衡算法的，除非是类似hash分发的那种场景，可以自己写个自定义的Rule，比如说，每次都根据某个请求参数，分发到某台机器上去。不过在分布式系统中，尽量减少这种需要hash分发的情况。

2 说这个，主要是，负载均衡的一些底层API罢了，主要是ILoadBalancer和IRule

3 但是如果在后面要把这里做的比较复杂的话，很有可能会站在一些内置的Rule的基础之上，吸收他们的源码，自己定制一个复杂的高阶的涵盖很多功能的负载均衡器

## 1.4 ribbon原生API中用于定时ping服务器判断其是否存活的接口（ping）

负载均衡器里，就是ILoadBalancer里，有IRule负责负载均衡的规则，选择一个服务器；还有一个IPing负责定时ping每个服务器，判断其是否存活

```java
    public static void pingTest() throws InterruptedException {
        BaseLoadBalancer balancer = new BaseLoadBalancer();
        List<Server> servers = new ArrayList<Server>();
        servers.add(new Server("localhost", 8080));
        servers.add(new Server("localhost", 8088));
        balancer.addServers(servers);
        // http://localhost:8080/

        balancer.setPing(new MyPing());
        // 这里就会每隔1秒去请求那两个地址
        balancer.setPingInterval(1);
        Thread.sleep(5000);

        for(int i = 0; i < 10; i++) {
            Server server = balancer.chooseServer(null);
            System.out.println(server);
        }
    }
```

不过说实话，这块一般最好稍微做的那啥一点，用个类似/health的接口来表明自己的健康状况，可以自定义一个Ping组件

**自定义一个Ping组件**

```java
//**自定义一个Ping组件**
public class MyPing implements IPing {

    @Override
    public boolean isAlive(Server server) {
        System.out.println("ping:"+server.getHost()+":"+server.getHostPort());
        return true;
    }
}
```

调用结果(两个服务各每隔1秒ping一次)：

![image-20210712221121852](Ribbon.assets/image-20210712221121852.png)

**扩展：**

​	**ribbon比较重要的几个API，RestClient、ILoadBalancer、IRule、IPing**

## 1.5 spring cloud环境中的Ribbon使用

​	说白了，就是用一个RestTemplate来访问别的服务，RestTemplate本身很简单，就是一个http请求的组件，本身没什么负载均衡的功能，他就是指定一个url，就访问这个url就得了。但是这里用@LoadBalanced注解之后，默认底层就会用ribbon实现负载均衡了。

​	很简单的道理，这里肯定是RestTemplate底层会去基于ribbon来对一个服务的service list进行负载均衡式的访问。那service list是从哪儿拿到的？ribbon和eureka整合起来使用了，在这个ribbon里，肯定server list是从eureka client里拿到的，对吧，人家本地不是缓存了完整的注册表么？

 然后呢，请求一个服务的时候，就找那个服务对应的server list，round robin轮询一下

**eureka client服务调用的样例代码：**

```java
@LoadBalanced
@Bean
public RestTemplate getRestTemplate() {
	return new RestTemplate();
}
```

![image-20210713204134354](Ribbon.assets/image-20210713204134354.png)

## 1.6 ribbon流程图

​	RestTemplate，请求：http://localhost:8080/sayHello，如果给：http://ServiceA/sayHello，RestTemplate绝对100%报错，因为你里面搞了一个ServiceA，他是一个服务名称，不是ip地址和主机名，也没有端口号，根本没法请求这个东西

​	ILoadBalancer里面包含IRule和IPing，IRule负责从一堆server list中根据负载均衡的算法，选择出来某个server，关键是，server list从哪儿来？

![image-20210713204308541](Ribbon.assets/image-20210713204308541.png)
