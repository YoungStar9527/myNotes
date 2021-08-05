# 1 前言

## 1.1 hystrix是用来干嘛的

### 1.1.1 核心功能

隔离、限流、熔断、降级、运维监控

核心功能为 隔离、熔断、降级

核心功能如图所示：

![image-20210804073649054](Hystrix.assets/image-20210804073649054.png)

### 1.1.2、Hystrix是什么？

在分布式系统中，每个服务都可能会调用很多其他服务，被调用的那些服务就是依赖服务，有的时候某些依赖服务出现故障也是很正常的。

Hystrix可以让我们在分布式系统中对服务间的调用进行控制，加入一些调用延迟或者依赖故障的容错机制。

Hystrix通过将依赖服务进行资源隔离，进而组织某个依赖服务出现故障的时候，这种故障在整个系统所有的依赖服务调用中进行蔓延，同时Hystrix还提供故障时的fallback降级机制

总而言之，Hystrix通过这些方法帮助我们提升分布式系统的可用性和稳定性

### 1.1.3 hystrix 设计原则是什么？ 

（1）对依赖服务调用时出现的调用延迟和调用失败进行控制和容错保护
（2）在复杂的分布式系统中，阻止某一个依赖服务的故障在整个系统中蔓延，服务A->服务B->服务C，服务C故障了，服务B也故障了，服务A故障了，整套分布式系统全部故障，整体宕机
（3）提供fail-fast（快速失败）和快速恢复的支持
（4）提供fallback优雅降级的支持
（5）支持近实时的监控、报警以及运维操作

调用延迟+失败，提供容错
阻止故障蔓延
快速失败+快速恢复
降级
监控+报警+运维

完全描述了hystrix的功能，提供整个分布式系统的高可用的架构

### 1.1.4、Hystrix要解决的问题是什么？

解决复杂的分布式系统架构中，高可用的问题，避免服务被拖垮。

![商品服务接口导致缓存服务资源耗尽的问题](Hystrix.assets/商品服务接口导致缓存服务资源耗尽的问题.png)

### 1.1.5 扩展

小型电商网站的静态化方案

![小型电商网站的静态化方案](Hystrix.assets/小型电商网站的静态化方案.png)

大型电商网站的详情页系统的架构

![大型电商网站的详情页系统的架构](Hystrix.assets/大型电商网站的详情页系统的架构.png)

## 1.2 功能概述

### 1.2.1 基本功能

**资源隔离、限流、熔断、降级、运维监控**

**资源隔离：**让你的系统里，某一块东西，在故障的情况下，不会耗尽系统所有的资源，比如线程资源

我实际的项目中的一个case，有一块东西，是要用多线程做一些事情，小伙伴做项目的时候，没有太留神，资源隔离，那块代码，在遇到一些故障的情况下，每个线程在跑的时候，因为那个bug，直接就死循环了，导致那块东西启动了大量的线程，每个线程都死循环

最终导致我的系统资源耗尽，崩溃，不工作，不可用，废掉了

资源隔离，那一块代码，最多最多就是用掉10个线程，不能再多了，就废掉了，限定好的一些资源

**限流：**高并发的流量涌入进来，比如说突然间一秒钟100万QPS，废掉了，10万QPS进入系统，其他90万QPS被拒绝了

**熔断：**系统后端的一些依赖，出了一些故障，比如说mysql挂掉了，每次请求都是报错的，熔断了，后续的请求过来直接不接收了，拒绝访问，10分钟之后再尝试去看看mysql恢复没有

**降级：**mysql挂了，系统发现了，自动降级，从内存里存的少量数据中，去提取一些数据出来

**运维监控：**监控+报警+优化，各种异常的情况，有问题就及时报警，优化一些系统的配置和参数，或者代码

### 1.2.2 再看Hystrix的更加细节的设计原则是什么？

（1）阻止任何一个依赖服务耗尽所有的资源，比如tomcat中的所有线程资源
（2）避免请求排队和积压，采用限流和fail fast来控制故障
（3）提供fallback降级机制来应对故障
（4）使用资源隔离技术，比如bulkhead（舱壁隔离技术），swimlane（泳道技术），circuit breaker（短路技术），来限制任何一个依赖服务的故障的影响
（5）通过近实时的统计/监控/报警功能，来提高故障发现的速度
（6）通过近实时的属性和配置热修改功能，来提高故障处理和恢复的速度
（7）保护依赖服务调用的所有故障情况，而不仅仅只是网络故障情况

调用这个依赖服务的时候，client调用包有bug，阻塞，等等，依赖服务的各种各样的调用的故障，都可以处理

### 1.2.3 Hystrix是如何实现它的目标的？

（1）通过HystrixCommand或者HystrixObservableCommand来封装对外部依赖的访问请求，这个访问请求一般会运行在独立的线程中，资源隔离
（2）对于超出我们设定阈值的服务调用，直接进行超时，不允许其耗费过长时间阻塞住。这个超时时间默认是99.5%的访问时间，但是一般我们可以自己设置一下
（3）为每一个依赖服务维护一个独立的线程池，或者是semaphore，当线程池已满时，直接拒绝对这个服务的调用
（4）对依赖服务的调用的成功次数，失败次数，拒绝次数，超时次数，进行统计
（5）如果对一个依赖服务的调用失败次数超过了一定的阈值，自动进行熔断，在一定时间内对该服务的调用直接降级，一段时间后再自动尝试恢复
（6）当一个服务调用出现失败，被拒绝，超时，短路等异常情况时，自动调用fallback降级机制
（7）对属性和配置的修改提供近实时的支持

# 2 hystrix使用及原理

## 2.1 基于hystrix的线程池隔离技术进行商品服务接口的资源隔离

### 2.1.1 前言

​	**hystrix进行资源隔离，其实是提供了一个抽象，叫做command，就是说，你如果要把对某一个依赖服务的所有调用请求，全部隔离在同一份资源池内**

​	对这个依赖服务的所有调用请求，全部走这个资源池内的资源，不会去用其他的资源了，这个就叫做资源隔离

​	hystrix最最基本的资源隔离的技术，线程池隔离技术

​	对某一个依赖服务，商品服务，所有的调用请求，全部隔离到一个线程池内，对商品服务的每次调用请求都封装在一个command里面

​	每个command（每次服务调用请求）都是使用线程池内的一个线程去执行的

​	所以哪怕是对这个依赖服务，商品服务，现在同时发起的调用量已经到了1000了，但是线程池内就10个线程，最多就只会用这10个线程去执行

![资源隔离生效的讲解](Hystrix.assets/资源隔离生效的讲解.png)

### 2.1.2 示例

**最基本的利用HystrixCommand进行资源隔离样例**

Command

```java
public class CommandHelloWorld extends HystrixCommand<ProductInfo> {

    private Long productId;

    public CommandHelloWorld(Long productId) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.productId = productId;
    }

    @Override
    protected ProductInfo run() {
        // 拿到一个商品id
        // 调用商品服务的接口，获取商品id对应的商品的最新数据
        // 用HttpClient去调用商品服务的http接口
        String url = "http://127.0.0.1:8082/getProductInfo?productId=" + productId;
        String response = HttpClientUtils.sendGetRequest(url);
        System.out.println(response);
        return JSONObject.parseObject(response, ProductInfo.class);
    }

    public void setProductId(Long productId) {
        this.productId = productId;
    }
}
```

调用

```java
	/**
	 * 示例情况：nginx开始各级缓存都失效了。nginx发送很多请求直接到缓存服务拉取最原始的数据
	 * @param productId
	 * @return
	 */
	@RequestMapping("/getProductInfo")
	@ResponseBody
	public String getProductInfo(Long productId) throws ExecutionException, InterruptedException {
		// 拿到一个商品id
		// 调用商品服务的接口，获取商品id对应的商品的最新数据
		// 用HttpClient去调用商品服务的http接口	
		//1 不进行资源隔离
//		String url = "http://127.0.0.1:8082/getProductInfo?productId=" + productId;
//		String response = HttpClientUtils.sendGetRequest(url);
//		System.out.println(response);
		// 2 常用资源隔离的调用方式
//		HystrixCommand<ProductInfo> hystrixCommand = new CommandHelloWorld(productId);
//		ProductInfo execute = hystrixCommand.execute();
//		System.out.println(execute);
		// 3 异步资源隔离的调用方式
		Future<ProductInfo> queue = new CommandHelloWorld(productId).queue();
		System.out.println(queue.get());
		Thread.sleep(1000);
		System.out.println(queue.get());
		return "success";
	}
```

请求多次接口，多个结果资源隔离示例

ObservableCommand

```java
public class ObservableCommandHelloWorld extends HystrixObservableCommand<ProductInfo> {


    private Long[] productIds;

    public ObservableCommandHelloWorld(Long[] productIds) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.productIds = productIds;
    }

    /**
     * 多次调用接口的command
     * @return
     */
    @Override
    protected Observable<ProductInfo> construct() {
        return Observable.create(new Observable.OnSubscribe<ProductInfo>() {

            @Override
            public void call(Subscriber<? super ProductInfo> observer) {
                try {
                    //if (!observer.isUnsubscribed()) {
                    for (Long productId : productIds) {
                        String url = "http://127.0.0.1:8082/getProductInfo?productId=" + productId;
                        String response = HttpClientUtils.sendGetRequest(url);
                        ProductInfo productInfo = JSONObject.parseObject(response, ProductInfo.class);
                        observer.onNext(productInfo);

                    }
                        //observer.onNext("Hi " + name + "!");
                        observer.onCompleted();
                    //}
                } catch (Exception e) {
                    observer.onError(e);
                }
            }
        } ).subscribeOn(Schedulers.io());
    }
}
```

调用

```java
	/**
	 * 请求多次接口
	 * @param productIds
	 * @return
	 */
	@RequestMapping("/getProductInfoAll")
	@ResponseBody
	public String getProductInfoAll(String productIds) throws ExecutionException, InterruptedException {
		Long[] collect = Arrays.asList(productIds.split(",")).stream().map(a -> Long.parseLong(a)).toArray(Long[]::new);
		//1 常用资源隔离调用方式
//		HystrixObservableCommand<ProductInfo> getProductInfoCommand = new ObservableCommandHelloWorld(collect);
//		Observable<ProductInfo> observe = getProductInfoCommand.observe();
//		observe.subscribe(new Observer<ProductInfo>() {
//			@Override
//			public void onCompleted() {
//				System.out.println("获取完了所有数据");
//			}
//
//			@Override
//			public void onError(Throwable e) {
//				e.printStackTrace();
//			}
//
//			//每条返回数据的回调方法
//			@Override
//			public void onNext(ProductInfo productInfo) {
//				System.out.println(productInfo);
//			}
//		});

		//2 该使用方式只能一次onNext回调，多次则报错
//		ProductInfo productInfo = new ObservableCommandHelloWorld(collect).toObservable().toBlocking().toFuture().get();
//		System.out.println(productInfo);
		//3 异步不常用资源隔离调用方式
		Future<ProductInfo> productInfoFuture = new ObservableCommandHelloWorld(collect).toObservable().toBlocking().toFuture();
		System.out.println(productInfoFuture.get());
		Thread.sleep(1000);
		System.out.println(productInfoFuture.get());
		//toObservable延迟执行 observe立即执行
		//延迟执行时指调用下一个subscribe方法时才执行
		return "success";
	}
```

**总结：**

HystrixCommand：是用来获取一条数据的
HystrixObservableCommand：是设计用来获取多个结果的

**command的四种调用方式**

**同步：**

1 new CommandHelloWorld("World").execute()，

2 new ObservableCommandHelloWorld("World").toBlocking().toFuture().get()

如果你认为observable command只会返回一条数据，那么可以调用上面的模式，去同步执行，返回一条数据

**异步：**

3 new CommandHelloWorld("World").queue()

4 new ObservableCommandHelloWorld("World").toBlocking().toFuture()

对command调用queue()，仅仅将command放入线程池的一个等待队列，就立即返回，拿到一个Future对象，后面可以做一些其他的事情，然后过一段时间对future调用get()方法获取数据

**立即调用/延迟调用**

 observe()：hot，已经执行过了
 toObservable(): cold，还没执行过（延迟执行时指调用下一个subscribe方法时才执行）

**好处：**

不让超出这个量的请求去执行了，保护说，不要因为某一个依赖服务的故障，导致耗尽了缓存服务中的所有的线程资源去执行

