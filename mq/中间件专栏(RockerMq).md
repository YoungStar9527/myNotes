# 1 订单系统相关场景

## 1.1  一个真实电商订单系统的整体架构、业务流程及负载情况

​	**订单系统核心业务：**

![image-20211224145132976](中间件专栏(RockerMq).assets/image-20211224145132976.png)

​	订单系统最核心的一个环节就出现了，就是要根据APP端传递过来的种种信息，完成订单的创建，此时需要在数据库中创建对应的订单记录，整个过程，就像下面这个图一样

![image-20211224145411880](中间件专栏(RockerMq).assets/image-20211224145411880.png)

**订单系统的非核心业务流程**

![image-20211224145448682](中间件专栏(RockerMq).assets/image-20211224145448682.png)

**订单系统的真实生产负载情况**

压力主要在两方面：

- 一方面是订单系统日益增长的数据量
- 一方面是在大促活动时每秒上万的访问压力

## 1.2 概括一下你们系统的架构设计、业务流程以及负载情况

OA系统、CRM系统、财务系统或者其他任何看起来很普通的系统，也许总共就几十个人用。

那你能不能思考一下，假设你的这个系统是一个SaaS云平台，要提供给几万个公司的百万用户去使用呢？

如果是这样，那你的系统必然会有很多的技术挑战，你可以去预估一下，当达到那个数量级之后，你的系统会有多大的数据量？多大的访问量？然后再去思考在这么大的数据量和访问量之下，现有的系统会有哪些技术难题？

接着你就可以思考，应该学习一些什么样的技术来解决这些问题？

## 1.3 系统面临的现实问题：下订单的同时还要发券、发红包、Push推送，性能太差

**系统压力越来越大到底指的是什么意思**

![image-20211224145945574](中间件专栏(RockerMq).assets/image-20211224145945574.png)

​	用户的使用习惯直接决定了他们使用我们APP的频率、时间段和时长，一般每隔几天用一次我们的APP？每次使用一般在什么时间段？每次使用多长时间？

​	这些东西都要通过对用户的分析得出来

**根据线上统计数据推算出系统的负载**

​	根据线上系统的接口统计数据来看，晚上购物最活跃的时候，订单系统下单最顶点的高峰时段每秒会有超过2000的请求，这就是订单系统的最高负载。其他时候都比这个负载会低不少。

**为什么系统的压力会越来越大**

​	要明白什么是系统压力，就得明白你的系统线上部署的机器情况和使用的数据库的机器情况

​	而且作为一个合格的互联网行业的Java工程师，要对各种机器配置大致能抗下的并发量有一个基本的了解

​	4核8G的机器一般每秒钟抗几百请求都没问题，现在才每秒两三百请求，CPU资源使用率都不超过50%。

​	可以说8台4核8G的机器，每台机器每秒高峰期两三百请求是很轻松的。

​	然后数据库服务器因为用的是16核32G的配置，因此之前压测的时候知道他即使每秒上万请求也能做到，只不过那个已经是他的极限了，会导致数据库服务器的CPU、磁盘、网络、IO、内存的使用率几乎达到极限。

​	但是一般来说在每秒四五千的请求的话，这样的数据库服务器是没什么问题的，何况经过线上监控统计，现在数据库服务器在高峰期的每秒请求量也就是三四千的样子，因此基本上还没什么大问题

![image-20211224150107260](中间件专栏(RockerMq).assets/image-20211224150107260.png)

**如果系统压力越来越大会怎么样**

![image-20211224150325837](中间件专栏(RockerMq).assets/image-20211224150325837.png)

​	有时候在高峰期负载压力很高的时候，如果数据库的负载较高，会导致数据库服务器的磁盘、IO、CPU的负载都很高，会导致数据库上执行的SQL语句性能有所下降。

​	因此在高峰期的时候，有的时候甚至需要几秒钟的时间完成上述几个步骤。

​	首先针对步骤8里的子步骤过多，速度过慢，让用户支付之后等待时间过长的问题，就是**订单系统第一个需要解决的问题**！

## 4.4 系统的核心流程性能如何？有没有哪个环节拖慢了速度

​	根据系统的负载情况，我们要搞明白线上系统部署的机器情况和数据库的机器情况，每台机器的配置情况，然后想想到底每台机器可以抗多大的访问量。得出当前系统的整体压力。

接着要思考，在当前这样的系统压力下：

- 系统的核心业务流程性能如何？
- 核心流程的每个步骤要耗费多长时间？
- 现在核心流程的性能你满意吗？是否还有优化的空间？
- 在系统高峰期的时候，机器和数据库负载很高，是否对核心流程的性能有影响？
- 如果有影响的话，会有多大的影响？

我的系统总共就几十个人用，根本没有压力可言，这怎么办？

那你就想，你的这个系统做一个SaaS云平台的模式，提供给几万个公司，百万用户使用，不就可以了？你要自己去模拟这个场景。

然后，你按照文中的思路去推算出系统高峰期的负载，以及你的线上系统的机器的压力，到底要部署多少机器去满足这个压力。

## 4.5 系统面临的现实问题：订单退款时经常流程失败，无法完成退款

**复杂的订单支付流程**

![image-20211224152030062](中间件专栏(RockerMq).assets/image-20211224152030062.png)

**对订单进行退款时需要干些什么**

本质上订单退款应该是一个订单支付的逆向过程，也就是说他应该做如下一些事：

- 重新给商品增加库存
- 更新订单状态为“已完成”
- 减少你的积分
- 收回你的优惠券和红包
- 发送Push告诉你退款完成了
- 通知仓储系统取消发货

最重要的是，需要通过第三方支付系统把钱重新退还给你。

而且如果电商平台都已经给你发货了，你才申请退款，实际上你还得把收到的商品给人家快递回去，等他们收到了商品再把钱退还给你。

这是退款的流程图

![image-20211224152554613](中间件专栏(RockerMq).assets/image-20211224152554613.png)

**PS:退款的最大问题：第三方支付系统如果退款失败怎么办(后续??)**

**如果用户下单后一直不付款怎么办**

​	此时订单的状态“待支付”，而且只要你下了订单，你订单里涉及到的商品，都会有对应的锁定库存的一个工作，相当于给你预先保留好这些商品。

​	我们的订单系统会启动一个后台线程，这个后台线程就是专门扫描数据库里那些待付款的订单。

​	如果发现超过24小时还没付款，就直接把订单状态改成“已关闭”了，释放掉锁定的那些商品库存。

![image-20211224152953406](中间件专栏(RockerMq).assets/image-20211224152953406.png)

## 4.6 你们系统出现过核心流程链路失败的情况吗

因为不管是什么系统，无论是一些管理信息系统，还是互联网系统，或者大数据系统，一定有一个核心的链路。

关键步骤失败了，这个时候会怎么样？如果某个步骤没有成功，是不是需要启动后台线程定时扫描进行补偿？

## 4.7 系统面临的现实问题：第三方客户系统的对接耦合性太高，经常出问题

**系统之间的耦合**

![image-20211224153823221](中间件专栏(RockerMq).assets/image-20211224153823221.png)

​	订单系统跟促销系统是强耦合的。因为促销系统任何一点接口修改，都会牵扯你围着他转，去配合他， 耗费你们订单团队的人力和时间，说明你们两个系统耦合在一起了。

​	要动一起动，要静一起静，这就是系统间的耦合。

​	订单系统不就跟仓储系统、第三方物流系统，全部耦合在一起了

​	他的性能突然降低，我们的系统性能就降低了，万一他接口突然调用失败，我们的这次操作也会失败，后续还要考虑重试机制

**PS:第三方系统，永远是不能完全信任的，他随时有可能出现意料之外的性能变差、接口失败的问题**

## 4.8 跟第三方系统对接过，有遇到什么问题

​	负责的系统是否跟某个第三方系统进行了耦合

​	系统跟第三方系统耦合了，是否遇到了性能上的问题？

​	比如你自己系统就只要20ms，结果第三方系统要200ms。

​	是否有稳定性的问题？比如第三方系统的接口有时候会超时、失败。

## 4.9 系统面临的现实问题：大数据团队需要订单数据，该怎么办

**大数据到底是干嘛的**

​	**所以每天如果有100万用户来访问你的APP，积累下来的一些浏览行为、访问行为、交易行为都是各种数据，这个数据量很大，所以你可以称之为“大数据”**

​	大数据团队:尽可能的搜集每天100万用户在你的APP上的各种行为数据

**几百行的大SQL直接查线上库的危害**

​	每次当有几十个几百行的大SQL同时运行在我们订单数据库里的时候，都会导致我们的数据库CPU负载很高，磁盘IO负载很高！

​	一旦我们的数据库负载很高，直接会导致我们的订单系统执行的一些增删改查的操作性能大幅度下降！

## 4.10 自己系统的数据，其他团队需要获取的

​	商品系统，即要对外提供商品数据访问的系统，需要大量的数据，也是需要库存数据、促销数据等等，此时也是需要进行跨系统的数据访问。

​	可以结合这种情况思考一下，在你们公司里，跨系统的数据访问是否存在？都是什么样的场景？

## 4.11 系统面临的现实问题：秒杀活动时数据库压力太大，该怎么缓解

**双11对一个订单系统到底有多大压力**

​	会执行多少条SQL在订单数据库上,一般你可以认为平均每个接口会执行2~3次的数据库操作

​	公司现在积累的注册用户已经千万级了，平时的日活用户都百万级，今年的双11参与活动的用户预计有可能会达到两三百万。

​	假设是这个量级的话，基本可以做一个设想，如果有200万用户参与双11活动，在双11购物最高峰的时候，肯定会比往年的高峰QPS高好几倍，预计有可能今年双11最高峰的时候，会达到每秒至少1万的QPS。

​	也就是说，光是系统被请求的QPS就会达到1万以上，那么系统请求数据库的QPS就会达到2万以上。仅仅凭借我们目前的数据库性能，是无论如何扛不住每秒2万请求的。

## 4.12 系统会不会遇到流量洪峰的场景，导致瞬时压力过大

​	各自的系统平时的QPS有多高

​	完全可以自己写一个简单的QPS统计框架，在你的各个接口被调用的时候，先执行这个QPS统计框架的代码。

​	然后在QPS统计框架里计算各个接口每秒被访问的次数，然后输出到你的日志文件里去即可。

​	当然，更好的方式是采用一些可视化的监控系统去观察你的系统的QPS。

​	接着建议大家去观察一下自己线上数据库的QPS，一般也都是基于一些可视化监控系统去看的

​	假设你的系统突然出现一阵流量洪峰，比如每秒QPS突然暴增100倍，甚至1000倍，此时你的系统能抗住吗？数据库能抗住吗？

## 4.13 一张思维导图给你梳理高并发订单系统面临的技术痛点

![image-20211224163549520](中间件专栏(RockerMq).assets/image-20211224163549520.png)



## 4.14 放大100倍压力，也要找出你系统的技术挑战

总结：

**第一**，先思考一下系统的核心业务流程，当然不是指那种查询之类的操作。所谓核心链路指的是对你的系统进行的数据更新的操作，这才是核心链路，因为查询操作一般来说不涉及复杂的业务逻辑，主要是对数据的展示。

对你的系统的核心链路分析一下，有哪些步骤，这些步骤各自的性能如何，综合起来让你的核心链路的性能如何？在这里是否有改进的空间？

**第二**，可以思考一下，在你的系统中，是否有类似后台线程定时补偿的逻辑？

比如订单时间未支付就要自动关闭它，你们系统里有没有那种后台线程，会定时扫描你的数据，对异常数据进行补偿、自动修复等操作的？

如果有的话，这种数据一般量有多大？如果没有，你可以思考一下，你们系统的核心数据是否需要类似的后台自动扫描机制？

**第三**，可以思考一下，在你的系统里有没有跟第三方系统进行耦合？就是一些核心流程里需要同步调用第三方系统进行查询、更新等操作，第三方系统是否对你的核心链路有性能和稳定性上的影响？

**第四**，可以思考一下，在你的核心链路中，是否存在那种关键步骤可能会失败的情况？万一失败了该怎么办？

**第五**，可以思考一下，平时是否存在其他系统需要获取你们数据的情况？他们是如何获取你们数据的？

是直接跑SQL从你们数据库里查询？或者是调用你们的接口来获取数据？是否存在这种情况？如果有，对你们有什么影响吗？

**第六**，系统是否存在流量洪峰的情况，有时候突然之间访问量增大好几倍，是否会对你们的系统产生无法承受的压力？

# 2 MQ选型及特点

## 2.1 解决订单系统诸多问题的核心技术：消息中间件到底是什么

**同步调用：**

​	用户发起一个请求，系统A收到请求，接着系统A必须立马去调用系统B，直到系统B返回了，系统A才能返回结果给用户，这种模式其实就是所谓的“同步调用”。

![image-20211230111130238](中间件专栏(RockerMq).assets/image-20211230111130238.png)

**依托消息中间件实现异步**

​	异步调用，意思就是系统A先干了自己的工作，然后想办法去通知了系统B。

​	但是系统B什么时候收到通知？什么时候去干自己的工作？这个系统A不管，不想管，也没法管，跟他就没关系了。

​	但是最终在正常下，系统B总会获取到这个通知，然后干自己该干的事儿。

![image-20211230111203993](中间件专栏(RockerMq).assets/image-20211230111203993.png)

**PS:消息中间件，其实就是一种系统，他自己也是独立部署的，然后让我们的两个系统之间通过发消息和收消息，来进行异步的调用，而不是仅仅局限于同步调用**

**消息中间件作用：**

​	**异步化提升性能，降低系统耦合，流量削峰**

​	MQ进行流量削峰的效果，**系统A发送过来的每秒1万请求是一个流量洪峰，然后MQ直接给扛下来了，都存储自己本地磁盘**，这个过程就是流量削峰的过程，瞬间把一个洪峰给削下来了，让系统B后续慢慢获取消息来处理。

**基于MQ优化系统：**

​	能不能提升你的核心链路的性能？能不能降低你系统跟其他系统的耦合度？能不能让你的系统应对流量洪峰？

## 2.2 Kafka、RabbitMQ 以及 RocketMQ 进行技术选型调研

**MQ选型对比**

- 业内常用的MQ有哪些？
- 每一种MQ各自的表现如何？
- 这些MQ在同等机器条件下，能抗多少QPS（每秒抗几千QPS还是几万QPS）？
- 性能有多高（发送一条消息给他要2ms还是20ms）？
- 可用性能不能得到保证（要是MQ部署的机器挂了怎么办）？
- 他们会不会丢失数据？
- 如果需要的话能否让他们进行线性的集群扩容（就是多加几台机器）？
- 消息中间件经常需要使用的一些功能他们都有吗（比如说延迟消息、事务消息、消息堆积、消息回溯、死信队列，等等）？
- 这些MQ在文档是否齐全？社区是否活跃？在行业内是否广泛运用？是用什么语言编写的？

**Kafka、RabbitMQ以及RocketMQ的调研对比**

​	**Kafka的优势和劣势**

​		首先**Kafka的吞吐量几乎是行业里最优秀的，在常规的机器配置下，一台机器可以达到每秒十几万的QPS**，相当的强悍。

​		Kafka性能也很高，基本**上发送消息给Kafka都是毫秒级的性能**。可用性也很高，**Kafka是可以支持集群部署的，其中部分机器宕机是可以继续运行的**。

​		但是Kafka比较为人诟病的一点，似乎是丢数据方面的问题，因为**Kafka收到消息之后会写入一个磁盘缓冲区里，并没有直接落地到物理磁盘上去**，所以要是**机器本身故障了，可能会导致磁盘缓冲区里的数据丢失**。

​		而且**Kafka**另外一个比较大的缺点，就是**功能相对单一**，主要是**支持发送消息给他，然后从里面消费消息，其他就没有什么额外的高级功能了。所以基于Kafka有限的功能，可能适用的场景并不是很多**。

​		因此综上所述，以及查阅了**Kafka技术在各大公司里的使用，基本行业里的一个标准，是把Kafka用在用户行为日志的采集和传输上，比如大数据团队要收集APP上用户的一些行为日志**，这种日志就是用Kafka来收集和传输的。

​		因为那种日志适当丢失数据是没有关系的，而且一般量特别大，要求吞吐量要高，一般就是收发消息，不需要太多的高级功能，所以Kafka是非常适合这种场景的。

​	**RabbitMQ的优势和历史**

​		再说RabbitMQ，在RocketMQ出现之前，国内大部分公司都从ActiveMQ切换到RabbitMQ来使用，包括很多一线互联网大厂，而且直到现在都有很多中小型公司在使用RabbitMQ。

​		**RabbitMQ的优势在于可以保证数据不丢失，也能保证高可用性，即集群部署的时候部分机器宕机可以继续运行，然后支持部分高级功能，比如说死信队列，消息重试之类的**，这些是他的优点。

​		但是他也有一些缺点，最为人诟病的，就是**RabbitMQ的吞吐量是比较低的，一般就是每秒几万的级别，所以如果遇到特别特别高并发的情况下，支撑起来是有点困难的**。

​		而且他进行**集群扩展的时候（也就是加机器部署），还比较麻烦**。

​		另外还有一个较为致命的缺陷，就是他的开**发语言是erlang，国内很少有精通erlang语言的工程师，因此也没办法去阅读他的源代码，甚至修改他的源代码**。

​		所以现在行业里的一个情况是，很多BAT等一线互联网大厂都切换到使用更加优秀的RocketMQ了，但是很多中小型公司觉得RabbitMQ基本可以满足自己的需求还在继续使用中，因为中小型公司并不需要特别高的吞吐量，RabbitMQ已经足以满足他们的需求了，而且也不需要部署特别大规模的集群，也没必要去阅读和修改RabbitMQ的源码。

​	**RocketMQ的优势和劣势**

​		RocketMQ是阿里开源的消息中间件，久经沙场，非常的靠谱。他几乎同时解决了Kafka和RabbitMQ的缺陷。

​		**RocketMQ的吞吐量也同样很高，单机可以达到10万QPS以上，而且可以保证高可用性，性能很高，而且支持通过配置保证数据绝对不丢失，可以部署大规模的集群，还支持各种高级的功能，比如说延迟消息、事务消息、消息回溯、死信队列、消息积压，等等**。

​		而**且RocketMQ是基于Java开发**的，符合国内大多数公司的技术栈，很容易就可以阅读他的源码，甚至是修改他的源码。

​		所以现在国内很多一线互联网大厂都切换为使用RocketMQ了，他们需要RocketMQ的高吞吐量，大规模集群部署能力，以及各种高阶的功能去支撑自己的各种业务场景，同时还可以根据自己的需求定制修改RocketMQ的源码。

​		RocketMQ是非常适合用在Java业务系统架构中的，因为他很高的性能表现，还有他的高阶功能的支持，可以让我们解决各种业务问题。

​		当然，RocketMQ也有一点美中不足的地方，就是经过我的调查发现，RocketMQ的官方文档相对简单一些，但是Kafka和RabbitMQ的官方文档就非常的全面和详细，这可能是RocketMQ目前唯一的缺点。

从架构原理上自己对比一下Kafka、RabbitMQ、RocketMQ

1. 他们都是如何集群化部署抗高并发的？
2. 他们对海量消息是如何分布式存储的？
3. 他们是如何实现主从多备份的高可用架构的？
4. 他们是如何实现集群路由让别人找到对应的机器发送消息和接收消息的？

## 2.3 RocketMQ 的架构原理和使用方式

**MQ如何集群化部署来支撑高并发访问**

​	RocketMQ是可以集群化部署的，可以部署在多台机器上，假设每台机器都能抗10万并发，然后你只要让几十万请求分散到多台机器上就可以了，让每台机器承受的QPS不超过10万

![image-20211230153802761](中间件专栏(RockerMq).assets/image-20211230153802761.png)

**MQ如果要存储海量消息应该怎么做**

​	本质上RocketMQ存储海量消息的机制就是分布式的存储

​	所谓分布式存储，就是把数据分散在多台机器上来存储，每台机器存储一部分消息，这样多台机器加起来就可以存储海量消息了

![image-20211230154928304](中间件专栏(RockerMq).assets/image-20211230154928304.png)

**高可用保障：万一Broker宕机了怎么办**

​	RocketMQ的解决思路是Broker主从架构以及多副本策略。

​	Master Broker收到消息之后会同步给Slave Broker，这样Slave Broker上就能有一模一样的一份副本数据！

​	如果任何一个Master Broker出现故障，还有一个Slave Broker上有一份数据副本，可以保证数据不丢失，还能继续对外提供服务，保证了MQ的可靠性和高可用性

![image-20211230160148745](中间件专栏(RockerMq).assets/image-20211230160148745.png)

**数据路由：怎么知道访问哪个Broker**

​	NameServer的概念，他也是独立部署在几台机器上的，然后所有的Broker都会把自己注册到NameServer上去，NameServer不就知道集群里有哪些Broker了

​	发送消息到Broker，会找NameServer去获取路由信息，就是集群里有哪些Broker等信息

​	如果系统要从Broker获取消息，也会找NameServer获取路由信息，去找到对应的Broker获取消息

![image-20211230160626422](中间件专栏(RockerMq).assets/image-20211230160626422.png)

# 3 RocketMq架构原理

## 3.1 消息中间件路由中心的架构原理是什么

**NameServer部署**

​	通常来说，NameServer一定会多机器部署，实现一个集群，起到高可用的效果，保证任何一台机器宕机，其他机器上的NameServer可以继续对外提供服务

**Broker是把自己的信息注册到哪个NameServer上去**

​	每个Broker启动都得向所有的NameServer进行注册

**系统如何从NameServer获取Broker信息**

​	RocketMQ中的生产者和消费者就是这样，**自己主动去NameServer拉取Broker信息的**

**Broker跟NameServer之间的心跳机制**

​	**Broker会每隔30s给所有的NameServer发送心跳**，告诉每个NameServer自己目前还活着

​	每次NameServer收到一个Broker的心跳，就可以更新一下他的最近一次心跳的时间

​	然后NameServer会每隔10s运行一个任务，去检查一下各个Broker的最近一次心跳时间，如果某个Broker超过120s都没发送心跳了，那么就认为这个Broker已经挂掉了。

![image-20211230172740604](中间件专栏(RockerMq).assets/image-20211230172740604.png)

## 3.2 要是没有这个路由中心，消息中间件可以正常运作么

**路由中心**

​	路由中心的角色需要去感知集群里所有的Broker节点，然后需要去配合生产者和消费者，让人家都能感知到集群里有哪些Broker，才能让各个系统跟MQ进行通信

​	Kafka的路由中心实际上是一个非常复杂、混乱的存在。他是由ZooKeeper以及某个作为Controller的Broker共同完成的。

​	RabbitMQ的话自己本身就是由集群每个节点同时扮演了路由中心的角色。

​	RocketMQ是把路由中心抽离出来作为一个独立的NameServer角色运行的，因此可以说在路由中心这块，他的架构设计是最清晰明了的。

**NameServer集群整体都故障了，失去了这个NameServer集群之后：**

- RocketMQ还能正常运行吗？
- 生产者还能发送消息到Broker吗？
- 消费者还能从Broker拉取消息吗？

## 3.3 Broker的主从架构原理是什么

**Master Broker是如何将消息同步给Slave Broker的**

​	RocketMQ的Master-Slave模式采取的是Slave Broker不停的发送请求到Master Broker去拉取消息

​	RocketMQ自身的Master-Slave模式采取的是**Pull模式**拉取消息

![image-20220104110153725](中间件专栏(RockerMq).assets/image-20220104110153725.png)

**RocketMQ 实现读写分离了吗**

​	问题：作为消费者的系统在获取消息的时候，是从Master Broker获取的？还是从Slave Broker获取的？

​	回答：**有可能从Master Broker获取消息，也有可能从Slave Broker获取消息**

​	1 消费者的系统在获取消息的时候会先发送请求到Master Broker上去，请求获取一批消息，此时Master Broker是会返回一批消息给消费者系统的

​	2 Master Broker在返回消息给消费者系统的时候，会根据当时Master Broker的负载情况和Slave Broker的同步情况，向消费者系统建议下一次拉取消息的时候是从Master Broker拉取还是从Slave Broker拉取

示例：

​	要是这个时候Master Broker负载很重，本身要抗10万写并发了，你还要从他这里拉取消息，给他加重负担，那肯定是不合适的。

​	所以此时Master Broker就会建议你从Slave Broker去拉取消息。

​	或者举另外一个例子，本身这个时候Master Broker上都已经写入了100万条数据了，结果Slave Broke不知道啥原因，同步的特别慢，才同步了96万条数据，落后了整整4万条消息的同步，这个时候你作为消费者系统可能都获取到96万条数据了，那么下次还是只能从Master Broker去拉取消息

总结：

​	所以在写入消息的时候，通常来说肯定是选择Master Broker去写入的

​	但是在拉取消息的时候，有可能从Master Broker获取，也可能从Slave Broker去获取，一切都根据当时的情况来定。

![image-20220104110507602](中间件专栏(RockerMq).assets/image-20220104110507602.png)

 **Broke挂掉的影响**

​	**Slave Broke挂掉了**

​		**有一点影响，但是影响不太大。少了Slave Broker，会导致所有读写压力都集中在Master Broker上**

​	**Master Broker挂掉了**

​		在RocketMQ 4.5版本之前，都是用Slave Broker同步数据，尽量保证数据不丢失，但是一旦Master故障了，Slave是没法自动切换成Master的。

​		所以在这种情况下，如果Master Broker宕机了，这时就得手动做一些运维操作，把Slave Broker重新修改一些配置，重启机器给调整为Master Broker，这是有点麻烦的，而且会导致中间一段时间不可用

**基于Dledger实现RocketMQ高可用自动切换**

​	**基于Raft协议实现的一个机制**

​	**把Dledger融入RocketMQ之后，就可以让一个Master Broker对应多个Slave Broker，也就是说一份数据可以有多份副本**，比如一个Master Broker对应两个Slave Broker

​	一旦**Master Broker宕机了，就可以在多个副本，也就是多个Slave中，通过Dledger技术和Raft协议算法进行leader选举**，直接将一个Slave Broker选举为新的Master Broker，然后这个新的Master Broker就可以对外提供服务了。

​	整个过程也许只要10秒或者几十秒的时间就可以完成，这样的话，就可以实现Master Broker挂掉之后，自动从多个Slave Broker中选举出来一个新的Master Broker，继续对外服务，一切都是自动的



![image-20220104110924181](中间件专栏(RockerMq).assets/image-20220104110924181.png)

问题：

- 假设如果没有RocketMQ 4.5新版本引入的Dledger技术，仅仅是靠之前的Master-Slave主从同步机制，那么在Master崩溃的时候，可能会造成多长时间的系统不可用？这个时候如何能够尽快的恢复集群运行？依赖手工运维的话，如何能尽快的去完成这个运维操作？
- 在RocketMQ 4.5之后引入了Dledger技术可以做到自动选举新的Master，那么在Master崩溃一直到新的Master被选举出来的这个过程中，你觉得对于使用MQ的系统而言，会处于一个什么样的状态呢？
- 希望大家去研究一下Kafka和RabbitMQ的多副本和高可用机制，Kafka是如何在集群里维护多个副本的？出现故障的时候能否实现自动切换？RabbitMQ是如何在集群里维护多个数据副本的？出现故障的时候能否实现自动切换？
- 既然有主从同步机制，那么有没有主从数据不一致的问题？Slave永远落后Master一些数据，这就是主从不一致。那么这种不一致有没有什么问题？有办法保证主从数据强制一致吗？这样做又会有什么缺点呢？

## 3.4  RocketMq基本模式

**同步发送消息（sync message ）**

​	概念：producer向 broker 发送消息，执行 API 时同步等待， 直到broker 服务器返回发送结果 

​	实操：

1）在test下编写测试类，发送同步消息。

```java

 @RunWith(SpringRunner.class)
 @SpringBootTest
 public class ProducerSimpleTest {
 
     @Autowired
     private ProducerSimple producerSimple;
 
     //测试发送同步消息
     @Test
     public void testSendSyncMsg(){
         this.producerSimple.sendSyncMsg("my-topic", "第一条同步消息");
         System.out.println("end...");
     }
 
 }
```

2）启动NameServer、Broker、管理端

3）执行testSendSyncMsg方法

4）观察控制台和管理端

控制台出现end... 表示消息发送成功。

进入管理端，查询消息。

**异步发送消息（async message）**

​	producer向 broker 发送消息时指定消息发送成功及发送异常的回调方法，调用 API 后立即返回，producer发送消息线程不阻塞 ，消息发送成功或失败的回调任务在一个新的线程中执行 。

```java
/**
  * 发送异步消息
  * @param topic
  * @param msg
  */
 public void sendASyncMsg(String topic, String msg){
     rocketMQTemplate.asyncSend(topic,msg,new SendCallback() {
         @Override
         public void onSuccess(SendResult sendResult) {
             //成功回调
             System.out.println(sendResult.getSendStatus());
         }
         @Override
         public void onException(Throwable e) {
             //异常回调
             System.out.println(e.getMessage());
         }
     });
 }
  /**
  * 测试类
  * @version 1.0
  **/
@Test
 public void testSendASyncMsg() throws InterruptedException {
     this.producerSimple.sendASyncMsg("my-topic", "第一条异步步消息");
     System.out.println("end...");
     //异步消息，为跟踪回调线程这里加入延迟
     Thread.sleep(3000);
 }
```

**发送单向消息（oneway message）**

​	producer向 broker 发送消息，执行 API 时直接返回，不等待broker 服务器的结果 。

```java
 /**
  * 发送单向消息
  * @param topic
  * @param msg
  */
 public void sendOneWayMsg(String topic, String msg){
     this.rocketMQTemplate.sendOneWay(topic,msg);
 }
```

**Push消费模式**

​	push，主动推送给消费者

​	push方式里，consumer把轮询过程封装了，并注册MessageListener监听器，取到消息后，唤醒MessageListener的consumeMessage()来消费，对用户而言，感觉消息是被推送过来的

​	优缺点：实时性高，但增加服务端负载，消费端能力不同，如果push的速度过快，消费端会出现很多问题

```java
import cn.hutool.core.lang.Console;
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
import org.apache.rocketmq.common.protocol.heartbeat.MessageModel;

public class PushConsumer {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("PushConsumerGroupName");
        consumer.setNamesrvAddr("127.0.0.1:9876");
        //一个GroupName第一次消费时的位置
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_LAST_OFFSET);
        consumer.setConsumeThreadMin(20);
        consumer.setConsumeThreadMax(20);
        //要消费的topic，可使用tag进行简单过滤
        consumer.subscribe("TestTopic", "*");
        //一次最大消费的条数
        consumer.setConsumeMessageBatchMaxSize(100);
        //消费模式，广播或者集群，默认集群。
        consumer.setMessageModel(MessageModel.CLUSTERING);
        //在同一jvm中 需要启动两个同一GroupName的情况需要这个参数不一样。
        consumer.setInstanceName("InstanceName");
        //配置消息监听
        consumer.registerMessageListener((MessageListenerConcurrently) (msgs, context) -> {
            try {
                //业务处理
                msgs.forEach(msg -> {
                    Console.log(msg);
                });
            } catch (Exception e) {
                System.err.println("接收异常" + e);
                return ConsumeConcurrentlyStatus.RECONSUME_LATER;
            }
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });
        consumer.start();
        System.out.println("Consumer Started.");
    }
}
```

**Pull消费模式**

​	pull，消费者主动去broker拉取

​	pull方式里，取消息的过程需要用户自己写，首先通过打算消费的Topic拿到MessageQueue的集合，遍历MessageQueue集合，然后针对每个MessageQueue批量取消息，一次取完后，记录该队列下一次要取的开始offset，直到取完了，再换另一个MessageQueue

​	优缺点：消费者从server端拉消息，主动权在消费端，可控性好，但是时间间隔不好设置，间隔太短，则空请求会多，浪费资源，间隔太长，则消息不能及时处理

```java
import cn.hutool.core.collection.CollUtil;
import cn.hutool.core.lang.Console;
import org.apache.rocketmq.client.consumer.DefaultLitePullConsumer;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

public class PullConsumer {
    private static boolean runFlag = true;
    public static void main(String[] args) throws Exception {
        DefaultLitePullConsumer consumer = new DefaultLitePullConsumer("PullConsumerGroupName");
        consumer.setNamesrvAddr("127.0.0.1:9876");
        //要消费的topic，可使用tag进行简单过滤
        consumer.subscribe("TestTopic", "*");
        //一次最大消费的条数
        consumer.setPullBatchSize(100);
        //无消息时，最大阻塞时间。默认5000 单位ms
        consumer.setPollTimeoutMillis(5000);
        consumer.start();
        while (runFlag){
            try {
                //拉取消息，无消息时会阻塞 
                List<MessageExt> msgs = consumer.poll();
                if (CollUtil.isEmpty(msgs)){
                    continue;
                }
                //业务处理
                msgs.forEach(msg-> Console.log(new String(msg.getBody())));
                //同步消费位置。不执行该方法，应用重启会存在重复消费。
                consumer.commitSync();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        consumer.shutdown();
    }
}
```

引用：

https://zhuanlan.zhihu.com/p/138652070

https://www.cnblogs.com/enenen/p/12773099.html

https://blog.csdn.net/qq_21383435/article/details/101113808

# 4 RocketMq落地及设计

## 4.1 设计一套高可用的消息中间件生产部署架构

**设计：**

​	**1 NameServer集群化部署，保证高可用性**

​	**2 基于Dledger的Broker主从架构部署**

​	**3 使用MQ的系统都要多机器集群部署**

​	**4 整体架构：高可用、高并发、海量消息、可伸缩**

**Broker是如何跟NameServer进行通信的**

​	**基本概念：**

​	**Broker会每隔30秒发送心跳到所有的NameServer上去，然后每个NameServer都会每隔10s检查一次有没有哪个Broker超过120s没发送心跳的，如果有，就认为那个Broker已经宕机了，从路由信息里要摘除这个Broker**

​	**流程：**

​	Broker会跟每个NameServer都建立一个TCP长连接，然后定时通过TCP长连接发送心跳请求过去

​	各个NameServer就是通过跟Broker建立好的长连接不断收到心跳包，然后定时检查Broker有没有120s都没发送心跳包，来判定集群里各个Broker到底挂掉了没有

**MQ的核心数据模型：Topic**

​	**基本概念：**

​	系统如果要往MQ里写入消息或者获取消息，首先得创建一些Topic，作为数据集合存放不同类型的消息，比如说订单Topic，商品Topic，等等

​	**分布式存储：**

​	可以在创建Topic的时候指定让他里面的数据分散存储在多台Broker机器上，比如一个Topic里有1000万条数据，此时有2台Broker，那么就可以让每台Broker上都放500万条数据

​	每个Broke在进行定时的心跳汇报给NameServer的时候，都会告诉NameServer自己当前的数据情况，比如有哪些Topic的哪些数据在自己这里，这些信息都是属于路由信息的一部分

**生产者系统是如何将消息发送到Broker的**

​	**流程：**

​	发送消息之前，得先有一个Topic，然后在发送消息的时候你得指定你要发送到哪个Topic里面去

​	知道你要发送的Topic，那么就可以跟NameServer建立一个TCP长连接，然后定时从他那里拉取到最新的路由信息，包括集群里有哪些Broker，集群里有哪些Topic，每个Topic都存储在哪些Broker上

​	生产者系统自然就**可以通过路由信息找到自己要投递消息的Topic分布在哪几台Broker上**，此时可以**根据负载均衡算法，从里面选择一台Broke机器出来**，比如round robine轮询算法，或者是hash算法，都可以

​	选择一台Broker之后，就可以跟那个**Broker也建立一个TCP长连接，然后通过长连接向Broker发送消息**即可

​	Broker收到消息之后就会存储在自己本地磁盘里去

​	**PS:生产者一定是投递消息到Master Broker的，然后Master Broker会同步数据给他的Slave Brokers，实现一份数据多份副本，保证Master故障的时候数据不丢失，而且可以自动把Slave切换为Master提供服务**

**消费者是如何从Broker上拉取消息的**

​	会**跟NameServer建立长连接，然后拉取路由信息**，接着找到自己要获取消息的**Topic在哪几台Broker上，就可以跟Broker建立长连接，从里面拉取消息了**

​	消费者系统可能会从Master Broker拉取消息，也可能从Slave Broker拉取消息，都有可能，一切都看具体情况。

![image-20220105143409318](中间件专栏(RockerMq).assets/image-20220105143409318.png)

**问题：**

- 在你们公司里有用MQ吗？
- 如果有的话是什么MQ？
- 你们的MQ在生产环境的部署架构是怎么做的？
- 路由中心、MQ集群、生产者和消费者分别是怎么部署的？为什么要那样部署？
- 那样部署可以实现高并发、海量消息、高可用和线性可伸缩吗？
- RabbitMQ和Kafka他们两个是如何实现生产架构部署，来支撑高并发、海量消息、高可用和可伸缩的呢？他们能实现这些吗？

## 4.2 部署一个小规模的 RocketMQ 集群

**前置条件**

​	1 jdk 1.7及以上(一般是1.8)

​	2 maven 3.5 及以上

​	3 git

​	4 4g内存及以上的虚拟机(linux服务器)

**部署流程**

​	**构建Dledger**

```sh
git clone https://github.com/openmessaging/openmessaging-storage-dledger.git
#需要网络环境良好才能clone
#不能从windows拷贝过来，拷贝过来会有很多window目录的不兼容问题
cd openmessaging-storage-dledger

mvn clean install -DskipTests
```

​	**构建RocketMQ**

```sh
git clone https://github.com/apache/rocketmq.git
#需要网络环境良好才能clone
#不能从windows拷贝过来，拷贝过来会有很多window目录的不兼容问题
cd rocketmq

git checkout -b store_with_dledger origin/store_with_dledger

mvn -Prelease-all -DskipTests clean install -U
```

​	**编辑相关文件**

```sh
cd distribution/target/apache-rocketmq
# 本地配好的目录/home/mq/rocketmq/distribution/target/apache-rocketmq
# 在这个目录中，需要编辑三个文件，一个是bin/runserver.sh，一个是bin/runbroker.sh，另外一个是bin/tools.sh
#在里面找到如下三行，然后将第二行和第三行都删了，同时将第一行的值修改为你自己的JDK的主目录
#[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=$HOME/jdk/java
#[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/java
#[ ! -e "$JAVA_HOME/bin/java" ] && error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!"
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/java/jdk1.8.0_202-amd64
#本地配好的，参考用
#
```

​	**本地配好的目录**

![image-20220117171143431](中间件专栏(RockerMq).assets/image-20220117171143431.png)

**PS:如果本地虚拟机内存不大，需要讲对应bin/runserver.sh等3个文件的jvm内存都改小(最好1g及一下)，才能正常启动**

**快速启动RocketMQ集群**

```sh
sh bin/dledger/fast-try.sh start
#这个命令会在当前这台机器上启动一个NameServer和三个Broker，三个Broker其中一个是Master，另外两个是Slave，瞬间就可以组成一个最小可用的RocketMQ集群。
```

**常用命令**

​	检查Broker集群的状态

```sh
sh bin/mqadmin clusterList -n 127.0.0.1:9876
# 检查一下RocketMQ集群的状态
```

![image-20220117171851912](中间件专栏(RockerMq).assets/image-20220117171851912.png)

​	**PS:BID为0的就是Master，BID大于0的就都是Slave，其实在这里也可以叫做Leader和Follower**

​	NameServer相关

```sh
nohup sh mqnamesrv &
#启动NameServer
#这个NameServer监听的接口默认就是9876，所以如果你在三台机器上都启动了NameServer，那么他们的端口都是9876，此时我们就成功的启动了三个NameServer了
```

​	Broker相关

```sh
nohup sh bin/mqbroker -c conf/dledger/broker-n0.conf &
#bin/mqbroker为启动broker 
#conf/dledger/broker-n0.conf 为指定配置文件
#第一个Broker的配置文件是broker-n0.conf，第二个broker的配置文件可以是broker-n1.conf，第三个broker的配置文件可以是broker-n2.conf
```

**配置文件说明**

```sh
# 这个是集群的名称，你整个broker集群都可以用这个名称
brokerClusterName=RaftCluster

# 这是Broker的名称，比如你有一个Master和两个Slave，那么他们的Broker名称必须是一样的，因为他们三个是一个分组，如果你有另外一组Master和两个Slave，你可以给他们起个别的名字，比如说RaftNode01
brokerName=RaftNode00

# 这个就是你的Broker监听的端口号，如果每台机器上就部署一个Broker，可以考虑就用这个端口号，不用修改
listenPort=30911

# 这里是配置NameServer的地址，如果你有很多个NameServer的话，可以在这里写入多个NameServer的地址
namesrvAddr=127.0.0.1:9876

# 下面两个目录是存放Broker数据的地方，你可以换成别的目录，类似于是/usr/local/rocketmq/node00之类的
storePathRootDir=/tmp/rmqstore/node00
storePathCommitLog=/tmp/rmqstore/node00/commitlog

# 这个是非常关键的一个配置，就是是否启用DLeger技术，这个必须是true
enableDLegerCommitLog=true

# 这个一般建议和Broker名字保持一致，一个Master加两个Slave会组成一个Group
dLegerGroup=RaftNode00

# 这个很关键，对于每一组Broker，你得保证他们的这个配置是一样的，在这里要写出来一个组里有哪几个Broker，比如在这里假设有三台机器部署了Broker，要让他们作为一个组，那么在这里就得写入他们三个的ip地址和监听的端口号
dLegerPeers=n0-127.0.0.1:40911;n1-127.0.0.1:40912;n2-127.0.0.1:40913

# 这个是代表了一个Broker在组里的id，一般就是n0、n1、n2之类的，这个你得跟上面的dLegerPeers中的n0、n1、n2相匹配
dLegerSelfId=n0

# 这个是发送消息的线程数量，一般建议你配置成跟你的CPU核数一样，比如我们的机器假设是24核的，那么这里就修改成24核
sendMessageThreadPoolNums=24
```

**主备切换测试**

​	此时我们可以用命令（lsof -i:30921）找出来占用30921端口的进程PID，接着就用kill -9的命令给他杀了，比如我这里占用30921端口的进程PID是4344，那么就执行命令：kill -9 4344

​	接着等待个10s，再次执行命令查看集群状态：

​	sh bin/mqadmin clusterList -n 127.0.0.1:9876

​	此时就会发现作为Leader的BID为0的节点，变成另外一个Broker了，这就是说Slave切换为Master了。

**流程总结：**

​	其实最关键的是，你的**Broker是分为多组的，每一组是三个Broker，一个Master和两个Slave。**

​	对**每一组Broker，他们的Broker名称、Group名称都是一样的，然后你得给他们配置好一样的dLegerPeers（里面是组内三台Broker的地址）**

​	然后他们得配置好对应的NameServer的地址，最后还有就是**每个Broker有自己的ID，在组内是唯一的就可以了，比如说不同的组里都有一个ID为n0的broker，这个是可以的。**

​	所以按照这个思路就可以轻松的配置好一组Broker，在三台机器上分别用命令启动Broker即可。启动完成过后，可以跟NameServer进行通信，检查Broker集群的状态

**代码测试**

​	pom

```xml
        <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-client</artifactId>
            <version>4.5.1</version>
        </dependency>
```

​	producer

```java
    public static void main(String[] args) throws MQClientException, InterruptedException {
        final DefaultMQProducer defaultMQProducer = new DefaultMQProducer("test_producer");
        defaultMQProducer.setNamesrvAddr("192.168.21.101:9876");
        defaultMQProducer.setSendMsgTimeout(60000);
        defaultMQProducer.start();
        for(int i=0;i<10;i++){
            new Thread(()->{
                while (true){
                    try {
                        Message message = new Message("TopicTest","TagA",("Test").getBytes(RemotingHelper.DEFAULT_CHARSET));
                        SendResult send = defaultMQProducer.send(message);
                        System.out.println(send);
                        //10秒发一次
                        Thread.sleep(10000);
                    } catch (UnsupportedEncodingException e) {
                        e.printStackTrace();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } catch (RemotingException e) {
                        e.printStackTrace();
                    } catch (MQClientException e) {
                        e.printStackTrace();
                    } catch (MQBrokerException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }
```

​	consumer

```java
    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("test_consumer");
        consumer.setNamesrvAddr("192.168.21.101:9876");
        consumer.subscribe("TopicTest","*");
        //注册回调接口，消费消息
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list,
                                                            ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                for (MessageExt messageExt : list) {
                    System.out.println("接收消息："+new String(messageExt.getBody()));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
    }
```

## 4.3 如何对RocketMQ集群进行可视化的监控和管理

**监控-部署流程：**

```sh
git clone https://github.com/apache/rocketmq-dashboard
#下载对应监控项目
cd rocketmq-externals/rocketmq-dashboard
mvn package -DskipTests
#打成jar包
java -jar rocketmq-console-ng-1.0.1.jar --server.port=8080 --rocketmq.config.namesrvAddr=127.0.0.1:9876
#启动项目并指定监控的nameserver地址,如果有多个地址可以用分号隔开
```

**监控界面相关**

​	在这个界面里可以让你看到Broker的大体消息负载，还有各个Topic的消息负载，另外还可以选择日期要看哪一天的监控数据，都可以看到。

![image-20220120140050383](中间件专栏(RockerMq).assets/image-20220120140050383.png)

​	可以看到各个Broker的分组，哪些是Master，哪些是Slave，他们各自的机器地址和端口号，还有版本号

​	包括最重要的，就是他们每台机器的生产消息TPS和消费消息TPS，还有消息总数。

​	这是非常重要的，通过这个TPS统计，就是每秒写入或者被消费的消息数量，就可以看出RocketMQ集群的TPS和并发访问量。

​	另外在界面右侧有两个按钮，一个是“状态”，一个是“配置”。其中点击状态可以看到这个Broker更加细节和具体的一些统计项，点击配置可以看到这个Broker具体的一些配置参数的值。

![image-20220120140122494](中间件专栏(RockerMq).assets/image-20220120140122494.png)

​	可以在这里创建、删除和管理Topic，查看Topic的一些装填、配置，等等，可以对Topic做各种管理。

![image-20220120140131880](中间件专栏(RockerMq).assets/image-20220120140131880.png)

消费者

![image-20220120140147474](中间件专栏(RockerMq).assets/image-20220120140147474.png)

生产者

![image-20220120140437711](中间件专栏(RockerMq).assets/image-20220120140437711.png)

消息查询

![image-20220120140458536](中间件专栏(RockerMq).assets/image-20220120140458536.png)

死信消息

![image-20220120140507727](中间件专栏(RockerMq).assets/image-20220120140507727.png)

消息轨迹

![image-20220120140526458](中间件专栏(RockerMq).assets/image-20220120140526458.png)

## 4.4 进行OS内核参数和JVM参数的调整

**OS内核调整**

​	vm.overcommit_memory

```sh
#vm.overcommit_memory”这个参数有三个值可以选择，0、1、2。
#如果值是0的话，在你的中间件系统申请内存的时候，os内核会检查可用内存是否足够，如果足够的话就分配内存给你，如果感觉剩余内存不是太够了，干脆就拒绝你的申请，导致你申请内存失败，进而导致中间件系统异常出错。

#因此一般需要将这个参数的值调整为1，意思是把所有可用的物理内存都允许分配给你，只要有内存就给你来用，这样可以避免申请内存失败的问题。

#比如我们曾经线上环境部署的Redis就因为这个参数是0，导致在save数据快照到磁盘文件的时候，需要申请大内存的时候被拒绝了，进而导致了异常报错。

#可以用如下命令修改：
echo 'vm.overcommit_memory=1' >> /etc/sysctl.conf。
```

​	vm.max_map_count

```sh
#这个参数的值会影响中间件系统可以开启的线程的数量，同样也是非常重要的

#如果这个参数过小，有的时候可能会导致有些中间件无法开启足够的线程，进而导致报错，甚至中间件系统挂掉。

#他的默认值是65536，但是这个值有时候是不够的，比如我们大数据团队的生产环境部署的Kafka集群曾经有一次就报出过这个异常，说无法开启足够多的线程，直接导致Kafka宕机了。

#因此建议可以把这个参数调大10倍，比如655360这样的值，保证中间件可以开启足够多的线程。

#可以用如下命令修改：
echo 'vm.max_map_count=655360' >> /etc/sysctl.conf。
```

​	vm.swappiness

```sh
#这个参数是用来控制进程的swap行为的，这个简单来说就是os会把一部分磁盘空间作为swap区域，然后如果有的进程现在可能不是太活跃，就会被操作系统把进程调整为睡眠状态，把进程中的数据放入磁盘上的swap区域，然后让这个进程把原来占用的内存空间腾出来，交给其他活跃运行的进程来使用。

#如果这个参数的值设置为0，意思就是尽量别把任何一个进程放到磁盘swap区域去，尽量大家都用物理内存。

#如果这个参数的值是100，那么意思就是尽量把一些进程给放到磁盘swap区域去，内存腾出来给活跃的进程使用。

#默认这个参数的值是60，有点偏高了，可能会导致我们的中间件运行不活跃的时候被迫腾出内存空间然后放磁盘swap区域去。

#因此通常在生产环境建议把这个参数调整小一些，比如设置为10，尽量用物理内存，别放磁盘swap区域去。

#可以用如下命令修改：
echo 'vm.swappiness=10' >> /etc/sysctl.conf。
```

​	ulimit

```sh
#对于一个中间件系统而言肯定是不能使用默认值的，如果你采用默认值，很可能在线上会出现如下错误：error: too many open files。

#因此通常建议用如下命令修改这个值
echo 'ulimit -n 1000000' >> /etc/profile。
```

​	OS参数总结

​	磁盘文件IO、网络通信、内存管理、线程数量有关系的

- 中间件系统肯定要开启大量的线程**（跟vm.max_map_count有关）**
- 而且要进行大量的网络通信和磁盘IO**（跟ulimit有关）**
- 然后大量的使用内存**（跟vm.swappiness和vm.overcommit_memory有关）**

**JVM相关参数调整**

​	脚本参数

```sh
#在rocketmq/distribution/target/apache-rocketmq/bin目录下，就有对应的启动脚本，比如mqbroker是用来启动Broker的，mqnamesvr是用来启动NameServer的。

#用mqbroker来举例，我们查看这个脚本里的内容，最后有如下一行：

sh ${ROCKETMQ_HOME}/bin/runbroker.sh org.apache.rocketmq.broker.BrokerStartup $@

#这一行内容就是用runbroker.sh脚本来启动一个JVM进程，JVM进程刚开始执行的main类就是org.apache.rocketmq.broker.BrokerStartup

#我们接着看runbroker.sh脚本，在里面可以看到如下内容：

JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"

JAVA_OPT="${JAVA_OPT} -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0"

JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:/dev/shm/mq_gc_%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintAdaptiveSizePolicy"

JAVA_OPT="${JAVA_OPT} -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m"

JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow"

JAVA_OPT="${JAVA_OPT} -XX:+AlwaysPreTouch"

JAVA_OPT="${JAVA_OPT} -XX:MaxDirectMemorySize=15g"

JAVA_OPT="${JAVA_OPT} -XX:-UseLargePages -XX:-UseBiasedLocking"

JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${JAVA_HOME}/jre/lib/ext:${BASE_DIR}/lib"

#JAVA_OPT="${JAVA_OPT} -Xdebug -Xrunjdwp:transport=dt_socket,address=9555,server=y,suspend=n"

JAVA_OPT="${JAVA_OPT} ${JAVA_OPT_EXT}"

JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"
```

​	JVM启动参数说明

```sh
#这个参数就是说用服务器模式启动，这个没什么可说的，现在一般都是如此
-server：

#这个就是很关键的一块参数了，也是重点需要调整的，就是默认的堆大小是8g内存，新生代是4g内存，如果物理机是48g内存的所以这里完全可以给他们翻几倍，比如给堆内存20g，其中新生代给10g，甚至可以更多一些，当然要留一些内存给操作系统来用
-Xms8g -Xmx8g -Xmn4g：

#这几个参数也是至关重要的，这是选用了G1垃圾回收器来做分代回收，对新生代和老年代都是用G1来回收，这里把G1的region大小设置为了16m，这个因为机器内存比较多，所以region大小可以调大一些给到16m，不然用2m的region，会导致region数量过多的
-XX:+UseG1GC -XX:G1HeapRegionSize=16m：

#这个参数是说，在G1管理的老年代里预留25%的空闲内存，保证新生代对象晋升到老年代的时候有足够空间，避免老年代内存都满了，新生代有对象要进入老年代没有充足内存了，默认值是10%，略微偏少，这里RocketMQ给调大了一些
-XX:G1ReservePercent=25：

#这个参数是说，当堆内存的使用率达到30%之后就会自动启动G1的并发垃圾回收，开始尝试回收一些垃圾对象，默认值是45%，这里调低了一些，也就是提高了GC的频率，但是避免了垃圾对象过多，一次垃圾回收耗时过长的问题
-XX:InitiatingHeapOccupancyPercent=30：

#这个参数默认设置为0了，在JVM优化专栏中，救火队队长讲过这个参数引发的案例，其实建议这个参数不要设置为0，避免频繁回收一些软引用的Class对象，这里可以调整为比如1000
-XX:SoftRefLRUPolicyMSPerMB=0：

#这一堆参数都是控制GC日志打印输出的，确定了gc日志文件的地址，要打印哪些详细信息，然后控制每个gc日志文件的大小是30m，最多保留5个gc日志文件
-verbose:gc -Xloggc:/dev/shm/mq_gc_%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintAdaptiveSizePolicy -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m：

#这个参数是说，有时候JVM会抛弃一些异常堆栈信息，因此这个参数设置之后，就是禁用这个特性，要把完整的异常堆栈信息打印出来
-XX:-OmitStackTraceInFastThrow：

#这个参数的意思是我们刚开始指定JVM用多少内存，不会真正分配给他，会在实际需要使用的时候再分配给他，所以使用这个参数之后，就是强制让JVM启动的时候直接分配我们指定的内存，不要等到使用内存的时候再分配
-XX:+AlwaysPreTouch：

#这是说RocketMQ里大量用了NIO中的direct buffer，这里限定了direct buffer最多申请多少，如果你机器内存比较大，可以适当调大这个值
-XX:MaxDirectMemorySize=15g：

#这两个参数的意思是禁用大内存页和偏向锁
-XX:-UseLargePages -XX:-UseBiasedLocking：

#总结
#RocketMQ默认的JVM参数是采用了G1垃圾回收器，默认堆内存大小是8G。这个其实完全可以根据大家的机器内存来调整，你可以增大一些也是没有问题的，然后就是一些G1的垃圾回收的行为参数做了调整，这个一般我们不用去动，然后就是对GC日志打印做了设置，这个一般也不用动。其余的就是禁用一些特性，开启一些特性，这些都直接维持RocketMQ的默认值即可。
```

**对RocketMQ核心参数进行调整**

```sh
#在下面的目录里有dledger的示例配置文件：
rocketmq/distribution/target/apache-rocketmq/conf/dledger

#在这里主要是有一个较为核心的参数：
#这个参数的意思就是RocketMQ内部用来发送消息的线程池的线程数量，默认是16
#其实这个参数可以根据你的机器的CPU核数进行适当增加，比如机器CPU是24核的，可以增加这个线程数量到24或者30，都是可以的。
sendMessageThreadPoolNums=16
```

**总结：**

**（1）**中间件系统在压测或者上生产之前，需要对三大块参数进行调整：**OS内核参数、JVM参数以及中间件核心参数**

**（2）**OS内核参数主要调整的地方都是跟磁盘IO、网络通信、内存管理以及线程管理有关的，需要适当调节大小

**（3）**JVM参数需要我们去中间件系统的启动脚本中寻找他的默认JVM参数，然后根据机器的情况，对JVM的堆内存大小，新生代大小，Direct Buffer大小，等等，做出一些调整，发挥机器的资源

**（4）**中间件核心参数主要也是关注其中跟网络通信、磁盘IO、线程数量、内存 管理相关的，根据机器资源，适当可以增加网络通信线程，控制同步刷磁盘或者异步刷磁盘，线程数量有多少，内存中一些队列的大小

## 4.5 对小规模RocketMQ集群进行压测，同时为生产集群进行规划

**压测思路：**

​	平时做压测，主要关注的还是要压测出来一个最合适的最高负载。

​	什么叫最合适的最高负载呢？

​	意思就是**在RocketMQ的TPS和机器的资源使用率和负载之间取得一个平衡。**

​	比如RocketMQ集群在机器资源使用率极高的极端情况下可以扛到10万TPS，但是当他仅仅抗下8万TPS的时候，你会发现cpu负载、内存使用率、IO负载和网卡流量，都负载较高，但是可以接受，机器比较安全，不至于宕机。

​	那么这个8万TPS实际上就是最合适的一个最高负载，也就是说，哪怕生产环境中极端情况下，RocketMQ的TPS飙升到8万TPS，你知道机器资源也是大致可以抗下来的，不至于出现机器宕机的情况。

​	所以我们做压测，其实最主要的是综合TPS以及机器负载，尽量找到一个最高的TPS同时机器的各项负载在可承受范围之内，这才是压测的目的。

**压测测试样例：**

（1）RocketMQ的TPS和消息延时

​	两个Producer不停的往RocketMQ集群发送消息，每个Producer所在机器启动了80个线程，相当于每台机器有80个线程并发的往RocketMQ集群写入消息。

​	而RocketMQ集群是1主2从组成的一个dledger模式的高可用集群，只有一个Master Broker会接收消息的写入。

​	然后有2个Cosumer不停的从RocketMQ集群消费数据。

​	每条数据的大小是500个字节，**这个非常关键**，因为这个数字是跟后续的网卡流量有关的。

​	一条消息从Producer生产出来到经过RocketMQ的Broker存储下来，再到被Consumer消费，基本上这个时间跨度不会超过1秒钟，这些这个性能是正常而且可以接受的。

​	同时在RocketMQ的管理工作台中可以看到，Master Broker的TPS（也就是每秒处理消息的数量），可以稳定的达到7万左右，也就是每秒可以稳定处理7万消息。

（2）cpu负载情况

​	可以通过top、uptime等命令来查看

​	比如执行top命令就可以看到cpu load和cpu使用率，这就代表了cpu的负载情况。

​	在你执行了top命令之后，往往可以看到如下一行信息：

​	load average：12.03，12.05，12.08

​	类似上面那行信息代表的是cpu在1分钟、5分钟和15分钟内的cpu负载情况

​	比如我们一台机器是24核的，那么上面的12意思就是有12个核在使用中。换言之就是还有12个核其实还没使用，cpu还是有很大余力的。

（3）内存使用率

​	free命令就可以查看到内存的使用率，根据当时的测试结果，机器上48G的内存，仅仅使用了一部分，还剩下很大一部分内存都是空闲可用的，或者是被RocketMQ用来进行磁盘数据缓存了。所以内存负载是很低的。

（4）JVM GC频率

​	使用jstat命令就可以查看RocketMQ的JVM的GC频率，基本上新生代每隔几十秒会垃圾回收一次，每次回收过后存活的对象很少，几乎不进入老年代

​	因此测试过程中，Full GC几乎一次都没有。

（5）磁盘IO负载

​	可以用top命令查看一下IO等待占用CPU时间的百分比，你执行top命令之后，会看到一行类似下面的东西：

​	Cpu(s): 0.3% us, 0.3% sy, 0.0% ni, 76.7% id, 13.2% wa, 0.0% hi, 0.0% si。

​	在这里的13.2% wa，说的就是磁盘IO等待在CPU执行时间中的百分比

​	如果这个比例太高，说明CPU执行的时候大部分时间都在等待执行IO，也就说明IO负载很高，导致大量的IO等待。

​	这个当时我们压测的时候，是在40%左右，说明IO等待时间占用CPU执行时间的比例在40%左右，这是相对高一些，但还是可以接受的，只不过如果继续让这个比例提高上去，就很不靠谱了，因为说明磁盘IO负载可能过高了。

（6）网卡流量

使用如下命令可以查看服务器的网卡流量：

```sh
sar -n DEV 1 2
#通过这个命令就可以看到每秒钟网卡读写数据量了。当时我们的服务器使用的是千兆网卡，千兆网卡的理论上限是每秒传输128M数据，但是一般实际最大值是每秒传输100M数据。

#因此当时我们发现的一个问题就是，在RocketMQ处理到每秒7万消息的时候，每条消息500字节左右的大小的情况下，每秒网卡传输数据量已经达到100M了，就是已经达到了网卡的一个极限值了。
#因为一个Master Broker服务器，每秒不光是通过网络接收你写入的数据，还要把数据同步给两个Slave Broker，还有别的一些网络通信开销。
#因此实际压测发现，每条消息500字节，每秒7万消息的时候，服务器的网卡就几乎打满了，无法承载更多的消息了。
```

![image-20220121172101586](中间件专栏(RockerMq).assets/image-20220121172101586.png)

**压测样例总结：**

​	最后针对本次压测做一点小的总结，实际上经过压测，最终发现我们的服务器的性能瓶颈在网卡上，因为网卡每秒能传输的数据是有限的

​	因此当我们使用平均大小为500字节的消息时，最多就是做到RocketMQ单台服务器每秒7万的TPS，而且这个时候cpu负载、内存负载、jvm gc负载、磁盘io负载，基本都还在正常范围内。

​	只不过这个时候网卡流量基本已经打满了，无法再提升TPS了。

​	因此在这样的一个机器配置下，RocketMQ一个比较靠谱的TPS就是7万左右

**总结**

1. 到底应该如何压测：应该在TPS和机器的cpu负载、内存使用率、jvm gc频率、磁盘io负载、网络流量负载之间取得一个平衡，尽量让TPS尽可能的提高，同时让机器的各项资源负载不要太高。
2. 实际压测过程：采用几台机器开启大量线程并发读写消息，然后观察TPS、cpu load（使用top命令）、内存使用率（使用free命令）、jvm gc频率（使用jstat命令）、磁盘io负载（使用top命令）、网卡流量负载（使用sar命令），不断增加机器和线程，让TPS不断提升上去，同时观察各项资源负载是否过高。
3. 生产集群规划：根据公司的后台整体QPS来定，稍微多冗余部署一些机器即可，实际部署生产环境的集群时，使用高配置物理机，同时合理调整os内核参数、jvm参数、中间件核心参数，如此即可

# 5 基于RocketMq改造订单系统

## 5.1 基于MQ实现订单系统的核心流程异步化改造

订单系统面临的技术问题包括以下一些环节：

1. 下单核心流程环节太多，性能较差
2. 订单退款的流程可能面临退款失败的风险
3. 关闭过期订单的时候，存在扫描大量订单数据的问题
4. 跟第三方物流系统耦合在一起，性能存在抖动的风险
5. 大数据团队要获取订单数据，存在不规范直接查询订单数据库的问题
6. 做秒杀活动时订单数据库压力过大

在订单系统中引入MQ技术来实现订单核心流程中的部分环节的异步化改造

支付完一个订单后，都需要执行一系列的动作，包括：

- 更新订单状态
- 扣减库存
- 增加积分
- 发优惠券
- 发短信
- 通知发货

订单核心流程的改造图

![image-20220124150601743](中间件专栏(RockerMq).assets/image-20220124150601743.png)

**PS:集群部署的话，Topic是一个逻辑上的概念，实际上他的数据是分布式存储在多个Master Broker中的**

**思考：**

​	谓的核心链路，不是说查询链路，即并不是一次请求全部是查询。而是说的是数据更新链路，即一次请求过后会对你的各种核心数据进行更新，同时还会调用其他服务或者系统进行数据更新或者查询，这样的一个链路叫做系统的核心链路。

​	有没有可能引入MQ技术把一些耗时的步骤做成异步化的方式，来优化核心数据链路的性能？

​	如果可以的话，你应该如何设计这个技术方案？哪些环节同步执行？哪些环节要异步执行？

## 5.2 基于MQ实现订单系统的第三方系统异步对接改造

实际上订单系统现在已经不需要直接调用推送系统和仓储系统了，仅仅只是发送一个消息到RocketMQ而已

![image-20220125163847148](中间件专栏(RockerMq).assets/image-20220125163847148.png)

问题：

​	系统是否跟第三方系统存在耦合的问题？尤其是在核心数据链路中，是否存在因为耦合了第三方系统导致性能经常出现抖动的问题？

​	能否在核心链路中引入MQ来跟第三方系统进行解耦？如果解耦之后能对你们核心链路的性能有多高的提升？

​	Kafka和RabbitMQ在使用的时候，有几种消息发送模式？有几种消息消费模式？

​	系统如果使用了MQ技术的话，那么你们平时使用的哪种消息发送模式？你们平时使用的是哪种消息消费模式？为什么？

​	几种消息发送模式下，在什么场景应该选用什么消息发送模式？几种消息消费模式下，在什么场景下应该选用什么消息消费模式？

## 5.3 基于MQ实现订单数据同步给大数据团队

**问题：**订单系统应该如何将订单数据发送到RocketMQ里去呢？

**方案1：**

​	比较简单的办法，就是在订单系统中但凡对订单执行增删改类的操作，就把这种对订单增删改的操作发送到RocketMQ里去。

​	但是这种方案的一个问题就是订单系统为了将数据同步给大数据团队，必须在自己的代码里耦合大量的代码去发送增删改操作到RocketMQ，这会导致订单系统的代码出现严重的污染，因为这些发送增删改操作到RocketMQ里的代码是跟订单业务没关系的。

**方案2：**

​	**就是用Canal、Databus这样的MySQL Binlog同步系统，监听订单数据库的binlog发送到RocketMQ里**

​	然后大数据团队的数据同步系统从RocketMQ里获取订单数据的增删改binlog日志，还原到自己的数据存储中去，可以是自己的数据库，或者是Hadoop之类的大数据生态技术。

​	然后大数据团队将完整的订单数据还原到自己的数据存储中，就可以根据自己的技术能力去出数据报表了，不会再影响订单系统的数据库了。

**Canal：**译意为水道/管道/沟渠，主要用途是基于 **MySQL 数据库增量日志解析**，提供**增量数据订阅和消费**。

​		https://blog.csdn.net/yehongzhi1994/article/details/107880162

**Databus：**是一个实时的低延时数据抓取系统。它将数据库作为唯一真实数据来源，并将变更从事务或提交日志中提取出来，然后通知相关的衍生数据库或缓存。

​		https://blog.csdn.net/acm_lkl/article/details/78645406

**问题：**是否有可能有其他团队需要从你这里获取数据？如果要从你这里获取数据，你又应该如何设计数据同步方案？

## 5.4 秒杀系统的技术难点以及秒杀商品详情页系统的架构设计

**PS:类似亿级流量的模式**

**第一个问题**，秒杀活动目前压力过大，应该如何解决？是不是简单的堆机器或者加机器就可以解决的？

**第二个问题**，那么数据库呢？是不是也要部署更多的服务器，进行分库分表，然后让更多的数据库服务器来抗超高的数据库高并发访问？

​	如果用堆机器的方法来解决这个问题，必然存在一个问题，就是随着你的用户量越来越大，你的并发请求越来越多，会导致你要不停的增加更多的机器，所以解决问题往往不能用这种简单粗暴堆机器的方案！

**方案：**

​	秒杀活动主要涉及到的并发压力就是两块，**一个是高并发的读，一个是高并发的写。**

​	1 **页面数据静态化+多级缓存**

​	2 多级缓存的架构，我们会使用**CDN + Nginx + Redis**的多级缓存架构

![image-20220125172735742](中间件专栏(RockerMq).assets/image-20220125172735742.png)

问题：

​	**你们有没有类似秒杀的业务场景？如果没有，自己想一个出来！例子，比如说一个考勤系统，可能每天就是早上上班时间并发访问量特别的大，或者是工资系统，每个月到发工资的时候公司发短信提醒了员工，马上大量的人登录上来进行查询。**

## 5.5 基于MQ实现秒杀订单系统的异步化架构以及精准扣减库存的技术方案



1. 在前端/客户端设置秒杀答题，错开大量人下单的时间，阻止作弊器刷单
2. 独立出来一套秒杀系统，专门负责处理秒杀请求
3. 优先基于Redis进行高并发的库存扣减，一旦库存扣完则秒杀结束
4. 秒杀结束之后，Nginx层过滤掉无效的请求，大幅度削减转发到后端的流量
5. 瞬时生成的大量下单请求直接进入RocketMQ进行削峰，订单系统慢慢拉取消息完成下单操作



对于瞬时超高并发抢购商品的场景，**首先必须要避免直接基于数据库进行高并发的库存扣减**，因为那样会对库存数据库造成过大的压力

问题：那么在你的系统中如果要应对瞬时超高并发，应该怎么处理？应该如何设计架构来抗下瞬时超高的并发？

## 5.6 MQ的订单系统架构阶段总结

![image-20220209165145721](中间件专栏(RockerMq).assets/image-20220209165145721.png)

# 6 RocketMq核心原理

## 6.1 深入研究一下生产者到底如何发送消息的

**MessageQueue**

​	**在创建Topic的时候需要指定一个很关键的参数，就是MessageQueue**。

​	RocketMQ引入了MessageQueue的概念，本质上就是一个数据分片的机制。

​	假设你的Topic有1万条数据，然后你的Topic有4个MessageQueue，那么大致可以认为会在每个MessageQueue中放入2500条数据

​	当然，这个不是绝对的，有可能有的MessageQueue的数据多，有的数据少，这个要根据你的消息写入MessageQueue的策略来定。

​	可能就是在2个Broker上，每个Broker放两个MessageQueue

​	总结：**MessageQueue就是RocketMQ中非常关键的一个数据分片机制，他通过MessageQueue将一个Topic的数据拆分为了很多个数据分片，然后在每个Broker机器上都存储一些MessageQueue**。通过这个方法，就可以实现Topic数据的分布式存储

![image-20220210172949270](中间件专栏(RockerMq).assets/image-20220210172949270.png)

**sendLatencyFaultEnable**

​	一旦打开了这个开关，那么他会有一个**自动容错机制**，比如如果某次访问一个Broker发现网络延迟有500ms，然后还**无法访问**，那么就会**自动回避访问这个Broker一段时间**，比如接下来3000ms内，就不会访问这个Broker了。

​	这样的话，就可以避免一个Broker故障之后，短时间内生产者频繁的发送消息到这个故障的Broker上去，出现较多次数的异常。而是在一个Broker故障之后，自动回避一段时间不要访问这个Broker，过段时间再去访问他。

​	那么这样过一段时间之后，可能这个Master Broker就已经恢复好了，比如他的Slave Broker切换为了Master可以让别人访问了。

问题：

- Kafka、RabbitMQ他们有类似的数据分片机制吗？
- 他们是如何把一个逻辑上的数据集合概念（比如一个Topic）给在物理上拆分为多个数据分片的？
- 拆分后的多个数据分片又是如何在物理的多台机器上分布式存储的？
- 为什么一定要让MQ实现数据分片的机制？
- 如果不实现数据分片机制，让你来设计MQ中一个数据集合的分布式存储，你觉得好设计吗？

## 6.2 深入研究一下Broker是如何持久化存储消息的

​	**Broker数据存储实际上才是一个MQ最核心的环节**，他决定了生产者消息写入的吞吐量，决定了消息不能丢失，决定了消费者获取消息的吞吐量，这些都是由他决定的。

**CommitLog消息顺序写入机制**

​	**生产者的消息发送到一个Broker**上的时候，他会把这个**消息直接写入磁盘上的一个日志文件，叫做CommitLog，直接顺序写入这个**文件

​	这个CommitLog是很多磁盘文件，每个文件限定最多1GB，Broker收到消息之后就直接追加写入这个文件的末尾，就跟上面的图里一样。如果一个CommitLog写满了1GB，就会创建一个新的CommitLog文件

**MessageQueue在数据存储**

​	ConsumeQueue中存储的每条数据不只是消息在CommitLog中的offset偏移量，还包含了消息的长度，以及tag hashcode，一条数据是20个字节，每个ConsumeQueue文件保存30万条数据，大概每个文件是5.72MB。

​	所以实际上**Topic的每个MessageQueue都对应了Broker机器上的多个ConsumeQueue文件，保存了这个MessageQueue的所有消息在CommitLog文件中的物理位置，也就是offset偏移量**。

![image-20220211171407712](中间件专栏(RockerMq).assets/image-20220211171407712.png)

**同步刷盘与异步刷盘**

​	**异步刷盘：**

​	Broker是基于OS操作系统的**PageCache**和**顺序写**两个机制，来提升写入CommitLog文件的性能的

​	数据写入CommitLog文件的时候，其实不是直接写入底层的物理磁盘文件的，而是先进入OS的PageCache内存缓存中，然后后续由OS的后台线程选一个时间，异步化的将OS PageCache内存缓冲中的数据刷入底层的磁盘文件

​	用**磁盘文件顺序写+OS PageCache写入+OS异步刷盘的策略**，基本上可以让消息写入CommitLog的性能跟你直接写入内存里是差不多的，所以正是如此，才可以让Broker高吞吐的处理每秒大量的消息写入

​	缺点：如果生产者认为消息写入成功了，但是实际上那条消息此时是在Broker机器上的os cache中的，如果此时Broker直接宕机，那么是不是os cache中的这条数据就会丢失了

​	**同步刷盘：**

​	使用同步刷盘模式的话，那么生产者发送一条消息出去，broker收到了消息，必须直接强制把这个消息刷入底层的物理磁盘文件中，然后才会返回ack给producer，此时你才知道消息写入成功了

​	优缺点：如果你强制每次消息写入都要直接进入磁盘中，必然导致每条消息写入性能急剧下降，导致消息写入吞吐量急剧下降，但是可以保证数据不会丢失

问题：

​		**同步刷盘和异步刷盘两种策略，分别适用于什么不同的场景呢？**

## 6.3 基于DLedger技术的Broker主从同步原理

**基于DLedger技术替换Broker的CommitLog**

​	基于DLedger技术来实现Broker高可用架构，实际上就是用DLedger先替换掉原来Broker自己管理的CommitLog，由DLedger来管理CommitLog

​	然后Broker还是可以基于DLedger管理的CommitLog去构建出来机器上的各个ConsumeQueue磁盘文件

**Raft协议选举Leader Broker**

​	发起多伦的投票，通过三台机器互相投票选出来一个人作为Leader

​	**确保有人可以成为Leader的核心机制就是一轮选举不出来Leader的话，就让大家随机休眠一下，先苏醒过来的人会投票给自己，其他人苏醒过后发现自己收到选票了，就会直接投票给那个人**

**基于Raft协议进行多副本同步**

​	**数据同步会分为两个阶段，一个是uncommitted阶段，一个是commited阶段**

​	Leader Broker上的DLedger收到一条数据之后，会标记为uncommitted状态，然后他会通过自己的DLedgerServer组件把这个uncommitted数据发送给Follower Broker的DLedgerServer。

​	接着Follower Broker的DLedgerServer收到uncommitted消息之后，必须返回一个ack给Leader Broker的DLedgerServer，然后如果Leader Broker收到超过半数的Follower Broker返回ack之后，就会将消息标记为committed状态。

​	然后Leader Broker上的DLedgerServer就会发送commited消息给Follower Broker机器的DLedgerServer，让他们也把消息标记为comitted状态

**PS:如果Leader Broker挂了，此时剩下的两个Follower Broker就会重新发起选举，他们会基于DLedger还是采用Raft协议的算法，去选举出来一个新的Leader Broker继续对外提供服务，而且会对没有完成的数据同步进行一些恢复性的操作，保证数据不会丢失。**

![image-20220211175444261](中间件专栏(RockerMq).assets/image-20220211175444261.png)

问题：**采用Raft协议进行主从数据同步，会影响TPS吗？**

问题概述：

​	在Leader接收消息写入的时候，基于DLedger技术写入本地CommitLog中，这个其实跟之前让Broker自己直接写入CommitLog是没什么区别的。

​	但是有区别的一点在于，Leader Broker上的DLedger在收到一个消息，将uncommitted消息写入自己本地存储之后，还需要基于Raft协议的算法，去采用两阶段的方式把uncommitted消息同步给其他Follower Broker

## 6.4 深入研究一下消费者是如何获取消息处理以及进行ACK的









