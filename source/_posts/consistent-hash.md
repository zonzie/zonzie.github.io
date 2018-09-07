---
title: 一致性哈希:原理和Java实现
date: 2018-09-06 20:25:18
tags:
    - hash
categories: hash
description: 一致性哈希的原理以及java实现
---
### 算法的提出
- 一致性哈希算法是在1997年由MIT提出的一种分布式哈希算法, 设计目标是为了解决因特网中的热点问题, 喂鸡百颗的解释: [点这里](https://zh.wikipedia.org/wiki/%E4%B8%80%E8%87%B4%E5%93%88%E5%B8%8C)
- 一致性哈希算法提出了在动态变化的cache中,判定hash算法好换的四个定义
    1. 平衡性: 指的是哈希的结果能够尽可能的分布到所有的缓存中去,这样可以使得所有的缓冲空间都得到利用
    2. 单调性: 如果已经有一些内容通过哈希分派到了相应的缓存中, 又有新的缓存加入到了系统. 哈希的结果能够保证原有的已分配的内容可以映射到原有的或者新的缓存中去,但是不会映射到旧的缓存区
    3. 分散性
    4. 负载
- 在分布式集群中,机器的添加删除,故障后自动脱离集群这些操作是分布式集群管理中最基本的功能,如果采用一般的hash(object)%N的算法,在集群节点稍有变动的情况下, 原有的数据就再也找不到了, 这样严重违反了单调性原则

### 一致性哈希的解释
1. 按照常用的哈希算法将对应的key映射到一个具有2^32个桶的空间中, 将这些数字头尾相连,想象成一个闭合的环
![consistent_hash_1](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/consistent_hash_1.png)
2. 将节点的标识通过hash算法处理后映射到环上,如图,计算三个节点node1-node3的hash值,映射到hash环上
![consistent_hash_2](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/consistent_hash_2.png)
3. 将要存储的key计算hash值后,也映射到环上,按照顺时针(或者逆时针)一个方向,将数据存储到最近的一个节点上, 如图所示
![consisten_hash_3](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/consistent_hash_3.png)
4. 如果加入了新的节点, 失效的缓存只有很少的一部分, 只有key4的缓存失效了,因为它的hash值会映射到node4上,其他的则保持不变
![consisten_hash_4](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/consistent_hash_4.png)
5. 如果有节点挂掉, 同样失效的缓存也是只有很少的一部分, node4挂掉后, 也只有key4会受到影响,其他部分保持不变
![consistent_hash_5](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/consistent_hash_5.png)
6. **但是**,哈希算法并不能保证平衡性,也就是说,通过key计算的hash值, 并不一定能均匀的分布在整个环中, 这样有可能会导致大量的key分布在同一个节点中,如果节点挂掉,就会导致其他节点的压力更大, 从而导致雪崩式的节点崩溃,图中node2承接了来自node3的压力
![consistent_hash_5](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/consistent_hash_7.png)
7. 因此,所有的节点必须均匀分布在整个哈希环中,因此提出了虚拟节点的概念, 虚拟节点是真实节点在哈希空间中的复制品, 一个真实节点可以有很多个虚拟节点, 虚拟节点越多, 整个哈希环中的节点分布就越均匀,图中node1的虚拟节点对应node1_v1和node1_v2, node2对应node2_v1和node2_v2
![consistent_hash_6](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/consistent_hash_6.png)

### hash算法的选取
这里选用的hash算法必须具有很好的分散性,java中的Object的hashcode实现,变化程度太小,例如,需要计算哈希值的key为"node-1","node-2", 计算结果为-1040172250, -1040172249, 计算结果过于集中, 这会使所有的节点分布在哈希环上很小的范围内, 导致缓存无法均匀分布在各个节点上,因此需要选取合适的hash算法, 推荐使用CRC32_HASH、FNV1_32_HASH、 KETAMA_HASH、murmur_hash

### 不带虚拟节点的代码实现
```java
import java.util.SortedMap;
import java.util.TreeMap;

/**
 * 一致性哈希
 * @author zonzie
 * @date 2018/9/5 19:37
 */
public class ConsistentHashingWithoutVirtualNode {

    /** 缓存服务器列表 */
    private static String[] servers = {"192.168.198.128:111", "192.168.198.128:112", "192.168.198.128:113", "192.168.198.128:114", "192.168.198.128:115"};

    /** key表示服务器的hash值, value表示服务器的名称 */
    private static SortedMap<Integer, String> sortedMap = new TreeMap<>();

    static {
        for (String server : servers) {
            int hash = hash(server);
            System.out.println("[" + server + "] 加入集合中, hash值为" + hash);
            sortedMap.put(hash, server);
        }
    }

    /** 使用FNV1_32_HASH算法计算服务器的Hash值*/
    private static int hash(String str) {
        final int p = 16777619;
        int hash = (int) 2166136261L;
        for(int i = 0; i < str.length(); i++)
            hash = (hash ^ str.charAt(i)) * p;
        hash += hash << 13;
        hash ^= hash >> 7;
        hash += hash << 3;
        hash ^= hash >> 17;
        hash += hash << 5;

        // 算出来值为负数就取绝对值
        if(hash < 0)
            hash = Math.abs(hash);
        return hash;
    }

    /**得到应当路由的节点*/
    private static String getServer(String node) {
        int hash = hash(node);
        // 大于这个hash值得所有的节点
        SortedMap<Integer, String> tailMap = sortedMap.tailMap(hash);
        // 第一个key就是离node最近的那个节点
        Integer integer = tailMap.firstKey();
        return tailMap.get(integer);
    }

    public static void main(String[] args) {
        String[] nodes = {"127.0.0.1:1111", "221.226.0.1:2222", "10.211.0.1:3333"};
        for (String node : nodes) {
            System.out.println("节点:" + node + " 哈希值: " + hash(node) + ", 被路由到节点:" + getServer(node));
        }
    }
}
```

### 带有虚拟节点的java实现
```java
import org.junit.Test;
import java.nio.ByteBuffer;
import java.nio.ByteOrder;
import java.util.*;

/**
 * 添加虚拟节点的一致性哈希
 * @author zonzie
 * @date 2018/9/5 20:14
 */
public class ConsistentHashingWithVirtualNode {

    /**
     *  真实的服务器列表
     */
    private static String[] servers = {"192.168.198.128:111", "192.168.198.128:112", "192.168.198.128:113", "192.168.198.128:114", "192.168.198.128:115"};

    /**
     *  真实节点列表
     */
    private static List<String> realNodes = new LinkedList<>();

    /**
     *  虚拟节点列表
     */
    private static SortedMap<Long, String> virtualNodes = new TreeMap<>();

    /**
     *  虚拟节点的数目
     */
    private static final int VIRTUAL_NODES = 100;

    /**
     * 分隔符
     */
    private static final String SEPARATOR = "=>";

    // 初始化节点数据
    static {
        // 先添加原始的服务器到真实的节点列表
        realNodes.addAll(Arrays.asList(servers));
        // 添加虚拟节点
        addVirtualNode(realNodes.toArray(realNodes.toArray(new String[]{})));
    }

    /**
     * 使用
     * FNV1_32_HASH或者murmurHash
     * 计算Hash值
     */
    private static long hash(String str) {
        int hash = murmurHash(str);
        return Math.abs(hash);
    }

    /**
     * 应当路由到的节点
     */
    private static String getServer(String node) {
        long hash = hash(node);
        SortedMap<Long, String> tailMap = virtualNodes.tailMap(hash);
        Long i = null;
        String virtualNode = null;
        if(tailMap.size() == 0) {
            i = virtualNodes.firstKey();
            virtualNode = virtualNodes.get(i);
        } else {
            i = tailMap.firstKey();
            virtualNode = tailMap.get(i);
        }

        // 第一个key就是顺时针过去离node最近的节点
        // 返回虚拟节点的名称
        return virtualNode.substring(0, virtualNode.indexOf(SEPARATOR));
    }

    /**
     * 动态添加节点
     */
    private static void addNode(String... nodes) {
        // 添加真实的节点
        realNodes.addAll(Arrays.asList(nodes));
        // 添加虚拟节点
        addVirtualNode(nodes);
    }

    /**
     * 删除一个节点
     */
    private static void removeNode(String... nodes) {
        // 移除真实节点
        realNodes.removeAll(Arrays.asList(nodes));
        // 移除虚拟节点
        for (String node : nodes) {
            for(int i = 0; i < VIRTUAL_NODES; i++) {
                String virtualNodeName = node + SEPARATOR + i;
                long hash = hash(virtualNodeName);
                System.out.println("移除虚拟节点:" + virtualNodeName + "  hash值为: " + hash);
                virtualNodes.remove(hash);
            }
        }
    }

    /**
     *  添加虚拟节点
     */
    private static void addVirtualNode(String... nodes) {
        // 添加虚拟节点
        for (String s : nodes) {
            for(int i = 0; i < VIRTUAL_NODES; i++) {
                String virtualNodeName = s + SEPARATOR + i;
                long hash = hash(virtualNodeName);
                System.out.println("虚拟节点[" + virtualNodeName + "]被添加, hash值为" + hash);
                virtualNodes.put(hash, virtualNodeName);
            }
        }
    }

    /**
     * FNV1_32_HASH
     */
    private static int fnvHash(String str) {
        final int p = 16777619;
        int hash = (int) 2166136261L;
        for(int i = 0; i < str.length(); i++)
            hash = (hash ^ str.charAt(i)) * p;
        hash += hash << 13;
        hash ^= hash >> 7;
        hash += hash << 3;
        hash ^= hash >> 17;
        hash += hash << 5;

        // 算出来值为负数就取绝对值
        if(hash < 0)
            hash = Math.abs(hash);
        return hash;
    }

    /**
     * murmurHash: 非加密hash算法, 速度快, 碰撞少, 随机分布特征表现良好
     */
    private static int murmurHash(String key) {
        ByteBuffer buf = ByteBuffer.wrap(key.getBytes());
        int seed = 0x1234ABCD;

        ByteOrder byteOrder = buf.order();
        buf.order(ByteOrder.LITTLE_ENDIAN);

        long m = 0xc6a4a7935bd1e995L;
        int r = 47;

        long h = seed ^ (buf.remaining() * m);

        long k;
        while (buf.remaining() >= 8) {
            k = buf.getLong();

            k *= m;
            k ^= k >>> r;
            k *= m;

            h ^= k;
            h *= m;
        }

        if (buf.remaining() > 0) {
            ByteBuffer finish = ByteBuffer.allocate(8).order(
                    ByteOrder.LITTLE_ENDIAN);
            finish.put(buf).rewind();
            h ^= finish.getLong();
            h *= m;
        }

        h ^= h >>> r;
        h *= m;
        h ^= h >>> r;
        buf.order(byteOrder);
        return (int) h;
    }


    /**
     * 测试
     */
    @Test
    public void consistentHashTest {
        HashMap<String, Integer> hashMap = new HashMap<>();
        ArrayList<String> list = new ArrayList<>();
        for(int i = 0; i < 100000; i++) {
            list.add(UUID.randomUUID().toString());
        }

        for(int i = 0; i < list.size(); i++) {
            String server = getServer(list.get(i));
            Integer integer = hashMap.get(server);
            if(integer == null) {
                hashMap.put(server, i);
            } else {
                hashMap.put(server, integer+1);
            }
        }
        Set<String> strings = hashMap.keySet();
        for (String string : strings) {
            System.out.println("节点[" +string+"]分配到的元素个数为"+ hashMap.get(string));
        }
    }
}
```

---
> 参考内容:<br/><http://m.elecfans.com/article/717709.html><br/> <https://www.cnblogs.com/shangxiaofei/p/6859904.html><br/><https://coderxing.gitbooks.io/architecture-evolution/di-san-pian-ff1a-bu-luo/631-yi-zhi-xing-ha-xi.html><br/>hash算法: <https://www.cnblogs.com/wanghetao/p/4658471.html>