## Android 内存分析

### Android Profiler - Memory

#### 名词解释

- Allocations

  标识某个类的实例数量

- Native Size / Shallow Size / Retained Size

  Native Size，Shallow Size，Retained Size这几组数据分别意味着什么呢？通过一个例子来说明。

  我们用下图来表示某段 Heap Dump 记录的应用内存状态。注意红色的节点，在这个示例中，这个节点所代表的对象从我们的工程中引用了 Native 对象:

  ![image-20220627142828572](assets/Untitled/image-20220627142828572.png)在 Android 8.0 之后，使用 Bitmap 便可能产生此类情景，因为 Bitmap 会把像素信息存储在原生内存中来减少 JVM 的内存压力。

  **Shallow Size**：这列数据其实非常简单，就是对象本身消耗的内存大小，在上图中，即为红色节点自身所占内存（以字节为单位）。

  **Native Size**：同样也很简单，它是类对象所引用的 Native 对象 (蓝色节点) 所消耗的内存大小（以字节为单位）。

  **Retained Size**：稍复杂些，它是下图中所有橙色节点的大小（以字节为单位）。

  ![image-20220627142955751](assets/Untitled/image-20220627142955751.png)

  由于一旦删除红色节点，其余的橙色节点都将无法被访问，这时候它们就会被 GC 回收掉。从这个角度上讲，它们是被红色节点所持有的，因此被命名为 "Retained Size"。