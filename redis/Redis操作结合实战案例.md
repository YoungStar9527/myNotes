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

## 3.5 社交网站的网址点击追踪机制/hash相关命令

​	hset,hget,hincrBy等

​	完整代码见 com.star.jvm.demo.redis.hash.ShortUrlDemo

​	社交网站（微博）一般会把你发表的一些微博里的长连接转换为短连接，这样可以利用短连接进行点击数量追踪，然后再让你进入短连接对应的长连接地址里去，所以可以利用hash数据结构去实现网址点击追踪机制

**为什么使用短连接来实现点击追踪**

​	1 原始url链接可能过长，短连接比较精简

​	2 某个用户发表的微博中的链接使用短连接追踪可靠性更高（因为不同的用户发表的微博的链接地址可能相同，而短连接生成是唯一的）

​	3 原始url连接字符比较复杂，各种不同情况的字符都可能会有，作为key不太靠谱，而使用短连接作为key相对可靠很多

使用

​	**核心代码**

```java
private static final String[] X36_ARRAY = "0,1,2,3,4,5,6,7,8,9,A,B,C,D,E,F,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,W,X,Y,Z".split(",");   
	/**
     * 获取短连接网址
     * 每次调用都生成新的短连接地址，新的短连接地址和参数url绑定
     * url地址和短连接生成没有关系，同一个url重复调用该方法，也会生成新的短连接，造成不同的短连接指向同一个url
     * @param url
     * @return
     */
    public String getShortUrl(String url) {
        long shortUrlSeed = jedis.incr("short_url_seed");

        StringBuffer buffer = new StringBuffer();
        //利用redis的incr自增长，然后10进制转36进制，接着hset存放在hash数据结构里，再提供一个映射转换的hget获取方法
        while(shortUrlSeed > 0) {
            buffer.append(X36_ARRAY[(int)(shortUrlSeed % 36)]);
            shortUrlSeed = shortUrlSeed / 36;
        }
        String shortUrl = buffer.reverse().toString();
        //重置对应key的访问次数
        jedis.hset("short_url_access_count", shortUrl, "0");
        //关联短连接和url,每次新生成的短连接关联url
        //放在url_mapping中的hash结构内，url_mapping对应的数据相当于java的一个HashMap实例
        jedis.hset("url_mapping", shortUrl, url);
        return shortUrl;
    }
	
    /**
     * 给短连接地址进行访问次数的增长
     * 每调用一次+1
     * @param shortUrl
     */
    public void incrementShortUrlAccessCount(String shortUrl) {
        //value设置为多少每次调用就增加多少
        jedis.hincrBy("short_url_access_count", shortUrl, 1);
    }
```

## 3.6 博客代码重构/批量操作/hash操作

​	发布/修改/查看/查看次数/点赞次数 等操作提取方法重构

​	利用mset,mget等批量操作或者,hset,hget等hash操作都可以实现代码重构

​	完整代码见 com.star.jvm.demo.redis.hash.BlogDemo

```java
	/**
     * 发表一篇博客
     */
    public boolean publishBlog(long id, Map<String, String> blog) {
        //hexists判断key中的键是否存在，存在title说明该博客已经发表过
        if(jedis.hexists("article::" + id, "title")) {
            return false;
        }
        blog.put("content_length", String.valueOf(blog.get("content").length()));

        jedis.hmset("article::" + id, blog);

        return true;
    }

    /**
     * 查看一篇博客
     * @param id
     * @return
     */
    public Map<String, String> findBlogById(long id) {
        //hgetAll一次性拿出对应的map结构数据
        Map<String, String> blog = jedis.hgetAll("article::" + id);
        incrementBlogViewCount(id);
        return blog;
    }
    /**
     * 增加博客浏览次数
     * @param id
     */
    public void incrementBlogViewCount(long id) {
        jedis.hincrBy("article::" + id, "view_count", 1);
    }
```

hash的数据结构比较适合放java对象，hash键值对对应java对象中属性和值

getrange，setrange，append等，对字符串的复杂操作hash是不支持的

**PS:hash不可以设置hashkey的过期时间**

## 3.7 基于令牌的登录会话机制/hash操作

​	用户每次登陆都需要持有令牌token，通过redis验证token是否存在等，如果token存在则通过登录。	通过hash记录所有用户的登录token，并在对应hash属性中设置超时时间。

完整代码见 com.star.jvm.demo.redis.hash.SessionDemo

```java
    /**
     * 模拟的登录方法
     * @param username
     * @param password
     * @return
     */
    public String login(String username, String password) {
        // 基于用户名和密码去登录
        System.out.println("基于用户名和密码登录：" + username + ", " + password);
        Random random = new Random();
        long userId = random.nextInt() * 100;
        // 登录成功之后，生成一块令牌
        String token = UUID.randomUUID().toString().replace("-", "");
        // 基于令牌和用户id去初始化用户的session
        initSession(userId, token);
        // 返回这个令牌给用户
        return token;
    }

    /**
     * 用户登录成功之后，初始化一个session
     * @param userId
     * @param token
     */
    public void initSession(long userId, String token) {
        SimpleDateFormat dateFormat = new SimpleDateFormat(
                "yyyy-MM-dd HH:mm:ss");

        Calendar calendar = Calendar.getInstance();
        calendar.setTime(new Date());
        calendar.add(Calendar.HOUR, 24);
        Date expireTime = calendar.getTime();

        jedis.hset("sessions",
                "session::" + token, String.valueOf(userId));
        jedis.hset("sessions::expire_time",
                "session::" + token, dateFormat.format(expireTime));
    }
```



## 3.8 秒杀活动下的公平队列抢购机制/list操作

​	秒杀系统有很多实现方案，其中有一种技术方案，就是对所有涌入系统的秒杀抢购请求，都放入redis的一个list数据结构里去，进行公平队列排队，然后入队之后就等待秒杀结果，专门搞一个消费者从list里按顺序获取抢购请求，按顺序进行库存扣减，扣减成功了就让你抢购成功

​	对于抢购请求入队列，就用**lpush** list request就可以了，然后对于出队列进行抢购，就用**rpop** list就可以了，**lpush**就是左边推入，**rpush**就是右边推入，**lpop**就是左边弹出，rpop就是右边弹出

​	所以你**lpush+rpop**，就是做了一个左边推入和右边弹出的先入先出的公平队列

​	完整代码见 com.star.jvm.demo.redis.list.SecKillDemo

```java
    /**
     * 秒杀抢购请求入队
     * @param secKillRequest
     */
    public void enqueueSecKillRequest(String secKillRequest) {
        jedis.lpush("sec_kill_request_queue", secKillRequest);
    }

    /**
     * 秒杀抢购请求出队
     * @return
     */
    public String dequeueSecKillRequest() {
        return jedis.rpop("sec_kill_request_queue");
    }

    public static void main(String[] args) throws Exception {
        SecKillDemo demo = new SecKillDemo();

        for(int i = 0; i < 10; i++) {
            demo.enqueueSecKillRequest("第" + (i + 1) + "个秒杀请求");
        }

        while(true) {
            String secKillRequest = demo.dequeueSecKillRequest();

            if(secKillRequest == null
                    || "null".equals(secKillRequest)
                    || "".equals(secKillRequest)) {
                break;
            }

            System.out.println(secKillRequest);
        }
    }
```

## 3.9 OA系统中代办事项列表管理

​	OA系统，自动化办公系统，说白了就是把企业日常运行的办公的日常事务都在OA系统里来做，请假，审批，开会，项目，任务，待办事项列表

​	lindex，lset，linsert，ltrim，lrem

​	插入待办事项，linsert list index event(根据position策略插入)

​	查询待办事项列表，lrange list 0 -1，所有都查询出来(下标范围查询)

​	完成待办事项，lrem list 0 event，就把这个待办事项给删了(根据value是否相同删除)

​	批量完成待办事项，ltrim list start_index end_index()(保留下标范围的value)

​	修改待办事项，lindex和lset(lindex获取下标，lset根据下标修改)

```java
    /**
     * 分页查询待办事项列表
     * @param userId
     * @param pageNo
     * @param pageSize
     * @return
     */
    public List<String> findTodoEventByPage(long userId, int pageNo, int pageSize) {
        int startIndex = (pageNo - 1) * pageSize;
        int endIndex = pageNo * pageSize - 1;
        return jedis.lrange("todo_event::" + userId, startIndex, endIndex);
    }

    /**
     * 插入待办事项
     */
    public void insertTodoEvent(long userId,
                                ListPosition position,
                                String targetTodoEvent,
                                String todoEvent) {
        jedis.linsert("todo_event::" + userId, position, targetTodoEvent, todoEvent);
    }

    /**
     * 修改一个待办事项
     * @param userId
     * @param index
     * @param updatedTodoEvent
     */
    public void updateTodoEvent(long userId, int index, String updatedTodoEvent) {
        jedis.lset("todo_event::" + userId, index, updatedTodoEvent);
    }

    /**
     * 完成一个待办事项
     * @param userId
     * @param todoEvent
     */
    public void finishTodoEvent(long userId, String todoEvent) {
        jedis.lrem("todo_event::" + userId, 0, todoEvent);
```



## 3.10 网站注册邮件发送机制/list-brpop操作

​	网站注册成功后一般会发送一份验证邮件到邮箱，但是邮件发送是比较慢的，这个时候就利用队列，将邮件发送任务放到队列去，让邮件发送系统从队列中获取任务来发送邮件

​	**利用brpop阻塞式获取队列中的任务**

完整代码见 com.star.jvm.demo.redis.list.SendMailDemo

```java
    /**
     * 让发送邮件任务入队列
     * @param sendMailTask
     */
    public void enqueueSendMailTask(String sendMailTask) {
        jedis.lpush("send_mail_task_queue", sendMailTask);
    }

    /**
     * 阻塞式获取发送邮件任务
     * @return
     */
    public List<String> takeSendMailTask() {
        return jedis.brpop(5, "send_mail_task_queue");
    }
```



## 3.11 网站每日UV数据指标去重统计/set集合操作































