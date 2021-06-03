# 1 Redis三个客户端框架比较

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

​	**set key value nx**，必须是这个key此时是不存在的，才能设置成功，如果说key要是存在了,此时设置是失败的

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

## 3.1 批量操作

mset,mget,msetnx 为批量操作key，m->multi的意思

**mset:一次set多个key**

**mget:一次get多个key**

**msetnx:一次set多个key，其中有一个存在则失败**

利用mset,mget,msetnx 进行批量操作比单个操作key减少了网络通信时间，提高操作效率

```java
    /**
     * 博客发布/查看/点赞
     * mget mset msetnx 批量操作key
     * m就是 multi
     * 如果是redis Cluster key值需要加大括号{}，会将大括号内的key放在同一个slot内，否则批量操作会因为不同key在slot内而失败
     * 单机redis则无需加大括号{}
     * @param jedisCluster
     */
    private static void mkeyTest(JedisCluster jedisCluster){
        String mset = jedisCluster.mset("{blog}:author1", "xjx", "{blog}:content1", "沙雕一号", "{blog}:title1", "略略略略略");
        System.out.println("博客发布结果："+mset);
        List<String> mget = jedisCluster.mget("{blog}:author1", "{blog}:content1", "{blog}:title1");
        System.out.println("博客查看结果："+mget);
        Long msetnx = jedisCluster.msetnx(
                "{blogauthor1}11", "xjx",
                "{blogauthor1}22", "沙雕二号",
                "{blogauthor1}33", "略略略略略",
                "{blogauthor1}44", "123");
        System.out.println("msetnx结果："+msetnx);
    }
```



## 3.2 统计，截取内容

**strlen:返回字数(根据字节数计算的一个中文2字节)**

**getrange:截取字符(根据字节数截取的一个中文2字节)**

```java
    /**
     * 统计长度或字数
     * strlen 统计
     * getrange
     * @param jedisCluster
     */
    private static void countAndLenthTest(JedisCluster jedisCluster){
        //按照字节数统计，中文两个字节，英文数字一个字节
        jedisCluster.set("strContent","测试test123");
        Long strlen = jedisCluster.strlen("strContent");
        System.out.println("博客正文统计："+strlen);
        //按照字节数截取，中文两个字节，英文数字一个字节
        String getrange = jedisCluster.getrange("strContent", 0, 5);
        System.out.println("博客摘要结果："+getrange);
    }
```



## 3.3 点赞，唯一ID生成器

​	ID，都是通过数据库的自增主键来实现的，有一些场景下，分库分表的时候，你同一个表里的数据都会分散在多个库的多个表里，这个时候就不能光是靠数据库来生成自增长的主键了，此时就需要生成唯一ID

​	snowflake算法，还有一些大厂开源的唯一ID生成组件，他是会解决一些原生的snowflake算的缺陷

​	**incr:每调用一次value值就+1(redis实现唯一id或者点赞等功能)**

​	**decr:每调用一次value值就-1(redis实现取消点赞等功能)**

```java
    /**
     * 点赞/唯一ID生成
     * incr每调用一次value值就+1
     * @param jedisCluster
     */
    private static void incrTest(JedisCluster jedisCluster){
        jedisCluster.del("idKey1");
        for(int i = 0; i < 10; i++){
            Long idKey1 = jedisCluster.incr("idKey1");
            System.out.println("循环次数"+(i+1)+"，id为"+idKey1);
        }
    }
```



## 3.4 用户操作日志审计/拼接value

​	用户每天的日志可以放在一个key里面，每次新的日志都拼接到当天的key内容中，可以通过redis来查看按天查询日志

​	**append:拼接value到之前的key里面**

```java
    /**
     * 用户操作日志审计
     * append 每次调用都将value拼接
     * @param jedisCluster
     */
    private static void appendTest(JedisCluster jedisCluster){
        jedisCluster.set("appContent","");
        //用append加入未存在的key和set是一个效果，但是已存在的key，set是覆盖,append是拼接
        //jedisCluster.append("appContent1111","");
        for(int i = 0; i < 10; i++){
            jedisCluster.append("appContent","用户操作第"+(i+1)+"次,");
            System.out.println(jedisCluster.get("appContent"));
        }
    }
```

