## Java 集合类

### Map 继承关系

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/f7fe16a2.png)

Map 接口主要有四个常用的实现类 **HashMap/HashTable/LinkedHashMap/TreeMap**

下面针对各个实现类的特点做一些说明：

(1) HashMap：它根据键的hashCode值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。 HashMap最多只允许一条记录的键为null，允许多条记录的值为null。HashMap非线程安全，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致。如果需要满足线程安全，可以用 Collections的synchronizedMap方法使HashMap具有线程安全的能力，或者使用ConcurrentHashMap。

(2) Hashtable：Hashtable是遗留类，很多映射的常用功能与HashMap类似，不同的是它承自Dictionary类，并且是线程安全的，任一时间只有一个线程能写Hashtable，并发性不如ConcurrentHashMap，因为ConcurrentHashMap引入了分段锁。Hashtable不建议在新代码中使用，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换。

(3) LinkedHashMap：LinkedHashMap是HashMap的一个子类，保存了记录的插入顺序，在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的，也可以在构造时带参数，按照访问次序排序。

(4) TreeMap：TreeMap实现SortedMap接口，能够把它保存的记录根据键排序，默认是按键值的升序排序，也可以指定排序的比较器，当用Iterator遍历TreeMap时，得到的记录是排过序的。如果使用排序的映射，建议使用TreeMap。在使用TreeMap时，key必须实现Comparable接口或者在构造TreeMap传入自定义的Comparator，否则会在运行时抛出java.lang.ClassCastException类型的异常。

### HashMap 实现原理

#### 存储结构

从实现结构上来讲，HashMap 是数组+链表+红黑树（Java 8 增加了红黑树部分）实现的，如下图所示：

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/e4a19398.png)

1. 图中的 Node 是HashMap 的一个内部类，实现了 Map.Entry 接口，本质就是一个键值对对象，源码如下：

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;    //用来定位数组索引位置
        final K key;
        V value;
        Node<K,V> next;   //链表的下一个node

        Node(int hash, K key, V value, Node<K,V> next) { ... }
        public final K getKey(){ ... }
        public final V getValue() { ... }
        public final String toString() { ... }
        public final int hashCode() { ... }
        public final V setValue(V newValue) { ... }
        public final boolean equals(Object o) { ... }
}
```

2. HashMap 使用哈希表（ Node[] table ）来存储数据。哈希表解决冲突一般有两种方式：开放地址法和链地址法。Java 采用了链地址法。

3. 如果数组很大，即使较差的 hash 算法也会比较分散；如果数组很小，即使较好的 hash 算法也会出现较多冲突，所以需要在空间成本和时间成本之间权衡。Java 中的 HashMap 实现了**扩容机制**和好的 **hash 算法**。

4. HashMap 中定义的几个字段

   ```java
   		 int threshold;             // 所能容纳的key-value对极限 
        final float loadFactor;    // 负载因子
        int modCount;  
        int size;  
   ```

   首先，Node[] table的初始化长度length(默认值是16)，Load factor为负载因子(默认值是0.75)，threshold是HashMap所能容纳的最大数据量的Node(键值对)个数。threshold = length * Load factor。也就是说，在数组定义好长度之后，负载因子越大，所能容纳的键值对个数越多。

   - 默认值

     ```java
         /**
          * 默认初始容量16，必须是2的幂
          */
         static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
     
         /**
          * 最大容量
          */
         static final int MAXIMUM_CAPACITY = 1 << 30;
     
         /**
          * 默认负载因子。当键值对的数量大于 CAPACITY * 0.75 时，就会触发扩容
          * 如无参的HashMap构造器，初始化容量为16，当键值对的数量大于 16 * 0.75 = 12时，就会触发扩容
          */
         static final float DEFAULT_LOAD_FACTOR = 0.75f;
     
         /**
          * 计数阈值。链表的元素大于8的时候，链表将转化为树
          */
         static final int TREEIFY_THRESHOLD = 8;
     
         /**
          * 计数阈值。resize操作时，红黑树的节点数量小于6时使用链表来代替树
          */
         static final int UNTREEIFY_THRESHOLD = 6;
     
         /**
          * 在转变成树之前，还会有一次判断，只有键值对数量大于 64 才会发生转换。
          * 这是为了避免在哈希表建立初期，多个键值对恰好被放入了同一个链表中而导致不必要的转化
          */
         static final int MIN_TREEIFY_CAPACITY = 64;
     ```

   - 属性

     ```java
         /**
          * 存储数据的哈希表，默认容量16。长度总是2的幂
          */
         transient Node<K,V>[] table;
     
         /**
          * Entry集合，主要用于迭代功能
          */
         transient Set<Map.Entry<K,V>> entrySet;
     
         /**
          * 实际存在的Node数量，不一定等于table的长度（初始化容量），甚至可能大于它。
          */
         transient int size;
     
         /**
          * HashMap结构被修改的次数，迭代过程中遵循Fail-Fast机制。
          * 保证多线程同时修改map时，能及时的发现（操作前备份的count和当前modCount不相等）并抛出异常终止操作。
          */
         transient int modCount;
     
         /**
          * 扩容阈值，超过(capacity * loadFactor)时，自动扩容容量为原来的二倍。 
          */
         int threshold;
     
         /**
          * 负载因子，可计算threshold：threshold = capacity * loadFactor
          */
         final float loadFactor;
     ```

#### 功能实现

- 确定数组中的索引位置

  好的 hash 算法，应当使元素尽量分布均匀，每个位置上的元素数量只有一个。HashMap 中使用的 hash 算法如下：

  ```java
  方法一：
  static final int hash(Object key) {   //jdk1.8 & jdk1.7
       int h;
       // h = key.hashCode() 为第一步 取hashCode值
       // h ^ (h >>> 16)  为第二步 高位参与运算
       return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
  方法二：
  static int indexFor(int h, int length) {  //jdk1.7的源码，jdk1.8没有这个方法，但是实现原理一样的
       return h & (length-1);  //第三步 取模运算
  }
  ```

  hash 算法本质上就是三步

  1. 取 key 的 hashCode 值

  2. hashCode 高位参与运算

     因为后面使用&运算符，而且操作数 length-1 只有低位有效。如果不加此步运算，很容易导致两个hashCode低位相同的 key 冲突。

  3. 取模运算

     当 length 总是 2 的 n 次方时，此表达式等价于对 length 取模，但是比取模具有更高的效率。

- 增加元素

  HashMap 的 put 方法可以通过下图来理解：

  ![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/d669d29c.png)

  具体源码如下：

  ```java
  public V put(K key, V value) {
    			// 对key的hashCode()做hash
          return putVal(hash(key), key, value, false, true);
  }
  
  final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                     boolean evict) {
          Node<K,V>[] tab; Node<K,V> p; int n, i;
    			// 步骤①：tab为空则创建
          if ((tab = table) == null || (n = tab.length) == 0)
              n = (tab = resize()).length;
    			// 步骤②：计算index，并对null做处理 
          if ((p = tab[i = (n - 1) & hash]) == null)
              tab[i] = newNode(hash, key, value, null);
          else {
              Node<K,V> e; K k;
            	// 步骤③：节点key存在，直接覆盖value
              if (p.hash == hash &&
                  ((k = p.key) == key || (key != null && key.equals(k))))
                  e = p;
              // 步骤④：判断该链为红黑树
              else if (p instanceof TreeNode)
                  e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
              // 步骤⑤：该链为链表
              else {
                  for (int binCount = 0; ; ++binCount) {
                      if ((e = p.next) == null) {
                          p.next = newNode(hash, key, value, null);
                          //链表长度大于8转换为红黑树进行处理
                          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                              treeifyBin(tab, hash);
                          break;
                      }
                      // key已经存在直接覆盖value
                      if (e.hash == hash &&
                          ((k = e.key) == key || (key != null && key.equals(k))))
                          break;
                      p = e;
                  }
              }
              if (e != null) { // existing mapping for key
                  V oldValue = e.value;
                  if (!onlyIfAbsent || oldValue == null)
                      e.value = value;
                  afterNodeAccess(e);
                  return oldValue;
              }
          }
          ++modCount;
     			// 步骤⑥：超过最大容量 就扩容
          if (++size > threshold)
              resize();
          afterNodeInsertion(evict);
          return null;
  }
  ```

- 扩容机制

  扩容(resize)就是重新计算容量，向HashMap对象里不停的添加元素，而HashMap对象内部的数组无法装载更多的元素时，对象就需要扩大数组的长度，以便能装入更多的元素。Java里的数组是无法自动扩容的，方法是使用一个新的数组代替已有的容量小的数组。

  ```java
  void resize(int newCapacity) {   //传入新的容量
       Entry[] oldTable = table;    //引用扩容前的Entry数组
       int oldCapacity = oldTable.length;         
       if (oldCapacity == MAXIMUM_CAPACITY) {  //扩容前的数组大小如果已经达到最大(2^30)了
           threshold = Integer.MAX_VALUE; //修改阈值为int的最大值(2^31-1)，这样以后就不会扩容了
           return;
       }
    
       Entry[] newTable = new Entry[newCapacity];  //初始化一个新的Entry数组
       transfer(newTable);                         //！！将数据转移到新的Entry数组里
       table = newTable;                           //HashMap的table属性引用新的Entry数组
       threshold = (int)(newCapacity * loadFactor);//修改阈值
  }
  ```

### LinkedHashMap 实现原理

大多数情况下，只要不涉及线程安全问题，Map基本都可以使用HashMap，不过HashMap有一个问题，就是迭代HashMap的顺序并不是HashMap放置的顺序，也就是无序。因此，Java 提供了 LinkedHashMap 来解决这个问题。

**在使用上，LinkedHashMap和HashMap的区别就是LinkedHashMap是有序的。** 上面这个例子是根据插入顺序排序，此外，LinkedHashMap还有一个参数决定**是否在此基础上再根据访问顺序(get,put)排序**。

#### 存储结构

LinkedHashMap的数据存储和HashMap的结构一样采用(数组+单向链表)的形式，只是在每个节点中增加了用于维护顺序的 `before`和`after`变量维护了一个双向链表来保存LinkedHashMap的存储顺序，当调用迭代器的时候不再使用HashMap的的迭代器，而是自己写迭代器来遍历这个双向链表即可。

```java
    /**
     * HashMap.Node subclass for normal LinkedHashMap entries.
     */
    static class LinkedHashMapEntry<K,V> extends HashMap.Node<K,V> {
        LinkedHashMapEntry<K,V> before, after;
        LinkedHashMapEntry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

所以 LinkedHashMap 单个节点所有的成员变量如下：

```java
LinkedHashMapEntry<K,V> before, after;
final int hash;
final K key;
V value;
Node<K,V> next;
```

其中 next 是用于维护 HashMap 中指定 table 位置上连接的 Entry 的顺序的。before/after 是用于维护 Entry 插入的先后顺序的。

#### 功能实现

LinkedHashMap 的基本实现思想就是——多态。它的 put 和 get 都是复用的父类中的方法，只是重写了 put 和 get 内部调用的某些方法。通过 head、tail 引用，记录双向链表的头和尾。

```java
    /**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMapEntry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMapEntry<K,V> tail;

    /**
     * The iteration ordering method for this linked hash map: <tt>true</tt>
     * for access-order, <tt>false</tt> for insertion-order.
     *
     * @serial
     */
    final boolean accessOrder;
```

#### 实现 LRU 缓存

LRU即Least Recently Used，最近最少使用，也就是说，当缓存满了，会优先淘汰那些最近最不常访问的数据。LinkedHashMap正好满足这个特性，当我们开启accessOrder为true时，**最新访问(get或者put(更新操作))的数据会被丢到队列的尾巴处，那么双向队列的头就是最不经常使用的数据了**。

此外，LinkedHashMap还提供了一个方法，这个方法就是为了我们实现LRU缓存而提供的，**removeEldestEntry(Map.Entry eldest) 方法。该方法可以提供在每次添加新条目时移除最旧条目的实现程序，默认返回 false**。

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
}
```

以下实现一个简陋的 LRU 缓存

```java
public class LRUCache extends LinkedHashMap
{
    public LRUCache(int maxSize)
    {
        super(maxSize, 0.75F, true);
        maxElements = maxSize;
    }

    protected boolean removeEldestEntry(java.util.Map.Entry eldest)
    {
        //逻辑很简单，当大小超出了Map的容量，就移除掉双向队列头部的元素，给其他元素腾出点地来。
        return size() > maxElements;
    }

    private static final long serialVersionUID = 1L;
    protected int maxElements;
}
```

### ConcurrentHashMap 实现原理

HashMap 是非线程安全的，其线程不安全主要体现在两个方面

- resize 死循环：单线程没问题，多线程 rehash 容易出现死循环
- 遍历时 fast-fail ：在遍历时修改 map 会报 CME

因此 Java 提供了 ConcurrentHashMap 来解决上述问题，保证线程安全。

#### Java 7 基于分段锁的ConcurrentHashMap

Java 7 中的ConcurrentHashMap的底层数据结构仍然是数组和链表。与HashMap不同的是，ConcurrentHashMap最外层不是一个大的数组，而是一个Segment的数组。每个Segment包含一个与HashMap数据结构差不多的链表数组。

<img src="http://www.jasongj.com/img/java/concurrenthashmap/concurrenthashmap_java7.png" alt="img" style="zoom:67%;" />



```java
/** Java 8 版本的 Segment **/
static class Segment<K,V> extends ReentrantLock implements Serializable {
        private static final long serialVersionUID = 2249069246763182397L;
        final float loadFactor;
        Segment(float lf) { this.loadFactor = lf; }
}
```

Segment继承自ReentrantLock，所以我们可以很方便的对每一个Segment上锁。

对于读操作，获取Key所在的Segment时，需要保证可见性，具体实现上可以使用**volatile**关键字，也可使用锁。但使用锁开销太大，而使用**volatile**时每次写操作都会让所有CPU内缓存无效，也有一定开销。

对于写操作，并不要求同时获取所有Segment的锁，因为那样相当于锁住了整个Map。它会先获取该Key-Value对所在的Segment的锁。获取锁时，并不直接使用lock来获取，事实上，它使用了自旋锁，如果tryLock获取锁失败，说明锁被其它线程占用，此时通过循环再次以tryLock的方式申请锁。如果在循环过程中该Key所对应的链表头被修改，则重置retry次数。如果retry次数超过一定值，则使用lock方法申请锁。这里使用自旋锁是因为自旋锁的效率比较高，但是它消耗CPU资源比较多，因此在自旋次数超过阈值时切换为互斥锁。

#### Java 8 基于 CAS 的 ConcurrentHashMap

Java 7为实现并行访问，引入了Segment这一结构，实现了分段锁，理论上最大并发度与Segment个数相等。Java 8为进一步提高并发性，摒弃了分段锁的方案，而是直接使用一个大的数组，采用`CAS + synchronized`来保证并发安全性。同时为了提高哈希碰撞下的寻址性能，Java 8在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为O(N)）转换为红黑树（寻址时间复杂度为O(long(N))）。

<img src="http://www.jasongj.com/img/java/concurrenthashmap/concurrenthashmap_java8.png" alt="img" style="zoom:67%;" />



































