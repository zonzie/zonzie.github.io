---
title: JDK中的Map
date: 2018-08-14 15:15:40
tags: 
	- hash算法
	- HashMap
	- threadLocal
categories: hashMap
description: jdK中的几种map的实现的源码注释,会一直更新
---
#### jdk1.7的hashMap源码注释
```java
import sun.security.action.GetPropertyAction;

import java.security.AccessController;
import java.util.Map;
import java.util.Objects;

/**
 * jdk1.7 HashMap的实现
 * @author zonzie
 * @date 2018/7/9 19:50
 */
public class TestHashMap7<K, V> {
    /** 初始容量, 默认16 */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

    /** 最大初始容量, 2^30 */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /** 负载因子, 默认0.75, 负载因子越小, hash冲突几率越低 */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /** 初始化一个Entry数组 */
    static final Entry<?,?>[] EMPTY_TABLE = {};

    /** 将初始化好的空数组赋值给table, table数组是hashMap实际存储数据的地方,并不在EMPTY_TABLE数组中 */
    transient Entry<K, V>[] table = (Entry<K, V>[]) EMPTY_TABLE;

    /** HashMap实际存储的元素个数 */
    transient int size;

    /** 临界值(HashMap实际能存储的大小), 公式为(threshold = capacity * loadFactor) */
    int threshold = 1;

    /** 负载因子 */
    final float loadFactor;

    /** HashMap 的结构被修改的次数, 用于迭代器 */
    transient int modCount;

    /**散列的默认阈值*/
    static final int ALTERNATIVE_HASHING_THRESHOLD_DEFAULT = Integer.MAX_VALUE;

    int hashSeed = 0;

    // holds values which can't be initialized until after VM is booted.
    private static class Holder {
        //Table capacity above which to switch to use alternative hashing.
        static final int ALTERNATIVE_HASHING_THRESHOLD;

        static {
//          关于AccessController.doPrivileged的解释  http://www.blogjava.net/DLevin/archive/2012/11/02/390637.html
//          关于jdk.map.althashing.threshold  https://my.oschina.net/huangy/blog/1619144
            String altThreshold = AccessController.doPrivileged(new GetPropertyAction(("jdk.map.althashing.threshold")));
            int threshold;
            try {
                threshold = (null != altThreshold) ? Integer.parseInt(altThreshold) : ALTERNATIVE_HASHING_THRESHOLD_DEFAULT;
                // disable alternative hashing if -1
                if(threshold == -1) {
                    threshold = Integer.MAX_VALUE;
                }
                if(threshold < 0) {
                    throw new IllegalArgumentException("value must be positive");
                }
            } catch (IllegalArgumentException failed) {
                throw new Error("Illegal value for 'jdk.map.althashing.threshold'", failed);
            }
            ALTERNATIVE_HASHING_THRESHOLD = threshold;
        }
    }

    // 指定初始化容量和负债因子的构造
    public TestHashMap7(int initialCapacity, float loadFactor) {
        // 判断设置的容量和负载因子合不合理
        if(initialCapacity < 0)
            throw new IllegalArgumentException("illegal initial capacity: " + initialCapacity);
        if(initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if(loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " + loadFactor);

        // 设置负载因子,临界值此时为默认大小, 后面第一次put时由inflateTable(int toSize) 方法初始化一个数组table
        this.loadFactor = loadFactor;
        threshold = initialCapacity;
    }

    public TestHashMap7(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    // 根据已有的map创建一个新的相应容量的map
    public TestHashMap7(Map<? extends K, ? extends V> m) {
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1, DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
    }

    public TestHashMap7() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
    }

    /**
     *初始化表的长度
     * @param toSize
     */
    private void inflateTable(int toSize) {
        int capacity = roundUpToPowerOf2(toSize);
        threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
        // 创建一个Entry数组,作为最初的table
        table = new Entry[capacity];
        // 是否需要改变hashSeed的值
        initHashSeedAsNeeded(capacity);
    }

    // 判断是否需要给hashSeed重新赋值,如果重新赋值,需要重新计算hash值和桶的位置
    final boolean initHashSeedAsNeeded(int capacity) {
        boolean currentAltHashing = hashSeed != 0;
        boolean useAltHashing = sun.misc.VM.isBooted() && (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
        boolean switching = currentAltHashing ^ useAltHashing;
        if(switching) {
            hashSeed = useAltHashing ? sun.misc.Hashing.randomHashSeed(this) : 0;
        }
        return switching;
    }

    // 计算容量的大小
    private static int roundUpToPowerOf2(int number) {
        return number >= MAXIMUM_CAPACITY ? MAXIMUM_CAPACITY : (number > 1) ? Integer.highestOneBit((number-1) << 1) : 1;
    }

    // 计算hash值
    final int hash(Object k) {
        int h = hashSeed;
        if(0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }
        h ^= k.hashCode();
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

    // 计算索引,桶的位置
    static int indexFor(int h, int length) {
        int i = h & (length - 1);
        return i;
    }

    // map的尺寸
    public int size() {
        return size;
    }

    // 判断是否为空
    public boolean isEmpty() {
        return size == 0;
    }

    // 获取null key的值
    private V getForNullKey() {
        if(size == 0) {
            return null;
        }
        for(Entry<K,V> e = table[0]; e != null; e = e.next) {
            if(e.key == null)
                return e.value;
        }
        return null;
    }

    // 判断是否存在key
    public boolean containsKey(Object key) {
        return getEntry(key) != null;
    }

    // 获取一个Entry
    private Entry<K,V> getEntry(Object key) {
        if(size == 0) {
            return null;
        }
        int hash = (key == null) ? 0 : hash(key);
        int i = indexFor(hash, table.length);
        // 遍历索引值为i的位置的entry的所有的元素
        for(Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            // 判断key的hash值是否相等,同时key的equals方法需要返回true
            if(e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }

    public V put(K key, V value) {
        if(table == EMPTY_TABLE) {
            // 如果没有初始化table的长度,赋予默认值16,表的容量是16*0.75=12
            inflateTable(threshold);
        }
        if(key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        // 遍历索引为i的entry
        for(Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            // 如果key的hash值相等同时key值也相等,用新的value替换旧的value,并返回
            if(e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                return oldValue;
            }
        }
        // 修改次数加1
        modCount++;
        // 添加一个新的entry
        addEntry(hash, key, value, i);
        return null;
    }

    // null值null键单独处理
    private V putForNullKey(V value) {
        for(Entry<K,V> e = table[0]; e != null; e = e.next) {
            if(e.key == null) {
                V oldValue = e.value;
                e.value = value;
//                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
        addEntry(0, null, value, 0);
        return null;
    }

    public V get(Object key) {
        if(key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);
        return null == entry ? null : entry.getValue();
    }

    // 静态内部类Entry,链表结构,1.7的hashMap使用拉链法解决hash碰撞
    static class Entry<K, V> implements Map.Entry<K,V> {

        final K key;
        V value;
        Entry<K,V> next;
        int hash;

        /**
         * 创建一个Entry
         * @param h key的hash值
         * @param k key
         * @param v value
         * @param n 下一个节点
         */
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }

        public K getKey() {
            return key;
        }

        public V getValue() {
            return value;
        }

        // 设置新值,返回旧值
        public V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if(!(o instanceof Map.Entry))
                return false;
            Map.Entry e = (Map.Entry)o;
            Object k1 = getKey();
            Object k2 = e.getKey();
            if(k1 == k2 || (k1 != null && k1.equals(k2))) {
                Object v1 = getValue();
                Object v2 = e.getValue();
                if(v1 == v2 || v1 != null && v1.equals(v2))
                    return true;
            }
            return false;
        }

        public final int hashCode() {
            return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
        }

        public final String toString() {
            return getKey() + "=" + getValue();
        }
    }

    void addEntry(int hash, K key, V value, int bucketIndex) {
        // 如果table的长度大于临界值同时桶的位置至少有一个元素,需要对表进行扩容
        if((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            // 扩容以后需要重新计算hash值
            hash = (null != key) ? hash(key) : 0;
            // 重新计算桶的位置
            bucketIndex = indexFor(hash, table.length);
        }
        // 创建一个新的entry
        createEntry(hash, key, value, bucketIndex);
    }

    // 根据给定的参数创建一个新的entry
    void createEntry(int hash, K key, V value, int bucketIndex) {
        // 取出指定位置的entry
        Entry<K,V> e = table[bucketIndex];
        // 创建的新的entry放在指定的桶的位置上
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        // map元素个数加1
        size++;
    }

    // 表扩容
    void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        // 如果容量达到最大值,不再扩容,直接返回
        if(oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }
        // 创建新的table,指定新的容量
        Entry[] newTable = new Entry[newCapacity];
        // 迁移数据到新的表
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        // 重新指定阈值
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY+1);
    }

    // 将旧表的数据迁移到新的扩容后的表,同时确定是否需要重新计算hash值
    void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        // 遍历旧的表
        for(Entry<K,V> e : table) {
            while (null != e) {
                Entry<K,V> next = e.next;
                if(rehash) {
                    // 使用新的hsahseed 重新计算hash值
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                // 根据hash值重新计算桶的位置
                int i = indexFor(e.hash, newCapacity);
                // 将entry中的next赋值为i处的元素
                e.next = newTable[i];
                // 再将i处的元素重新赋值为整个entry
                newTable[i] = e;
                // 指向旧表中entry的下一个节点
                e = next;
            }
        }
    }
}
```

#### ThreadLocal的源码注释
```java
import java.lang.ref.WeakReference;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @author zonzie
 * @date 2018/8/27 15:24
 */
public class ThreadLocalTest<T> {

    /**
     * threadLocals基于每一个线程的线性检测哈希映射附加到每一个线程上,
     * ThreadLocal对象本身作为key, 通过threadLocalHashCode搜索,
     * 这是一个自定义的hashcode, 消除了在相同线程数使用连续构造的threadLocal的常见情况下的冲突,同时在不太常见的情况下保持良好的行为
     */
    private final int threadLocalHashCode = nextHashCode();

    /**
     * 给出下一个hashcode,从零开始
     */
    private static AtomicInteger nextHashCode = new AtomicInteger();

    /**
     * magic number ,以这个数为间隙的数值与2^n取模得到的数值可以均匀的分布在整个哈希表中
     * ThreadLocalMap使用的解决冲突的方法是 "线性探测法",
     * 均匀分布的好处是可以很快就探测到下一个临近的可用的slot,从而保证效率,因此哈希表的大小要保证是2^n
     * 和连续增长的hashcode不同, 0x61c88647 更适合长度为2^n的表
     */
    private static final int HASH_INCREMENT = 0x61c88647;

    /**
     * 返回下一个hashcode
     */
    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }

    /**
     * 为threadLocal初始化一个值
     */
    protected  T initialValue() {
        return null;
    }

    /**
     * 创建一个thread local 变量
     */
    public ThreadLocalTest() {
    }

    /**
     * 返回当前线程的此线程局部变量副本中的值
     * 如果变量没有当前线程的值，则首先将其初始化为调用initialValue方法返回的值
     * @return
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if(map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if(e != null)
                return (T) e.value;
        }
        return setInitialValue();
    }

    /**
     *
     * @return
     */
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if(map != null)
            map.set(this,value);
        else
            createMap(t, value);
        return value;
    }

    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

    /**
     * 获取和一个线程关联的map
     * @param t
     * @return
     */
    private ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    /**
     * 移除一个元素
     */
    private void remove() {
        ThreadLocalMap m = getMap(Thread.currentThread());
        if(m != null)
            m.remove(this);
    }

    /**
     * 创建一个和threadLocal关联的map
     * @param t
     * @param firstValue
     */
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

    static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
        return new ThreadLocalMap(parentMap);
    }

    T childValue(T parentValue) {
        throw new UnsupportedOperationException();
    }

    /**
     * threadLocalMap 是个自定义的哈希映射,仅适合维护threadLocal的值,
     * 为了帮助处理非常大且长寿命的用法，哈希表条目使用WeakReferences作为键,
     * 但是，由于未使用引用队列，因此只有在表开始空间不足时才能保证删除过时条目
     */
    static class ThreadLocalMap {

        /**
         * 这个entries 在这个哈希映射中继承自WeakReference,
         * 使用主要引用字段作为key(总会是一个threadLocal对象)
         * 需要注意的是key是null时,意味着这个key不再被引用, 所以这个这个entry会从表中擦除,
         * 这些entries被称为 "陈旧的entry(stale entries)"
         * --------------------------------------------
         * 使用弱引用: java中的四级引用中的第三级,一个对象,如果没有强引用链可达,一般活不过下一次GC
         * 这里使用弱引用,在threadLocal没有强引用可达时, 会被GC回收, 在ThreadLocalMap里对应的Entry会失效,这为垃圾清理提供便利
         */
        static class Entry extends WeakReference<ThreadLocalTest> {
            // ThreadLocal中实际存储的数据
            Object value;

            public Entry(ThreadLocalTest k, Object v) {
                super(k);
                value = v;
            }
        }

        /**
         * 初始化容量 -- 必须是2^n次方
         */
        private static final int INITIAL_CAPACITY = 16;

        /**
         * 这个table
         * table.length 必须是2^n次方
         */
        private Entry[] table;

        /**
         * 表中的entry的个数
         */
        private int size = 0;

        /**
         *  需要扩容的阈值,默认是0
         */
        private int threshold;

        /**
         * 设置阈值,最坏情况下 2/3 负载因子
         */
        private void setThreshold(int len) {
            threshold = len * 2 / 3;
        }

        /**
         * 上一个索引
         */
        private static int nextIndex(int i, int len) {
            return ((i + 1 < len) ? i + 1 : 0);
        }

        /**
         * 下一个索引
         */
        private static int prevIndex(int i, int len) {
            return ((i -1) >= 0 ? i - 1 : len - 1);
        }

        /**
         * 构建一个新的map,初始化容量 (firstKey, firstValue)
         * ThreadLocalMaps是延迟构建的,只有在至少有一个entry的时候去构建一个ThreadLocalMaps
         */
        ThreadLocalMap(ThreadLocalTest firstKey, Object firstValue) {
            // 初始化table数组
            table = new Entry[INITIAL_CAPACITY];
            // 用firstKey的threadLocalhashCode与初始化大小16取模得到哈希值,使用"&(2^n - 1)"代替"%(2^n)",加快计算效率
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            // 创建这个节点
            table[i] = new Entry(firstKey, firstValue);
            // 设置节点表的大小 = 1
            size = 1;
            // 设定需要扩容的阈值
            setThreshold(INITIAL_CAPACITY);
        }

        /**
         * 构建一个新的map, 包含所有的继承的ThreadLocals,只会被createInheritedMap调用
         * @param parentMap
         */
        private ThreadLocalMap(ThreadLocalMap parentMap) {
            Entry[] parentTable = parentMap.table;
            int len = parentTable.length;
            setThreshold(len);
            table = new Entry[len];

            for(int j = 0; j < len; j++) {
                Entry e = parentTable[j];
                if(e != null) {
                    ThreadLocalTest key = e.get();
                    if(key != null) {
                        Object value = key.childValue(e.value);
                        Entry c = new Entry(key, value);
                        int h = key.threadLocalHashCode & (len - 1);
                        while (table[h] != null)
                            h = nextIndex(h, len);
                        table[h] = c;
                        size++;
                    }
                }
            }
        }

        /**
         * 获取和这个entry关联的key,这里只处理直接命中的情况,
         * 其他情况由getEntryAfterMiss处理,这里被设计为最大化直接命中的性能
         * @param key threadLocalTest 对象
         * @return 与key关联的entry
         */
        private Entry getEntry(ThreadLocalTest key) {
            // 根据key获取索引
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            // 对应的Entry存在并且没有失效且弱引用指向的ThreadLocal就是key,则命中返回
            if(e != null && e.get() == key)
                return e;
            else
                // 没有直接命中,继续向后找
                return getEntryAfterMiss(key, i, e);
        }

        /**
         * 当getEntry方法没有直接在哈希槽中发现这个值时被调用
         * @param key threadLocalTest object
         * @param i key 的 hashcode
         * @param e 在table中的entry
         * @return 返回值
         */
        private Entry getEntryAfterMiss(ThreadLocalTest key, int i, Entry e) {
            Entry[] tab = table;
            int len = tab.length;

            // 基于线性探测法不断向后探测直到遇到空的entry
            while(e != null) {
                ThreadLocalTest k = e.get();
                // 命中返回
                if(k == key)
                    return e;
                // key已经被回收,调用expungeStaleEntry清理无效的entry
                if(k == null)
                    expungeStaleEntry(i);
                else
                    // 继续往后走
                    i = nextIndex(i, len);
            }
            return null;
        }

        /**
         * 设置与这个key关联的值
         * @param key threadLocalTest 对象
         * @param value 要关联的值
         */
        private void set(ThreadLocalTest key, Object value) {

            // 这里没有像getEntry的时候一样, 使用一个fast path,
            // 因为使用set去创建一个对象和替换一个已经存在的的对象一样常见
            // 因此, fast path会更容易失败
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len - 1);

            // 线性探测, 基于一个索引向后找
            for(Entry e = table[i]; e != null; e = tab[i = nextIndex(i, len)]) {
                ThreadLocalTest k = e.get();
                // 找到返回
                if(k == key) {
                    e.value = value;
                    return;
                }

                // 替换失效的entry
                if(k == null) {
                    replaceStableEntry(key, value, i);
                    return;
                }
            }

            table[i] = new Entry(key, value);
            int sz = ++size;
            if(!cleanSomeSlots(i,sz) && sz >= threshold)
                rehash();
        }

        /**
         * 移除这个key的entry
         */
        private void remove(ThreadLocalTest key) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len - 1);
            for(Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
                // 显示的断开弱引用
                e.clear();
                // 进行段清理
                expungeStaleEntry(i);
                return;
            }
        }

        /**
         * 将set操作期间遇到的陈旧entry替换为指定键的entry。
         * 无论指定键的entry是否已存在，value参数中传递的值都存储在entry中
         *
         * 作为副作用，此方法将清除包含所有的过时entry的。两个空槽之间的一系列entry都会被清除
         * @param key the key
         * @param value the value associated with key
         * @param staleSlot 搜索key的时候遇到的第一个陈旧的entry
         */
        private void replaceStableEntry(ThreadLocalTest key, Object value, int staleSlot) {
            Entry[] tab = table;
            int len = tab.length;
            Entry e;

            // 先备份,再清理过时的entry
            // 避免由于GC释放串中的refs（即，每当GC运行时）不断进行增量重复。
            // 向前扫描, 查找最前的一个无效的slot
            int slotToExpunge = staleSlot;
            for(int i = prevIndex(staleSlot, len); (e = tab[i]) != null; i = prevIndex(i, len)) {
                if(e.get() == null) {
                    slotToExpunge = i;
                }
            }

            // Find either the key or trailing null slot of run, whichever
            // occurs first
            for(int i = nextIndex(staleSlot, len); (e = tab[i]) != null; i = nextIndex(i, len)) {
                ThreadLocalTest k = e.get();

                // 如果我们找到键，那么我们需要将它与旧的entry交换以维护哈希表顺序
                // 旧的哈希槽将会被送到expungeStaleEntry去移除或者rehash
                if(k == key) {
                    // 更新value的值
                    e.value = value;

                    tab[i] = tab[staleSlot];
                    tab[staleSlot] = e;

                    // 如果存在,就开始擦除前面旧entry
                    // 如果在扫描过程中(一开始的向前扫描和i之前的向后扫描)
                    // 找到了之前的无效slot,则以那个位置作为清理的起点,否则以当前的i作为清理的起点
                    if(slotToExpunge == staleSlot)
                        slotToExpunge = i;
                    // 从slotToExpunge开始做一次连续段的清理,再做一次启发式的清理
                    cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
                    return;
                }

                // If we didn't find stale entry on backward scan, the
                // first stale entry seen while scanning for key is the
                // first still present in the run.
                // 如果当前的slot已经无效,并且向前扫描过程中没有无效的slot,则更新slotToExpunge为当前位置
                if(k == null && slotToExpunge == staleSlot)
                    slotToExpunge = i;
            }

            // 如果没有发现key, 把新的entry放到旧的槽中
            tab[staleSlot].value = null;
            tab[staleSlot] = new Entry(key, value);

            // If there are any other stale entries in run, expunge them
            // 如果当前的slot已经无效,并且向前扫描过程中没有无效的slot,则做一次清理
            if(slotToExpunge != staleSlot)
                cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
        }

        /**
         * 核心清理函数 从staleSlot开是遍历,将无效(弱引用指向对象被回收)的entry清理(先将对应的value置空,再将table[i]置空,直到扫到空的entry)
         * 另外,对非空的entry做rehash
         * 这个函数的作用: 从staleSlot开始清理连续段的slot(断开强引用 rehash slot)
         * -----------
         * 通过重新处理staleSlot和下一个空槽之间的任何可能碰撞的条目来清除过时的entry
         * 这也会删除在尾随空值之前遇到的任何其他陈旧entry
         * @param staleSlot
         * @return
         */
        private int expungeStaleEntry(int staleSlot) {
            Entry[] tab = table;
            int len = tab.length;

            // 擦除旧的槽中的entry,显示的断开强引用
            tab[staleSlot].value = null;
            // 显示的设置entry为null, 以便于垃圾回收
            tab[staleSlot] = null;
            size--;

            // rehash直到遇到null
            Entry e;
            int i;
            // 从i开始连续清理直到遇到为null的entry
            for(i = nextIndex(staleSlot, len); (e = tab[i]) != null; i = nextIndex(i ,len)) {
                ThreadLocalTest k = e.get();
                if(k == null) {
                    e.value = null;
                    tab[i] = null;
                    size--;
                } else {
                    /*
                     * 对于还没有被回收的情况,需要做一次rehash
                     *
                     * 如果rehash后,新的hashcode h != i, 则从h向后线性探测到第一个空的slot,把当前的entry放进去
                     */
                    int h = k.threadLocalHashCode & (len - 1);
                    if(h != i) {
                        tab[i] = null;

                        // Unlike Knuth 6.4 Algorithm R, we must scan until
                        // null because multiple entries could have been stale.
                        // 源码注释表示: 不能套用Knuth高德纳的著作TAOCP6.4章中的R算法
                        // R算法描述了如何从使用线性探测的散列表中删除一个元素, R算法维护了一个上次删除元素的index,
                        // 当在某个非空连续段中扫描到某个entry的哈希值取模后的索引,还没遍历到时, 会将entry挪到index那个位置,并且
                        // 更新当前位置为新的index,然后继续向后扫描直到遇到空的entry
                        // 由于使用了弱引用,每个slot的状态有 有效,无效,空,不能直接使用R算法
                        // expungeStaleEntry函数在扫描过程中会对无效的slot清理将之转换为空的slot
                        // 直接使用R算法, 可能会出现具有相同的哈希值的entry之间断开(中间有空的entry)
                        while (tab[h] != null)
                            h = nextIndex(h, len);
                        tab[h] = e;
                    }
                }
            }
            // 返回staleSlot之后第一个空的slot索引
            return i;
        }

        /**
         * 启发式扫描一些旧的entry的slot。
         * 添加新元素或删除另一个旧元素时会调用此方法。
         * 它执行对数扫描，作为无扫描（快速但保留垃圾）和与元素数量成比例的多个扫描之间的平衡，
         * 这将找到所有垃圾但会导致一些插入花费O（n）时间。
         * 这个函数会有两处会被调用, 一处是插入的时候可能会被调用,另外是在替换无效slot的时候可能会被调用
         * 区别是前者传入的n为元素的个数, 后者为table的容量
         * @param i 一个已知没有旧的entry的位置。 扫描从i之后的元素开始。
         * @param n 扫描控制次数 正常情况下,如果扫描了log n次,没有发现无效的slot,函数也就结束了
         *          但这里如果发现了无效的slot,会将n置为table的长度len,做一次连续段的清理,再从下一个空的slot开始继续扫描
         * @return  true if any stale entries have been removed.
         */
        private boolean cleanSomeSlots(int i, int n) {
            boolean removed = false;
            Entry[] tab = table;
            int len = tab.length;
            do {
                // i 在任何情况下自己都不会是一个无效的slot,所以从下一个开始判断
                i = nextIndex(i, len);
                Entry e = tab[i];
                if(e != null && e.get() == null) {
                    n = len;
                    removed = true;
                    i = expungeStaleEntry(i);
                }
            } while ((n >>>= 1) != 0);
            return removed;
        }

        /**
         * 首先扫描整个表,删除无用的entry
         * 如果不足以缩小表的大小, 则表的大小加倍
         */
        private void rehash() {
            expungeStaleEntries();

            if(size >= threshold - threshold / 4) {
                resize();
            }
        }

        /**
         * 扩容
         * 表的容量加倍
         */
        private void resize() {
            Entry[] oldTab = table;
            int oldLen = oldTab.length;
            int newLen = oldLen * 2;
            Entry[] newTab = new Entry[newLen];
            int count = 0;

            for(int j = 0; j < oldLen; ++j) {
                Entry e = oldTab[j];
                if(e != null) {
                    ThreadLocalTest k = e.get();
                    if(k == null) {
                        e.value = null;
                    } else {
                        int h = k.threadLocalHashCode & (newLen - 1);
                        while (newTab[h] != null)
                            h = nextIndex(h, newLen);
                        newTab[h] = e;
                        count++;
                    }
                }
            }

            setThreshold(newLen);
            size = count;
            table = newTab;
        }

        /**
         * 擦除表中的所有的无用的节点
         */
        private void expungeStaleEntries() {
            Entry[] tab = table;
            int len = tab.length;
            for(int j = 0; j < len; j++) {
                Entry e = tab[j];
                if(e != null && e.get() == null)
                    expungeStaleEntry(j);
            }
        }
    }
}
```

> 参考内容: <br/><http://www.blogjava.net/DLevin/archive/2012/11/02/390637.html> <br/> <https://my.oschina.net/huangy/blog/1619144> <br/> 拉链法: <http://www.cnblogs.com/lizhanwu/p/4303410.html> <br/> 为啥是31: <https://blog.csdn.net/qq_38182963/article/details/78940047> <br/><https://stackoverflow.com/questions/299304/why-does-javas-hashcode-in-string-use-31-as-a-multiplier%EF%BC%89> <br/> <https://crossoverjie.top/2018/07/23/java-senior/ConcurrentHashMap/> <br/> <https://www.cnblogs.com/chenssy/p/3521565.html> <br/> ThreadLocal解释: <https://www.cnblogs.com/micrari/p/6790229.html> <br/> java中的引用:<https://www.cnblogs.com/huajiezh/p/5835618.html>
