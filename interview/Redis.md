### 面试——Redis 系列
#### Redis 介绍
>Redis 是一个分布式缓存系统，具有**内存缓存**和**持久化**双重功能，可以作为 nosql 数据库使用。
#### 为什么使用 Redis
> * 完全基于内存，非常迅速。
> * 数据结构多，数据操作简单。
> * 采用单线程，避免了不必要的上下文切换和资源竞争，也不存在多线程的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗。
> * 使用多路复用 I/O 模型，非阻塞 IO（NIO）。
#### Redis 数据结构
* 基础数据结构

Redis的基础数据结构有五种：String，Hash， List，Set，SortedSet。

高级：Bitmap（用来实现布隆过滤器）, pub/sub（订阅发布功能，用来做消息队列）

下面我将演示这五种基础数据结构：
首先通过 redis-server 启动 redis，然后通过 redis-cli 启动 redis 客户端，打开客户端之后执行 **ping** 指令，检查 redis 是否存活。

* 1. String
      String 存储的命令 set key value EX,  EX（过期时间,单位为秒）也可以不设置，
      如下：

    ![avatar](http://qdyf5dtd9.bkt.clouddn.com/image/blog/redis/string.png )
* 2. Hash
      Hash 存储的命令 hmset key field value [field value ...] EX
      一般可以用来存储 Java 对象，例如 Java 应用中的用户信息：

    ![avatar](http://qdyf5dtd9.bkt.clouddn.com/image/blog/redis/hash.png)
* 3. List（有序）
List 为有序列表，List 存储的命令 lpush key value [ value ...] EX
其中：key 为列表的名称。
![avatar](http://qdyf5dtd9.bkt.clouddn.com/image/blog/redis/list.png)
* 4. Set（无序）
Set 为无序且不重复的数据，在实际开发中可应用于去重，后者会覆盖前者。
命令：sadd key member [member ...]
![avatar](http://qdyf5dtd9.bkt.clouddn.com/image/blog/redis/set.png)
* 5. SortedSet（有序）
命令：zadd key score member [score member ...]
score 为整型数字，根据score来排序，如下：
![avatar](http://qdyf5dtd9.bkt.clouddn.com/image/blog/redis/sortedset.png)
>在实际工作中，我们可以利用 sortedset 实现延时队列，拿时间戳作为 score ，消息内容作为 key 调用 zadd 指令来生产消息，消费者用 zrangebyscore 指令获取 N 秒之前的数据轮询进行处理。

***注：删除指令：del key***
***关于设置 key 的有效时间存在的问题，在 key 的有效时间设置的过于集中，到了过期的时间点， Redis 可能会出现短暂的卡顿现象，严重的话会出现缓存雪崩，我们一般会在时间上加一个随机值，使得失效时间分散，避免出现缓存雪崩。***

#### Redis 事务
>Redis 的事务具有隔离性和原子性
* 1. 隔离性：一个事务在执行的过程中不会被其他请求打断执行
* 2. 原子性：要么全部成功，要么全部失败（跟关系型数据的事务原子性有一定理解上的差异。Redis的事务不存在回滚机制，所以这句话跟传统意义的效果有一定的差异）。
>说明：Redis事务是没有事务回滚概念。因为Redis语法简单，不会出现运行异常。即使出现语法异常是不会导致事务回滚。

#### Redis 安全配置
```
config set requirepass 123456 //配置redis的密码
config get requirepass //查看密码
验证密码：auth 123456
带密码登录：./redis-cli -h 127.0.0.1 -p 6379  -a 123456   // -h表示主机IP，-p端口，-a密码
```

#### Redis 持久化策略
>Redis 的持久化策略分为两种 AOF（增量持久化）和 RDB（全量持久化）。
* 1. AOF
>AOF 默认每秒进行数据同步持久化策略，即使数据丢失，也仅仅只丢失1秒钟的数据。
```
redis中appendonly的配置是用来开启AOF存储策略的。yes为开启，no为关闭。
125) "appendonly" 126) "no"
此配置每秒进行持久化操作
121) "appendfsync" 122) "everysec"
```
* 2. RDB
>RDB 会每隔一段时间中对 Redis 服务中当下的数据集进行快照。RDB 在 Redis 中默认了三种数据存储的条件，如下：

时间（秒）|修改次数
---|:--
900|1
300|10
60|10000
当900s内至少1次修改就会触发 Redis 的持久化将数据写入到 RDB 的持久化文件中；

当300s内至少10次修改就会触发 Redis 的持久化将数据写入到 RDB 的持久化文件中；

当60s内至少10000次修改就会触发 Redis 的持久化将数据写入到 RDB 的持久化文件中；
>在redis的配置中save属性表示的就是RDB持久化策略。

`指令"save" 130) "900 1 300 10 60 10000"`

注：如果允许一定的数据丢失，建议只使用RDB，因为性能高一些。
如果能接受数据丢失的量比较小，建议使用AOF。
如果不接受数据的丢失，建议是使用AOF+RDB方式

#### Redis 持久化的相关问题
* 1. 如何保证 Redis 中数据有的完整性？
>首先 RDB 是做镜像持久化，AOP 是做增量持久化。因为 RDB 会比较耗时，性能比较低，不够实时，在 Redis 宕机的时候会丢失大量的数据，这时候就需要配合 AOF 来做持久化操作。在 Redis 示例重启的时候，首先会使用 RDB 的持久化文件重新构建内存，再使用 AOF 重放近期的操作指令来实现完整恢复重启之前的状态。在这里我们可以把 RDB 理解为一个表的全量数据，把 AOF 理解为 每次操作的日志。服务器重启的时候，先把整张表的数据导入内存中，当然数据也可以不完整，这时候再回放 AOF 中近期的操作日志，这下子数据不就完整了嘛。这里需要注意的是当 RDB 或者 AOF 文件存在错误时，Redis 会启动失败。
* 2. 如果工作中突然断电怎么办？
>这个问题就很好回答了，在前面 AOF 中我们提到了可以开启 AOF 每秒中进行持久化操作，这里的操作是 sync 的，就算断电了也只会丢失这一秒钟的数据。
* 3. 既然你提到了 AOF 是 sync 的原理，那请你谈谈 RBD 的原理？
>在 RDB 中有两个很重要的词汇 **fork** 和 **cow**;
fork 指的是 Redis 通过创建子进程来进行 RDB 操作；cow 指的是 copy-on-write，子进程创建之后，子进程能够获得和父进程完全相同的内存空间，父进程对内存的修改对于子进程是不可见的，两者不会相互影响；通过 fork 创建子进程时不会立刻触发大量内存的拷贝，内存在被修改时会以页为单位进行拷贝，这也就避免了大量拷贝内存而带来的性能问题；

#### Redis 分布式锁
* 单机Redis实现分布式锁

通过 `set resource_name my_random_value NX PX 30000` 来获取锁，
my_random_value 是由客户端生成的一个随机字符串。
NX 表示只有当 resource_name 对应的 key 值 不存在的时候才能 SET 成功。这保证了只有第一个请求的客户端才能获取锁，而其他的客户端在锁没有被释放之前都无法获得锁。
PX 30000 表示这个锁有个30秒的过期时间。
也可以通过 setnx 指令，然后再给这个 key 加上 expire 即可，但是这种方式存在风险，在setnx之后执行expire之前进程意外crash或者要重启维护了，这个锁就永远得不到释放了。

### 缓存雪崩，穿透，击穿
#### 雪崩

> 所谓的缓存雪崩，我举个栗子能有助于理解：对于频繁访问的资源，我们不会直接访问数据库，这样会给数据库带来很大的压力导致数据库直接挂掉，对于 Redis 中的数据大部分我们都会设置 EX（失效时间），当缓存的过期时间过于集中就会造成大量失效，这时候访问的流量就会直接给到数据库，这时候数据库肯定顶不住直接挂掉，我们重启数据库由于缓存已经失效了，流量还是会访问到数据库，数据库还是会一直挂掉，反复横跳，为了保证这种情况的发生，我们在**设置缓存有效时间时尽量分散**。

#### 穿透

>可以理解为漏网之鱼，假设 我们设计了一个API供第三方调用，参数为整型，我们将 序列id为 0 - 100 数据放在缓存中，这时候有个用户为了考验API的稳定性，在给定的参数范围外进行查询，例如 -1，这时候 redis 中肯定是没有数据的就会访问数据库，假设这是个暴力访问呢，我们的数据库是不是就垮掉了！为了避免这种情况，我一定要对访问的参数进行**校验**。
##### 补充：Redis 避免缓存穿透 —— BloomFilter 

>BloomFilter 
>
>此处为借鉴，没有做任何修改，资料来源为 掘金敖丙——[Redis-避免缓存穿透的利器之 BloomFilter](https://juejin.im/user/59b416065188257e671b670a/posts)
>
>
```
 <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>23.0</version>
 </dependency>    
/**
 * 测试布隆过滤器(可用于redis缓存穿透)
 * 
 * @author 敖丙
 */
public class TestBloomFilter {

    private static int total = 1000000;
    private static BloomFilter<Integer> bf = BloomFilter.create(Funnels.integerFunnel(), total);
//    private static BloomFilter<Integer> bf = BloomFilter.create(Funnels.integerFunnel(), total, 0.001);

    public static void main(String[] args) {
        // 初始化1000000条数据到过滤器中
        for (int i = 0; i < total; i++) {
            bf.put(i);
        }

        // 匹配已在过滤器中的值，是否有匹配不上的
        for (int i = 0; i < total; i++) {
            if (!bf.mightContain(i)) {
                System.out.println("有坏人逃脱了~~~");
            }
        }

        // 匹配不在过滤器中的10000个值，有多少匹配出来
        int count = 0;
        for (int i = total; i < total + 10000; i++) {
            if (bf.mightContain(i)) {
                count++;
            }
        }
        System.out.println("误伤的数量：" + count);
    }

}

```

#### 击穿

> 缓存击穿就很好理解了，对着一个点精准打击，比如某博，对吧，热点话题，这时候突然爆出某个八卦的话题（例如某娟找男朋友），导致七大姑八大姨，街坊邻居等等访问的流量呈现指数式增长，恰好这个 key 失效了，导致大量的流量直接请求数据库了，所有某博经常挂嘛，我们都是将热点数据设置为**永久不失效** ！

### Redis 线程模型

Redis 内部使用的文件事件处理器 `file event handler ` ,这个文件事件处理器是单线程的，所以 Redis 才叫单线程模型。这个文件处理器的结构分为4个部分：多个 socket， I/O多路复用程序，文件事件分派器，事件处理器。

多个 **Socket** 可能会并发产生不同的操作，每个操作对应不同的文件事件处理器，但是 I/O 多路复用程序会监视多个 **Socket** ，会将多个 **Socket** 产生的事件放到队列中，文件事件分派器每次会取出一个事件分派给对应的事件处理器进行处理。

### Docker-Redis

```sh
 docker run -d --privileged=true -p 6379:6379 --restart always -v /Users/apple/Volumes/redis/conf:/etc/redis/redis.conf  -v /Users/apple/Volumes/redis/data:/data --name myredis redis redis-server /etc/redis/redis.conf --requirepass "123456" --appendonly yes
```

#### 参数解释

```
1-d                                                  -> 以守护进程的方式启动容器
2-p 6379:6379                                        -> 绑定宿主机端口:容器端口
3--name myredis                                      -> 指定容器名称
4--restart always                                    -> 开机启动
5--privileged=true                                   -> 提升容器内权限
6-v /Users/apple/Volumes/redis/conf:/etc/redis/redis.conf                    -> 映射配置文件
7-v /Users/apple/Volumes/redis/data:/data                                   -> 映射数据目录
8--appendonly yes                                    -> 开启数据持久化
9--requirepass "123456"                                         -> 配置密码
```



