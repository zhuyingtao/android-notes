## Android ArrayMap

在Android开发时，我们使用的大部分都是Java的api，比如HashMap这个api，使用率非常高，但是对于Android这种对内存非常敏感的移动平台，很多时候使用一些java的api并不能达到更好的性能，相反反而更消耗内存，所以针对Android这种移动平台，也推出了更符合自己的api，比如SparseArray、ArrayMap用来代替HashMap在有些情况下能带来更好的性能提升。

### SparseArray

### 实现原理

SparseArray比HashMap更省内存，在某些条件下性能更好，主要是因为它避免了对key的自动装箱（int转为Integer类型）。它内部则是通过两个数组来进行数据存储的，一个存储key，另外一个存储value。可以看到，它只能存储 key 为 int 的类型。

```java
private int[] mKeys;
private Object[] mValues;
```

![SparseArray](assets/android-bundle/SparseArray.jpg)

在存储和读取数据时，使用的是二分查找法

```java
 public void put(int key, E value) {
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);
        ...
        }
 public E get(int key, E valueIfKeyNotFound) {
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);
        ...
        }
```

#### 应用场景

SparseArray 由于各个操作都依赖二分查找，所以在数据量大的情况下性能下降比较明显。如果满足以下两个条件，可以使用 SparseArray 代替 HashMap

- 数据量不大，最好在千级以内
- key 值必须为 int 类型

### ArrayMap

首先看一下 ArrayMap 的数据结构

```java
public final class ArrayMap<K, V> implements Map<K, V> {

    private static final boolean CONCURRENT_MODIFICATION_EXCEPTIONS = true;
    
    private static final int BASE_SIZE = 4;  // 容量增量的最小值
    private static final int CACHE_SIZE = 10; // 缓存数组的上限

    static Object[] mBaseCache; //用于缓存大小为4的ArrayMap
    static int mBaseCacheSize;
    static Object[] mTwiceBaseCache; //用于缓存大小为8的ArrayMap
    static int mTwiceBaseCacheSize;

    final boolean mIdentityHashCode;
    int[] mHashes;         //由key的hashcode所组成的数组
    Object[] mArray;       //由key-value对所组成的数组，是mHashes大小的2倍
    int mSize;             //成员变量的个数
}
```

ArrayMap中包含了两个小数组。 第一个数组（Hash-Array）按顺序包含指定的哈希键。 第二个数组（Key Value Array）根据第一个数组存储对象的键和值。

- mHashes是一个记录所有key的hashcode值组成的数组，是从小到大的排序方式；
- mArray是一个记录着key-value键值对所组成的数组，是mHashes大小的2倍；

![arrayMap](assets/android-bundle/arrayMap.jpg)

其中mSize记录着该ArrayMap对象中有多少对数据，执行put()或者append()操作，则mSize会加1，执行remove()，则mSize会减1。mSize往往小于mHashes.length，如果mSize大于或等于mHashes.length，则说明mHashes和mArray需要扩容。

ArrayMap类有两个非常重要的静态成员变量mBaseCache和mTwiceBaseCacheSize，用于ArrayMap所在进程的全局缓存功能：

- mBaseCache：用于缓存大小为4的ArrayMap，mBaseCacheSize记录着当前已缓存的数量，超过10个则不再缓存；
- mTwiceBaseCacheSize：用于缓存大小为8的ArrayMap，mTwiceBaseCacheSize记录着当前已缓存的数量，超过10个则不再缓存。

为了减少频繁地创建和回收Map对象，ArrayMap采用了两个大小为10的缓存队列来分别保存大小为4和8的Map对象。

使用 SparseArray 与使用 ArrayMap 相比，效率更高。

- key 只能是基本类型，避免自动装箱带来的内存消耗，不需要 hash 数组，减少内存消耗
- key 为其他类型时使用 ArrayMap

### 总结

从以下几个角度总结一下：

- 数据结构
  - ArrayMap和SparseArray采用的都是两个数组，Android专门针对内存优化而设计的
  - HashMap采用的是数据+链表+红黑树
- 内存优化
  - ArrayMap比HashMap更节省内存，综合性能方面在数据量不大的情况下，推荐使用ArrayMap；
  - Hash需要创建一个额外对象来保存每一个放入map的entry，且容量的利用率比ArrayMap低，整体更消耗内存
  - SparseArray比ArrayMap节省1/3的内存，但SparseArray只能用于key为int类型的Map，所以int类型的Map数据推荐使用SparseArray；
- 性能方面：
  - ArrayMap查找时间复杂度O(logN)；ArrayMap增加、删除操作需要移动成员，速度相比较慢，对于个数小于1000的情况下，性能基本没有明显差异
  - HashMap查找、修改的时间复杂度为O(1)；
  - SparseArray适合频繁删除和插入来回执行的场景，性能比较好
- 缓存机制
  - ArrayMap针对容量为4和8的对象进行缓存，可避免频繁创建对象而分配内存与GC操作，这两个缓存池大小的上限为10个，防止缓存池无限增大；
  - HashMap没有缓存机制
  - SparseArray有延迟回收机制，提供删除效率，同时减少数组成员来回拷贝的次数
- 扩容机制
  - ArrayMap是在容量满的时机触发容量扩大至原来的1.5倍，在容量不足1/3时触发内存收缩至原来的0.5倍，更节省的内存扩容机制
  - HashMap是在容量的0.75倍时触发容量扩大至原来的2倍，且没有内存收缩机制。HashMap扩容过程有hash重建，相对耗时。所以能大致知道数据量，可指定创建指定容量的对象，能减少性能浪费。
- 并发问题
  - ArrayMap是非线程安全的类，大量方法中通过对mSize判断是否发生并发，来决定抛出异常。但没有覆盖到所有并发场景，比如大小没有改变而成员内容改变的情况就没有覆盖
  - HashMap是在每次增加、删除、清空操作的过程将modCount加1，在关键方法内进入时记录当前mCount，执行完核心逻辑后，再检测mCount是否被其他线程修改，来决定抛出异常。这一点的处理比ArrayMap更有全面。

### Bundle

Bundle 内部是由 ArrayMap 实现的。Bundle 的使用场景为小数据量，一般在 Activity 之间只传递 10 个以内的数据，相比之下，使用 ArrayMap 保存数据，其操作速度和内存占用上都具有优势。故使用 Bundle 来传递数据，可以保证更快的速度和更少的内存消耗。

此外，Android 中使用 Intent 传递数据，要求数据为基本类型或可序列化类型。HashMap 使用的是 Serializable 进行序列化，使用起来简单，但是开销很大，序列化和反序列化过程需要大量的 I/O 操作；而 Bundle 使用的是 Parcelable 进行序列化，是 Android 平台推荐的序列化方式，虽然使用起来麻烦些，但是开销更小，效率很高。故为了更快速地进行数据的序列化和反序列化，Android 系统封装了 Bundle 类，供传输较少量数据的时候使用。

