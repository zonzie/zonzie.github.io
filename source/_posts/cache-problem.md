---
title: 缓存系统的问题
date: 2018-09-17 19:27:33
tags: 
    - redis
    - cache
categories: redis
description: 系统中的缓存可能会遇到的问题--缓存穿透, 缓存雪崩, 缓存击穿
---

##### 高并发系统中缓存可能会遇到的问题
1. [缓存穿透](#缓存穿透)
2. [缓存雪崩](#缓存雪崩)
3. [缓存击穿](#缓存击穿)

##### 缓存穿透
- 查询一个一定不存在的key, 由于缓存是没有命中时被动写入的, 一般查不到这个key就不写入缓存, 这将导致不存在的key每次请求都会到达持久层去查询, 在流量大时, 对系统性能会有影响, 可能导致数据库崩溃
- 解决方案:
    1. 使用布隆过滤器,计算所有的可能的key的多种hash值,存入位数组中,一个一定不存在的key会直接被拦截掉, 从而避免了请求到达持久层, 占用数据库资源
    2. 将value为空的key也存入缓存,可以设置较短的过期时间

##### 缓存雪崩
-  指的是缓存中的数据在同一个时间一起失效, 导致大量的请求直接到达数据库层面, 导致数据库直接崩溃
- 解决方案:
    1. 在缓存失效时, 对访问数据库的操作加锁,只允许单个线程访问数据库中的同一条数据
    2. 在设置缓存失效时间时, 加一个随机数, 错开所有的缓存的失效时间, 避免集体失效事件

##### 缓存击穿
- 某些热点数据在某些时间被超高并发的访问, 此时缓存失效, 同样会导致大量的请求涌向数据库, 在重建缓存未完成时, db瞬间被大量的请求压垮
- 解决方案:
    1. 同一条数据在同一时间只允许单个线程访问数据库
    2. 在即将过期之前更新缓存
    3. 设置服务降级

**以lettuce为例解决redis缓存击穿问题:**
lettuce的jar包:
```xml
    <dependencies>
        <dependency>
            <groupid>biz.paluch.redis</groupid>
            <artifactid>lettuce</artifactid>
            <version>5.0.0.beta1</version>
        </dependency>
    </dependencies>
```

**代码实现**:
```java
import com.lambdaworks.redis.*;
import com.lambdaworks.redis.api.statefulredisconnection;
import com.lambdaworks.redis.api.async.redisasynccommands;
import com.lambdaworks.redis.api.sync.rediscommands;
import java.util.concurrent.arrayblockingqueue;
import java.util.concurrent.executionexception;
import java.util.concurrent.threadpoolexecutor;
import java.util.concurrent.timeunit;

/**
 * @author zonzie
 * @date 2018/8/10 17:25
 */
public class lettuceclient {

    // lettuce connection对象
    private static statefulredisconnection<string, string> connection;

    // 线程池
    private static threadpoolexecutor poolexecutor;

    // 互斥锁的key
    private static final string mutex_key = "mutex_key";

    static {
        // 初始化connection
        redisclient client = redisclient.create(redisuri.create("redis://192.168.198.128:6379"));
        connection = client.connect();
        // 初始化线程池
        poolexecutor = new threadpoolexecutor(5, 10, 200, timeunit.seconds, new arrayblockingqueue<runnable>(5), new threadpoolexecutor.discardoldestpolicy());
    }

    /**
     * 解决缓存击穿问题, 方法一
     * 为了解决缓存击穿的问题, 业界常用的做法--设置mutex key
     * @param key 互斥锁的key
     */
    public string mutexget(string key) throws interruptedexception {
        rediscommands<string, string> sync = connection.sync();
        string value = sync.get(key);
        if(value == null) {
            string set = sync.set(mutex_key, "1", setargs.builder.nx().px(3 * 60));
            if("ok".equalsignorecase(set)) {
                // 从数据库获取value
                value = "getvaluebykey";
                sync.set(key, value, setargs.builder.px(1000 * 60 * 60 * 2));
                sync.del(mutex_key);
                return value;
            } else {
                thread.sleep(50);
                // 递归重试
                mutexget(key);
            }
        }
        return value;
    }

    /**
     * 解决缓存击穿的问题,方法二
     * 提前更新key
     * @param key
     */
    public string get(final string key) throws interruptedexception {
        final rediscommands<string, string> sync = connection.sync();
        string value = sync.get(key);
        long ttl = sync.ttl(key);
        // 如果key已经过期,立即从数据库获取新的value
        if(value == null) {
            string set = sync.set(mutex_key, "1", setargs.builder.nx().px(3 * 60));
            if("ok".equalsignorecase(set)) {
                // 从数据库获取value
                string var = "getvaluebykeyfromdb";
                sync.set(key, var, setargs.builder.px(1000 * 60 * 60 * 2));
                sync.del(mutex_key);
                // 返回新的值
                return var;
            } else {
                thread.sleep(50);
                // 递归重试
                get(key);
            }
        }
        // 如果10秒后key过期
        if(ttl < 10) {
            // 异步执行缓存更新操作
            poolexecutor.execute(new runnable() {
                public void run() {
                    // 设置互斥锁
                    string set = sync.set(mutex_key, "1", setargs.builder.nx().px(3 * 60));
                    if("ok".equalsignorecase(set)) {
                        // 从数据库获取value
                        string var = "getvaluebykeyfromdb";
                        // 设置新的value
                        sync.setex(key, 2*60*60, var);
                        // 删除互斥锁
                        sync.del(mutex_key);
                    }
                }
            });
            // 返回旧的值
            return value;
        }
        return value;
    }
}
```

---
> 参考内容: [缓存中的热点key的问题](http://carlosfu.iteye.com/blog/2269687?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)