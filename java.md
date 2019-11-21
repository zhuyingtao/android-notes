# Java

## Java 集合

- [Java容器类](http://alexyyek.github.io/2015/04/06/Collection/)

### transient 关键字的用法

此关键字与序列化相关。

实现了 Serilizable 接口的类，在属性前添加 transient 关键字，则在序列化对象的时候，这个属性就不会被序列化。

1. 只能修饰属性，不能修饰方法和类，不能修饰局部变量
2. 静态变量无论是否被修饰，均不能被序列化
3. 若实现的是Externalizable接口，则没有任何东西可以自动序列化，需要在writeExternal方法中进行手工指定所要序列化的变量，这与是否被transient修饰无关

[Java transient关键字使用小记](https://www.cnblogs.com/lanxuezaipiao/p/3369962.html)

### volatile 关键字的用法

此关键字与多线程同步相关。

#### 1. 内存模型

- CPU - Cache - Memory
- 缓存不一致问题

#### 2. 并发编程

- 原子性：一个操作或者多个操作，要么全部执行且不会被打断，要么就都不执行
- 可见性：多个线程访问同一个变量时，一个线程修改了变量值，其他线程能够立即看到这个修改的值
- 有序性：程序的执行顺序按照代码的先后顺序执行。JVM 编译时会进行指令重排，程序执行顺序不一定一致但结果一致。指令重排不会影响单个线程的执行，但会影响多线程执行的正确性。

```java
boolean inited = false;

//线程1:
context = loadContext();   //语句1
inited = true;             //语句2
 
//线程2:
while(!inited ){
  sleep()
}
doSomethingwithconfig(context);
```

总结，并发程序正确地执行，必须要保证**原子性、可见性以及有序性**。只要有一个没有被保证，就有可能会导致程序运行不正确。

#### 3. Java 并发编程

- 原子性

  在 Java 中，对基本数据类型的变量的读取和赋值操作是原子性操作（而且必须是将数字赋值给某个变量，变量之间的相互赋值不是原子操作）。要实现更大范围操作的原子性，用 synchronized 和 Lock 来实现。

- 可见性

  在 Java 中，用 volatile 关键字来保证可见性。

  当一个共享变量（属性，静态属性）被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。普通的共享变量不能保证可见性，因为什么时候被写入内存是不确定的。

  另外，通过synchronized和Lock也能够保证可见性，synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。因此可以保证可见性。

- 有序性

  在 Java 中，用 volatile 关键字来保证有序性。另外也可以通过 synchronized 和 Lock 来保证有序性。

#### 4. volatile 关键字

一个共享变量（类的属性、静态属性）被 volatile 修饰之后，那么就具备了两层含义：

1. 保证了可见性
2. 保证了有序性，即禁止了指令重排
3. 不能保证原子性

实现原理：

> 观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令

#### 5. 使用场景

synchronized关键字是防止多个线程同时执行一段代码，那么就会很影响程序执行效率，而volatile关键字在某些情况下性能要优于synchronized，但是要注意volatile关键字是无法替代synchronized关键字的

- 状态标记

  ```java
  volatile boolean inited = false;
  
  //线程1:
  context = loadContext();  
  inited = true;            
   
  //线程2:
  while(!inited ){
  sleep()
  }
  doSomethingwithconfig(context);
  ```

- Double-Check

  ```java
  public class Singleton {
  
  		private static volatile Singleton instance = null;
  		
      private Singleton() {
      }
      
      public static Singleton getInstance() {
          if (instance == null) {
              synchronized (Singleton.class) {
                  if (instance == null) {
                      instance = new Singleton();
                  }
              }
          }
          return instance;
      }
  }
  ```

  使用 volatile 的原因：

  new 操作不是原子的，实际包含 3 条汇编指令：new,dup,init。当一个线程执行了 new() 时，对应的汇编指令可能发生了重排序，导致第 3 步完成，而第 2 步还没有进行，但是这时仍然会释放锁。一个新的线程到来，发现 instance 已经不为 null 了，直接返回。这时的 instance 是不完整的则会报错。也就是说，会有一个【instance 已经不为 null 但是仍然没有完成初始化】的中间状态。

  使用 volatile 关键字修饰，对它的写操作就会有一个内存屏障，限制了指令重排序。这样，在它的赋值完成之前，就不会有读操作。保证了在一个写操作完成之前，不会调用读操作。

[Java并发编程：volatile关键字解析](https://www.cnblogs.com/dolphin0520/p/3920373.html)

### Object 类

- getClass()

- hashCode()

- equals(Object)
  重写了equals 方法，必须重写 hashCode 方法（若确定没有用到类似 Map 的地方，则不必重写 hashCode，Map诸多方法使用 hashCode 来判断两个对象是否相等）

- clone()
  先判断是否可 clone，再调用 internalClone

- internalClone()
  native 方法

- toString()

- finalize()
  在垃圾回收器清除对象之前调用。在实际应用中，不要依赖 finalize 方法回收任何资源，因为很难知道这个方法什么时候才能调用。

- wait()/wait(long)/wait(long,int)

  导致线程进入等待状态，直到它被其他线程通过notify()或者notifyAll唤醒。该方法只能在**同步方法**中调用。如果当前线程不是锁的持有者，该方法抛出一个IllegalMonitorStateException异常。

- notify()

  **随机选择一个**在该对象上调用wait方法的线程，解除其阻塞状态。该方法只能在**同步方法**或**同步块**内部调用。如果当前线程不是锁的持有者，该方法抛出一个IllegalMonitorStateException异常。

- notifyAll()

  解除**所有**那些在该对象上调用wait方法的线程的阻塞状态。该方法只能在**同步方法**或**同步块**内部调用。如果当前线程不是锁的持有者，该方法抛出一个IllegalMonitorStateException异常。

[Object.wait()与Object.notify()的用法](https://www.cnblogs.com/xwdreamer/archive/2012/05/12/2496843.html)

### 强引用/软引用/弱引用/虚引用

- 强引用

  正常情况下都是强引用，只要对象有强引用，必定不会被回收，即使 OOM

- 软引用

  用来描述一些有用但不是必须的对象，用 SoftReference 类来表示。对于软引用关联着的对象，只有内存不足的时候 JVM 才会回收该对象。实用场景：网页缓存、图片缓存等。

- 弱引用

  也是用来描述非必须对象，用 WeakReference 类表示。对于弱引用关联着的对象，当 JVM 进行垃圾回收时，无论内存是否充足，都会被回收。

- 虚引用

  虚引用不影响对象的生命周期，用 PhantomReference 类表示。对于虚引用关联着的对象，相当于没有引用与之关联一样，在任何时候都可能被垃圾回收器回收。唯一的用处：能在对象被GC时收到系统通知。

### 重载/重写

### 垃圾回收机制

1. 垃圾检测

   - 引用计数法

     通过引用计数来判断一个对象是否可以被回收。优点：实现简单，效率较高；缺点：无法解决循环引用的问题

     Java 不用，Python 用

   - 可达性分析法

     通过一系列“GC Roots”对象作为起点进行搜索。如果在“GC Roots”和一个对象之间没有可达路径，则称该对象是不可达的。

2. 垃圾回收

   - Mark-Sweep（标记-清除）算法

     标记阶段，标记出所有需要被回收的对象；清除阶段，回收被标记的对象所占用的空间。

     此算法实现起来比较容易，但是容易产生内存碎片。

   - Copying（复制）算法

     将内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块内存用完了，将存活的对象复制到另一块上面，然后把此块内存清空。

     此算法实现也比较简单，运行高效且不容易产生内存碎片。但内存使用率降低，能够使用的内存缩减到原来的一半。

   - Mark-Compact（标记-整理）算法

     标记阶段，和 Mark-Sweep 算法一样。完成标记后，将存活的对象都向一端移动，然后清理掉端边界以外的内存。

   - Generational Collection（分代收集）算法

     目前大部分 JVM 使用此算法。它的核心思想是根据对象存活的生命周期将内存划分为若干个不同区域。一般情况下分为老年代（Tenured Generation）和新生代（Young Generation）。根据不同代的特点选择最合适的回收算法。

     老年代：

     每次垃圾回收只有少量对象需要被回收。一般使用 Mark-Compact 算法。

     新生代：

     每次垃圾回收都有大量对象需要被回收。一般使用 Copying 算法。但不是根据 1：1 来划分新生代内存空间的。

     一般分为一块较大的 Eden 空间和两块较小的 Survivor 空间，每次使用 Eden 空间和其中一块 Survivor 空间，当进行回收时，将这两块上还存活的对象复制到另一块 Survivor 空间上，然后清理掉这两块空间。

     ![img](https://images0.cnblogs.com/i/288799/201406/181512325519249.jpg)

     对象主要分配在新生代的Eden Space和From Space，少数情况下会直接分配在老年代。如果新生代的 Eden Space和From Space的空间不足，则会发起一次GC，如果进行了GC之后，Eden Space和From Space能够容纳该对象就放在Eden Space和From Space。在GC的过程中，会将Eden Space和From Space中的存活对象移动到To Space，然后将Eden Space和From Space进行清理。如果在清理的过程中，To Space无法足够来存储某个对象，就会将该对象移动到老年代中。在进行了GC之后，使用的便是Eden space和To Space了，下次GC时会将存活对象复制到From Space，如此反复循环。当对象在Survivor区躲过一次GC的话，其对象年龄便会加1，默认情况下，如果对象年龄达到15岁，就会移动到老年代中。

3. 典型的垃圾回收器

   - Serial/Serial Old

   - ParNew

   - Parallel Scavenge

   - CMS

   - G1

     G1 是当今收集器技术发展最前沿的成果