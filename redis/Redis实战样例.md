1 Redis三个客户端框架比较

**概念：**

　　Jedis：是Redis的Java实现客户端，提供了比较全面的Redis命令的支持，

　　Redisson：实现了分布式和可扩展的Java数据结构。

　　*Lettuce：高级Redis客户端，用于线程安全同步，异步和响应使用，支持集群，Sentinel，管道和编码器。*

***优点：***

　　*Jedis：比较全面的提供了Redis的操作特性*

　　*Redisson：促使使用者对Redis的关注分离，提供很多分布式相关操作服务，例如，分布式锁，分布式集合，可通过Redis支持延迟队列*

　　*Lettuce：主要在一些分布式缓存框架上使用比较多*

*可伸缩：*

Jedis：使用阻塞的I/O，且其方法调用都是同步的，程序流需要等到sockets处理完I/O才能执行，不支持异步。Jedis客户端实例不是线程安全的，所以需要通过连接池来使用Jedis。

Redisson：基于Netty框架的事件驱动的通信层，其方法调用是异步的。Redisson的API是线程安全的，所以可以操作单个Redisson连接来完成各种操作

*Lettuce：基于Netty框架的事件驱动的通信层，其方法调用是异步的。Lettuce的API是线程安全的，所以可以操作单个Lettuce连接来完成各种操作*

引用：https://www.cnblogs.com/liyan492/p/9858548.html

# 2 最简单的分布式锁/nx

​	企业级分布式锁，基于redis的分布式锁的企业级实现，是通过redisson框架，基于复杂的lua 脚本去实现的，有一整套的实现锁的逻辑在里面。

​	set key value nx，必须是这个key此时是不存在的，才能设置成功，如果说key要是存在了,此时设置是失败的

​	比如说有很多台机器，要竞争一把锁,此时你可以让他们对同一个锁的key,比如 lock_key,设置一个value，而且都是要加上nx选项的，此时redis可以通过nx选项，保证说只有一台机器可以设置成功，可以加锁成功,其他机器都是会失败的

```java
	public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1");
        setNxTest(jedis);
    }
	/**
     * 最简单的分布式锁样例
     * @param jedis
     */
    private static void  setNxTest(Jedis jedis){
        //还原锁
        jedis.del("lock_test");
        //第一次加锁
        jedis.set("lock_test","value1", SetParams.setParams().nx());
        System.out.println("第一次加锁：结果"+jedis.get("lock_test"));
        //第二次加锁,已存在的key进行 set nx操作会返回失败状态，可以利用这个特性实现分布式锁
        jedis.set("lock_test","value1", SetParams.setParams().nx());
        System.out.println("第二次加锁：结果"+jedis.get("lock_test"));
    }
```



# 3 博客网站的文章发布、查看、点赞、统计等/批量操作key

3.1 批量操作

mset,mget,msetnx 为批量操作key，m->multi的意思

mset:一次set多个key

mget:一次get多个key

msetnx:一次set多个key，其中有一个存在则失败

利用mset,mget,msetnx 进行批量操作比单个操作key减少了网络通信时间，提高操作效率

3.2 统计，截取内容

strlen:返回字数

getrange:截取字符(根据字节数截取的一个中文2字节)















