## 前言

> 这篇主要讲讲面试中经常遇到的 HashMap ！在下才疏学浅，有不对的地方欢迎指出。

## 名词

* Hash（散列函数）：是把任意长度的输入通过散列算法转换为**均匀分布**固定长度的输出。散列算法是一种广义的算法，使用散列算法可以**提高存储空间的利用率**，可以**提高数据查询的效率**，也可以**作为数字签名来保障数据传递的安全性**。
* Hash 冲突

## 结构

HashMap 是由 **数组** 和 **链表**组合构成的数据结构。他是用来存储 key - value 键值对的集合。在 Java 7 中叫做 Entry，在 Java 8 中叫做 Node。HashMap 数组每个元素的初始值都是 Null。Node 的 next 是 链表的next指针。

### PUT 方法

在前面我们提到 HashMap 是由数组 和 链表组合构成的数据结构。为什么会有链表呢？链表是如何形成的？

#### 源码

```java
    /**
     * Associates the specified value with the specified key in this map.
     * If the map previously contained a mapping for the key, the old
     * value is replaced.
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
      	// 定义引用，可以通过 org.openjdk.jol 查看结构
        Node<K,V>[] tab; Node<K,V> p; int n, i;
      	// 判断 table 是否为空，因为在 HashMap 的构造函数中是不会去初始化 table 的。
        if ((tab = table) == null || (n = tab.length) == 0)
          	// 扩容，拿到桶状数组的长度
            n = (tab = resize()).length;
      	// 判断 key 所在数组的位置是否为空
      	// i = (n - 1) & hash 计算 key 在数组中对应的下标，这里的 (n - 1) & hash 等同于 n % hash, 但是性能优于 n % hash
        if ((p = tab[i = (n - 1) & hash]) == null)
          	// 如果为空，则在构造一个 Node<K, V> 放入桶状数组中，
            tab[i] = newNode(hash, key, value, null);
      	// 如果所对应的 key 不为空
        else {
          	// 定义对象引用
            Node<K,V> e; K k;
          	// 判断 Node<K,V> p 的 hash 值 和 key 的hash 值是否相等， key 不为空的情况下 p.key 是否等于 key
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
              	// key 相同，将 对象p 的引用指向 e，即保留原始的 key-value
                e = p;
          	// 不相同，再判断 Node<K,V> p 是否为红黑树   注意：当链表长度超过 8 的时候才会转为红黑树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
          	// 以上条件都不满足
            else {
              	// 遍历链表
                for (int binCount = 0; ; ++binCount) {
                  	// Node<K,V> p = tab[(length - 1) & hash];  
                  	// 如果 p.next 为 null，则 构造新的 Node<K, V> 指向 p.next。
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                      	// 如果链表的长度达到了 8 个，将链表转为 红黑树 处理，提高查找性能
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                  	// 如果当前 e 的 hash 和新 key 的 hash 相等，则跳出循环
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                  	// 对应 e 的引用指向 p
                    p = e;
                }
            }
          	// 如果 e 不为空
            if (e != null) { // existing mapping for key
              	// 定义旧的 value
                V oldValue = e.value;
              	// onlyIfAbsent:true 不覆盖旧值 / false 覆盖旧值
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
              	// 这个函数只在 LinkedHashMap 中用到，这里为空函数
                afterNodeAccess(e);
              	// 返回旧值
                return oldValue;
            }
        }
        ++modCount;
      	// 将数组大小加一，并判断是否需要扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

由上面的代码我们可以看出调用 put 方法的时候 实际上是 调用了 `putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict)` 方法。

#### 参数解释

|     参数     |                             释义                             |
| :----------: | :----------------------------------------------------------: |
|     hash     |                     key 值对应的 hash 值                     |
|     key      |                        用户传入的 key                        |
|    value     |                       用户传入的 value                       |
| onlyIfAbsent | 该参数的作用是：待存储的 key 在已存在的情况下，要不要用 新值去覆盖旧值，true 不覆盖， false 覆盖 |
|    evict     |              该参数是用来区分当前是否是构造模式              |



#### 示例

```java
private static void testPut(Map<Object, Object> sourceMap) {
    // 默认 sourceMap 的长度为 16。
    sourceMap.put("luozijing", "帅比");
    sourceMap.put("lvzijing", "大帅比");
}
```

我们看这段代码，两次调用 sourceMap 的 put() 方法，会计算出 key 所对应的 hash 去计算 index 值。假设一种极端情况，这两个 key 计算所得的 index 都为 1，这种情况称为 **hash 冲突**，这时候就会形成链表，如下图所示：

<img src="http://qiniuyun.whitenip.site/image/blog/hashmaphash%E5%86%B2%E7%AA%81%E8%BD%AC%E9%93%BE%E8%A1%A8.png" title="hash冲突转链表" style="zoom:50%;" />

**看到这里咱们是不是会有些疑问，为什么是 lvzijing-大帅比 这个键值对在链表中，而不是 luozijing-帅比 在链表中呢？**

​	这就是 **Java 8 之前**和 **Java 8** 的区别了，其实在 **Java 8 之前**采用的是**头插法**，**Java 8** 中采用的是**尾插法**，说来尴尬，博主只用过 Java 8，但是面对面试官的时候，我们肯定不能这么直接的回答。咱们可以先了解[扩容机制](#扩容机制)。因为头插法存在 **环形链表** 的风险。

​	

### 如何实现 HashMap 的顺序存储？

> 可以参考 LinkedHashMap 的底层实现

* 方法一

  > 维护一张表，存储数据插入的顺序，可以使用 vector。但是如果删除数据呢，首先得在 vector 里面找到那个数据，再删除，而删除又要移动大量数据。性能效率很低。 使用 list，移动问题可以解决，但是查找数据的 O(n) 时间消耗，如果删除 m 次，那查找数据的性能就是0（n*m)，那总体性能也是 O(n2)。性能还是没法接受。

* 方法二

  > 可以在 hashmap 里面维护插入顺序的 id, 在 value 建一个字段存储id值，再维护一张表 vector，并且id对应vector里面的值。 插入的时候，id＋＝1， hashmap.insert，vector.push_back. 删除的时候，先 hashmap.find(key), 得到 value, 并从 value 中得到 id,  通过 id把对应 vector 值置为无效。 更新：删除＋插入。 维护工作 OK 了，输出的时候直接输出 vector 里面的值就可以了， 无效的就 continue。 算法复杂度为 O(n) 

* 方法三

  >  Java 里面有个容器 LinkedHashMap, 它能实现按照插入的顺序输出结果。 它的原理也是维护一张表，但它是链表，并且hashmap 中维护指向链表的指针，这样可以快速定位链表中的元素进行删除。 它的时间复杂度也是 O(n), 空间上要比“方法二”少些

### GET 方法 

#### 源码

```java
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
```

get 方法就很简单了，传入 key 调用 getNode 方法，需要传入 key 对应的 hash 值和 key，key 存在返回 value， 不存在返回 null。

## 扩容机制

扩容操作发生的情况有两种：

* 初始化（HashMap 的构造函数都不会去初始化 Node<K,V>[] table）
* table 的大小超过 threshold 值。

关于扩容机制需要引入 **负载因子** `static final float DEFAULT_LOAD_FACTOR = 0.75f;` 和 **默认初始化长度**  `static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;`

话不多说先贴上源码（JDK 1.8），同时进行分析：

```java
    /**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;	// 保存旧数组（当前操作的数组）
        int oldCap = (oldTab == null) ? 0 : oldTab.length; // 计算旧数组的容量
        int oldThr = threshold;	// 保存旧阈值
        int newCap, newThr = 0; // 定义新数组的容量、阈值的引用并赋值
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) { // 判断旧数组是否达到最大容量限制
                threshold = Integer.MAX_VALUE; // 已经超过最大容量限制，不再扩容直接返回
                return oldTab;
            }
            // 新数组的容量（旧数组容量的两倍）是否小于最大容量 和 旧数组的容量是否大于等于默认初始化容量 做与运算
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY) 
                newThr = oldThr << 1; // 扩容两倍的阈值
        }
      	// 在构造 HashMap 时，
      	// 如果我们没有指定 initialCapacity ，threshold 会初始化为 0
      	// 如果我们指定 initialCapacity ，threshold 会初始化为大于 initialCapacity 的最小的2的次幂
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr; // 这里指的是，将指定 initialCapacity 将新数组的容量等于旧数组的阈值，即容量为旧数组的两倍。
        else {               // 都不满足的情况，新数组容量和阈值都使用默认值
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) { // 初始化新阈值
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr; // 将新阈值赋给当前对象
      	// 开始构造新数组
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap]; // 以newCap构造新的Node数组
        table = newTab; // 将新数组赋给当前对象
        if (oldTab != null) { // 旧数组不为空将执行以下操作
            for (int j = 0; j < oldCap; ++j) { // 遍历旧数组
                Node<K,V> e; // 创建 Node<K,V> 的引用 e
                if ((e = oldTab[j]) != null) { // 将旧的Node<K,V>指向对象引用 e  ，并做非空判断
                    oldTab[j] = null; // 清除旧表的引用（真正的 Node 节点还存在），释放内存
                    if (e.next == null)	// 判断 e 后面是否还有元素
                        newTab[e.hash & (newCap - 1)] = e; // 没有元素，计算 e 在新数组中的 index,并存放 
                    // 有元素，继续判断是否是红黑树，做拆分处理
                    else if (e instanceof TreeNode) 
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap); 
                    else { // 链表结构逻辑，避免 hash 冲突
                      	// 定义链表及两个 Node 引用，从名字上面看出为 lo头节点，lo尾节点
                        Node<K,V> loHead = null, loTail = null; 
                      	// 定义链表及两个 Node 引用，从名字上面看出为 hi头节点，hi尾节点
                        Node<K,V> hiHead = null, hiTail = null;	
                        Node<K,V> next; // 定义下一个 Node 引用
                        // do while 循环体，按照顺序遍历链表中的节点，将节点 e 插入到链表中
                        do { 
                            next = e.next;
                          	// 以 (e.hash & oldCap) == 0 为判断依据来选择插入的链表
                            if ((e.hash & oldCap) == 0) {
                              // 插入 lo链表
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                              // 插入 hi链表
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null); // 当下个节点没有元素时结束循环
                      	// 如果 lo链表非空，则将整个 lo链表放到新数组的 j 位置上
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                      	// 如果 hi链表非空，则将整个 hi链表放到新数组的 j + oldCap 位置上
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

#### 链表拆分

给大家贴上一张链表拆分网图：

<img src="http://qiniuyun.whitenip.site/image/blog/hashmap%E9%93%BE%E8%A1%A8%E6%8B%86%E5%88%86.png" title="链表拆分" style="zoom:200%;" />



##### 关于(e.hash & oldCap) == 0

拿默认的容量来举例：

```30java
首先Hash的公式：
n = (tab = resize()).length; // 源码 629 行
i = (n - 1) & hash;  				 // 源码 630 行
即 index = (capacity - 1) & hash;
为什么是 index = hash & (capacity - 1) 而不是 hash % capacity 取余数？
回答是：位运算更高效！

oldCap = 16; // oldCap 一定是2的幂
二进制表示为：0000 0000 0000 0000 0000 0000 0001 0000
n - 1 = 15;
二进制表示为：0000 0000 0000 0000 0000 0000 0000 1111
n - 1 比 oldCap 低 4 位，再去和 hash 做位于计算，就保证了 index 一定在 capacity 范围内。
--------------------------------------------------
再回到 (e.hash & oldCap) == 0
这里其实就是做的一个临界值计算。
下图为我的计算过程：
```

<img src="http://qiniuyun.whitenip.site/image/blog/hashmap%E9%93%BE%E8%A1%A8%E6%8B%86%E5%88%86%E8%AE%A1%E7%AE%97%E8%BF%87%E7%A8%8B.jpg" title="blockchina" title="链表拆分计算示例" style="zoom:150%;" />

### HashMap 的默认初始化长度是多少，为什么？

源码中很明确的指出了，HashMap 的默认初始化长度 **DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16** 且必须为2的幂。

## ConcurrentHashMap 你了解多少？

