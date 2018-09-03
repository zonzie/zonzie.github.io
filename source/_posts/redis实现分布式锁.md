---
title: redis实现分布式锁
date: 2018-08-13 09:49:26
tags: 
	- redis
	- 分布式
	- lock
categories: redis
description: 使用redis实现分布式锁的方法
---
#### redis实现分布式锁

##### 基本的实现思路
- 一个线程,在redis中存入一个加锁的key,和一个特有的value,表示获取到了锁
- 线程需要加锁的操作结束后, 再删除这个key,删除之前需要比对value是否一致
- 其他线程在操作相同的资源之前,也去存入相同的key,如果这个key已经存在,则存入key失败,即获取锁失败,一直重试直到获取锁成功,或者达到超时时间放弃获取锁

##### redis实现的并不可靠的方法
- 这里使用的spring提供的redisTemplate,需要做一些简单的配置
- 由于redisTemplate没有提供同时设置`NX`,`PX`参数的方法,因此这里使用lua脚本实现
```java
    /**
     * 加锁
     * @param key
     * @param value
     * @return
     */
    @GetMapping(value = "lock")
    public Object lock(String key, String value) {
        String script = "local key = KEYS[1]; local value = ARGV[1]; if redis.call('set', key, value, 'NX' ,'PX', 5000) then return 1 else return 0 end";
        DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>(script, Long.class);
        Object execute = redisTemplate.execute(redisScript, Collections.singletonList(key), Collections.singletonList(value));
        System.out.println(execute);
        return execute;
    }

    /**
     * 阻塞锁
     */
    @GetMapping(value = "blockLock")
    public String blockLock(String key, String value) throws InterruptedException {
        // 被阻塞的时间超过5秒就停止获取锁
        int blockTime = 5000;
        // 默认的间隔时间
        int defaultTime = 1000;
        for(;;) {
            if(blockTime >= 0) {
                String script = "local key = KEYS[1]; local value = ARGV[1]; if redis.call('set', key, value, 'NX' ,'PX', 5000) then return 1 else return 0 end";
                DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>(script, long.class);
                Long result = redisTemplate.execute(redisScript, Collections.singletonList(key), value);
                System.out.println("try lock ... ,result: "+result);
                if(result != null && result == 1) {
                    // 得到了锁
                    return "lock success";
                } else {
                    blockTime -= defaultTime;
                    Thread.sleep(1000);
                }
            } else {
                // 已经超时
                return "lock timeout..., please retry later...";
            }
        }
    }

    /**
     * 解锁
     * @param key
     * @param value
     */
    @GetMapping("unlock")
    public String unlock(String key, String value) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end";
        DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>(script, Long.class);
        Long execute = redisTemplate.execute(redisScript, Collections.singletonList(key), value);
        System.out.println("unlock result: "+execute);
        if(execute != null && execute != 0) {
            // 解锁成功
            return "unlock success";
        } else {
            return "unlock failed";
        }
    }
```
- 缺点: 仅适合没有slave的单节点redis,因为slave和master之间的数据是异步复制的,有可能并不一致,只要不是单节点,不推荐使用

##### redis官方推荐的方法-redLock
###### redLock-参考文档:<https://redis.io/topics/distlock>
- redis把分布式锁的算法称之为redLock-红锁, 红锁需要N个(N>=3)redis独立节点,节点之间没有任何联系,保持独立即可.
- 基本实现思路: 客户端在每个redis实例上获取锁,只要大多数实例上获取锁成功就算加锁成功, redLock的算法实现各种语言的版本有很多,java的实现是redisson,可以直接引入使用
- 客户端获取锁的基本流程
    - 首先客户端记录当前的时间,用于计算获取锁的耗时
    - 客户端使用相同的key和随机的value,从所有的redis节点上获取锁.客户端在每个实例上设置锁的过程,需要设置超时时间(5-50ms),不成功就换下一个实例继续设置锁,用来防止客户端阻塞在一个down掉的实例上
    - 客户端需要计算获得锁的总耗时.客户端从至少N/2+1个节点上成功获取锁,且总耗时小于锁过期时间才能成功获得锁
    - 客户端获得锁之后,该锁的有效期不再是最初的过期时间,因为客户端要从多个节点上获得锁,需要去掉这些过程耗时
    - 如果客户端最终获得锁失败,必须在所有的节点上执行锁释放的操作
    - 执行完毕后,主动释放所有的节点上的锁
- redLock保证了
    - 锁的互斥性,同一时间只能有一个锁
    - 不会死锁
    - 多节点容错,只要大部分节点获取锁成功就算得到了锁
- 使用redisson的例子
```java
Config config = new Config();
config.useSingleServer().setAddress("redis://192.168.198.128:6379");
RedissonClient redisson1 = Redisson.create(config);

// 同样的方法再创建两个新的连接  redisson2, redisson3
RLock lock1 = redisson1.getLock("resource");
RLock lock2 = redisson2.getLock("resource");
RLock lock3 = redisson3.getLock("resource");
RedissonRedLock lock = new RedissonRedLock(lock1, lock2, lock3);

// 获取锁
boolean res = lock.tryLock(WAIT_TIME, TIME_OUT, TimeUnit.SECONDS);
// 解锁
lock.unlock();
```
- 一个简单的工具类
```java
public class RedissonRedLocker {
    private static final int WAIT_TIME = 10; // 10s
    private static final long TIME_OUT = 10; // 10s

    private static final String REDIS_NODES_URL = "ip1:port,ip2:port,ip3:port";
    private static RedissonClient[] clients;

    static {
        initRedisInstance();
    }

    private static void initRedisInstance() {
        String[] redisAddrs = REDIS_NODES_URL.split(",");
        List<RedissonClient> list = new ArrayList<RedissonClient>();
        for(String addr: redisAddrs) {
            list.add(getRedisInstance(addr));
        }
        clients = list.toArray(clients);
    }

    private static RedissonClient getRedisInstance(String addr) {
        Config config = new Config();
        config.useSingleServer().setAddress("redis://" + addr);
        return Redisson.create(config);
    }

    private static RedissonRedLock getRedLock(String resource) {
        List<RLock> locks = new ArrayList<RLock>();
        for(RedissonClient client : clients) {
            locks.add(client.getLock(resource));
        }
        return new RedissonRedLock((RLock[]) locks.toArray());
    }

    private static boolean tryLock(RedissonRedLock lock) {
        boolean res = false;
        try {
            res = lock.tryLock(WAIT_TIME, TIME_OUT, TimeUnit.SECONDS);
        } catch(InterruptedException e) {
            e.printStackTrace();
            lock.unlock();
        }
        return res;
    }
}
```
> 参考内容: <br/><http://baijiahao.baidu.com/s?id=1596540166265981065&wfr=spider&for=pc> <br/> <https://blog.csdn.net/l1028386804/article/details/73523810> <br> <https://crossoverjie.top/2018/03/29/distributed-lock/distributed-lock-redis/> <br> <https://blog.csdn.net/forezp/article/details/70305336>