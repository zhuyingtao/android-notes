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

  导致线程进入等待状态，直到它被其他线程通过notify()/notifyAll或者一段时间后超时唤醒。该方法只能在**同步方法**中调用。如果当前线程不是锁的持有者，该方法抛出一个IllegalMonitorStateException异常。

  wait()方法是一个本地方法，其底层是通过一个叫做监视器锁的对象来完成的。

  调用 wait()方法后，线程会释放锁。

  一个通过 wait()方法阻塞的线程，必须满足被唤醒后（超时唤醒或者 notify/notifyAll）再次竞争到锁才会继续执行。

- notify()

  **随机选择一个**在该对象上调用wait方法的线程，解除其阻塞状态。该方法只能在**同步方法**或**同步块**内部调用。如果当前线程不是锁的持有者，该方法抛出一个IllegalMonitorStateException异常。

- notifyAll()

  解除**所有**那些在该对象上调用wait方法的线程的阻塞状态。该方法只能在**同步方法**或**同步块**内部调用。如果当前线程不是锁的持有者，该方法抛出一个IllegalMonitorStateException异常。

[Object.wait()与Object.notify()的用法](https://www.cnblogs.com/xwdreamer/archive/2012/05/12/2496843.html)

### Thread 类

Java 中线程的状态可分为五种：New, Runnable, Running, Blocked, Dead

![img](https://images2015.cnblogs.com/blog/820406/201605/820406-20160504003349169-1489389267.png)

- sleep()

  让当前线程暂停指定的时间，交出 CPU 权限，让 CPU 去执行其他的线程。sleep 是让线程进入阻塞状态。

  sleep() 与 wait()的区别：

  wait () 方法只能在同步方法中调用，sleep() 方法可以直接调用。wait() 方法需要释放锁，sleep() 方法不需要释放锁。

- yield()

  让当前线程交出 CPU 权限，让 CPU 去执行其他的线程。跟 sleep 类似，同样不会释放锁。但是 yield 不能控制交出 CPU 的时间。yield 并不是让线程进入阻塞状态，而是让线程重回就绪状态。

- join()

  在 main 线程中，调用 thread.join()，则 main 线程会等待 thread 线程执行完毕或等待一段时间后再继续执行。

  也就是说，join() 方法的作用是父线程等待子线程执行完毕后再执行。

  源码实现如下：

  ```java
  public final synchronized void join(long millis)
        throws InterruptedException {
            long base = System.currentTimeMillis();
            long now = 0;
    
            if (millis < 0) {
                throw new IllegalArgumentException("timeout value is negative");
            }
   
            if (millis == 0) {
               while (isAlive()) {
                   wait(0);
               }
            } else {
               while (isAlive()) {
                   long delay = millis - now;
                   if (delay <= 0) {
                       break;
                   }
                   wait(delay);
                   now = System.currentTimeMillis() - base;
               }
            }
        }
  ```

  通过源码可以看出，join()方法就是通过 wait()来将线程阻塞。根据 isAlive()方法判定 join 的线程是否还在运行，如果还在运行，则继续等待，直到 join 的线程执行完毕，当前线程才能继续执行。

  - 为什么 wait()一定要放在循环中？

    《Effective Java》中说：永远不要在循环之外调用 wait 方法。

    wait 方法的官方注释是这样说的：

    ```java
    /**  A thread can also wake up without being notified, interrupted, or
         * timing out, a so-called <i>spurious wakeup</i>.  While this will rarely
         * occur in practice, applications must guard against it by testing for
         * the condition that should have caused the thread to be awakened, and
         * continuing to wait if the condition is not satisfied.  In other words,
         * waits should always occur in loops, like this one:
         * <pre>
         *     synchronized (obj) {
         *         while (&lt;condition does not hold&gt;)
         *             obj.wait(timeout);
         *         ... // Perform action appropriate to condition
         *     }
         * </pre>
    */     
    ```

    大致意思是 thread 有可能被**虚假唤醒(spurious wakeup)**，此时如果用 if 的话，唤醒后线程会从 wait 之后的代码开始运行，有可能因为不满足判定条件而执行了后面的代码，从而出现错误。如果用 while 的话，唤醒后会重新判断条件，不满足则继续执行wait。

- interrupt()

  中断线程，单独调用interrupt方法可以使得处于**阻塞状态**的线程抛出一个异常。通过interrupt方法和isInterrupted()方法来停止**正在运行**的线程。

- setDaemon()/isDaemon()

  用来设置线程是否成为守护线程和判断线程是否是守护线程。

  守护线程和用户线程的区别在于：守护线程依赖于创建它的线程，而用户线程则不依赖。比如，如果在main线程中创建了一个守护线程，当main方法运行完毕之后，守护线程也会随着消亡。而用户线程则不会，用户线程会一直运行直到其运行完毕。在JVM中，像垃圾收集器线程就是守护线程。

![img](https://images0.cnblogs.com/blog/288799/201409/061046391107893.jpg)

### synchronized

多个线程同时访问临界资源（也称为共享资源），就可能产生线程安全问题。解决线程安全问题，需要同步互斥访问，也就是在访问临界资源的代码前面加上一个锁，当访问完临界资源后释放锁，让其他线程继续访问。

##### 类锁和对象锁

对象锁：每个对象都有一把锁（monitor），修饰方法和代码块。

类锁：实际上是 class 对象的锁，因为一个类只会有一个 class 对象，所以也称为类锁，修饰静态方法和静态代码块。也可以当作对象锁使用。

Java 中，提供了两种方法实现同步互斥访问：synchronized 和 Lock。

##### synchronized 实现原理：

synchronized 代码块通过反编译字节码可以看到，多了 monitorenter 和 monitorexit 两条指令。

### Lock

##### 为什么需要 Lock？

synchronized 释放锁有两种情况：1. 代码执行完 2. 出现异常。如果线程进入了阻塞状态，依然不释放锁，其他线程只能等。Lock 可以实现等待一段时间或响应中断。

synchronized 不能区分读写操作，当是多个读操作时，虽然不会冲突，但只能同步操作。Lock 可以区分读写，对多个读操作不做同步互斥处理。

总之，Lock 提供了更多的功能，更加灵活，是为了提升效率，解决 synchronized 的缺陷而出现的。

##### Lock 与 synchronized

- synchronized 是 Java 语言的关键字，是内置特性。Lock 不是内置的，是一个类。
- synchronized 不需要手动释放锁。Lock 必须手动释放锁。
- synchronized 线程不能响应中断，等待的线程会一直等待下去。Lock 可以响应中断。
- synchronized 不能判断有没有获得锁。Lock 可以知道获取锁的状态。
- Lock 可以提高多个读操作的效率。

##### concurrent.lock 包下的类

- Lock 接口

  ```java
  public interface Lock {
      void lock();
      void lockInterruptibly() throws InterruptedException;
      boolean tryLock();
      boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
      void unlock();
      Condition newCondition();
  }
  ```

- ReentrantLock 类，可重入锁，是唯一实现了 Lock 接口的类

  lock 的使用方法，一定要在 finally 中释放锁。

  ```java
  Lock lock = ...;
  lock.lock();
  try{
      //处理任务
  }catch(Exception ex){
       
  }finally{
      lock.unlock();   //释放锁
  }
  ```

- ReadWriteLock 接口

  ```java
  public interface ReadWriteLock {
      /**
       * Returns the lock used for reading.
       *
       * @return the lock used for reading.
       */
      Lock readLock();
   
      /**
       * Returns the lock used for writing.
       *
       * @return the lock used for writing.
       */
      Lock writeLock();
  }
  ```

- ReentrantReadWriteLock 类，可重入读写锁

##### 不同锁的概念

- 可重入锁

  synchronized 和 ReentrantLock 都是此类锁。基于线程分配锁，而不是基于方法调用。比如，methodA 中需要锁，methodB也需要锁，在 methodA 中调用 methodB，由于 methodA 已经获得了锁，methodB 就不需要重新去申请锁了。

- 可中断锁

  可以响应中断的锁。Lock 是此类锁。lockInterruptibly() 可以实现中断等待的线程。

- 公平锁

  以请求锁的顺序来获取锁。synchronized 是非公平锁。ReentrantLock 默认是非公平锁，但是可以通过参数设置为公平锁。

- 读写锁

### ThreadLocal 类

ThreadLocal，可以认为是线程本地变量。通过 ThreadLocal 类，可以为成员变量在每一个线程中创建一个副本，那么每一个线程都可以访问自己的副本变量。

```java
public T get() { }
public void set(T value) { }
public void remove() { }
protected T initialValue() { }
```

首先，在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）。

初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。

##### 使用场景

最常见的 ThreadLocal 使用场景为 用来解决数据库连接、session 管理等。

### 同步容器

集合框架四大接口：List、Set、Queue、Map，其中，List、Set、Queue 继承了 Collection 接口。

ArrayList、LinkedList、HashMap 这些容器都是非线程安全的，如果有多个线程并发访问，就会出现问题。为了方便开发者，Java 提供了同步容器。

同步容器有 2 类：

- Vector、Stack、HashTable

  Vector 与 ArrayList 类似，但是 Vector 中的方法都是 syncrhonized，即进行了同步处理。Stack 继承了 Vector 类。

  HashTable 与 HashMap 类似，但HashTable 进行了同步处理。

- Collections.synchronizedCollection/List/Map() 方法

同步容器的缺陷：

- 性能问题

- 并不一定安全

  同步容器保证了操作的串行，但是当线程进行复合操作时，仍然是非线程安全的。Vector 的 get(i) 和 remove(i) 操作，虽然能保证只有一个线程访问 vector，但是可能因为 i 的值而导致数组越界问题。

- CME 异常

  iterator 的 next 方法中会调用 checkForComodification 方法，当 expectedModCount 和 modCount 不一致时（比如在遍历时调用 list.remove），就会报 CME 异常。

  解决方法：

  单线程：使用 iterator.remove方法，此方法中会操作 expectedModCount = modCount。

  多线程：使用 iterator 迭代的时候加锁；使用并发容器 CopyOnWriteArrayList 代替 ArrayList 和 Vector。

### 并发容器

##### CopyOnWriteArrayList

CopyOnWrite 即写时复制。就是往一个容器中添加元素的时候，不直接往当前容器添加，而是先 copy，往新的容器里添加，添加完元素后，再把原引用指向新的容器。这样的好处是可以进行并发的读，而不需要加锁。

CopyOnWrite 容器用于读多写少的并发场景。比如白名单、黑名单之类

##### ConcurrentHashMap

### 阻塞队列

阻塞队列，会对当前线程产生阻塞，比如一个线程从一个空的阻塞队列中取元素，此时线程会被阻塞直到阻塞队列中有了元素。当队列中有元素后，被阻塞的线程会自动被唤醒（不需要我们编写代码去唤醒）。这样提供了极大的方便性。

- ArrayBlockingQueue
- LinkedBlockingQueue
- PriorityBlockingQueue
- DelayQueue

### 线程池（重新阅读）

[Java并发编程：线程池的使用](https://www.cnblogs.com/dolphin0520/p/3932921.html)

##### ThreadPoolExecutor

Executor -> ExecutorService -> AbstractExecutorService -> ThreadPoolExecutor

### 并发编程辅助类

##### CountDownLatch

实现类似计数器的功能，所有线程执行完毕后才会继续执行。

```java
public CountDownLatch(int count) {  };  //参数count为计数值
public void await() throws InterruptedException { };   //调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  //和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public void countDown() { };  //将count值减1
```

##### CyclicBarrier

字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。

```java
public CyclicBarrier(int parties, Runnable barrierAction) {}
public CyclicBarrier(int parties) {}
public int await() throws InterruptedException, BrokenBarrierException { };
public int await(long timeout, TimeUnit unit)throws InterruptedException,BrokenBarrierException,TimeoutException { };
```

##### Semaphore

字面意思为 信号量，Semaphore可以控制同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。Semaphore其实和锁有点类似，它一般用于控制对某组资源的访问权限。

```java
public Semaphore(int permits) {          //参数permits表示许可数目，即同时可以允许多少线程进行访问
    sync = new NonfairSync(permits);
}
public Semaphore(int permits, boolean fair) {    //这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
    sync = (fair)? new FairSync(permits) : new NonfairSync(permits);
}
public void acquire() throws InterruptedException {  }     //获取一个许可
public void acquire(int permits) throws InterruptedException { }    //获取permits个许可
public void release() { }          //释放一个许可
public void release(int permits) { }    //释放permits个许可
```

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
     
     可作为 GC Roots 的对象：
     
     - 虚拟机栈（栈帧中本地变量表）中引用的对象
     - 方法区中静态属性引用的对象
     - 方法区中常量引用的对象
     - 本地方法栈中Native方法引用的对象

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
     
     ![image-20200305232705354](assets/java/image-20200305232705354.png)
     
     Java堆的内存划分如图所示，分别为年轻代、Old Memory（老年代）、Perm（永久代）。其中在Jdk1.8中，永久代被移除，使用MetaSpace代替。
     
     1、新生代：
     （1）使用复制清除算法（Copinng算法），原因是年轻代每次GC都要回收大部分对象。新生代里面分成一份较大的Eden空间和两份较小的Survivor空间。每次只使用Eden和其中一块Survivor空间，然后垃圾回收的时候，把存活对象放到未使用的Survivor（划分出from、to）空间中，清空Eden和刚才使用过的Survivor空间。
     （2）分为Eden、Survivor From、Survivor To，比例默认为8：1：1
     （3）内存不足时发生Minor GC
     
     2、老年代：
     （1）采用标记-整理算法（mark-compact），原因是老年代每次GC只会回收少部分对象。
     3、Perm：用来存储类的元数据，也就是方法区。
     （1）Perm的废除：在jdk1.8中，Perm被替换成MetaSpace，MetaSpace存放在本地内存中。原因是永久代进场内存不够用，或者发生内存泄漏。
     （2）MetaSpace（元空间）：元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。

3. 典型的垃圾回收器

   - Serial/Serial Old

   - ParNew

   - Parallel Scavenge

   - CMS

   - G1

     G1 是当今收集器技术发展最前沿的成果

### 抽象类与接口

- 语法层面上的区别
  1. 抽象类可以提供方法的实现细节，接口只能存在 public abstract 方法
  2. 抽象类成员变量可以是各种类型的，接口成员变量只能是 public static final 类型的
  3. 抽象类能含有静态代码块及静态方法，接口不能有
  4. 一个类只能继承一个抽象类，一个类可以实现多个接口
- 设计层面上的区别
  1. 抽象类是对一种事物的抽象，接口是对行为的抽象
  2. 抽象类作为很多子类的父类，是一种模板式设计；接口是一种行为规范，是一种辐射式设计
  

##### Java 8 中接口的变化

- 新增静态方法

  静态方法使用 static 关键字声明，实现类不能覆盖静态方法。

- 新增默认方法

  默认方法使用 default 关键字声明，实现类可以选择不实现这个方法。

### 内部类

内部类四种形式：成员内部类、局部内部类、匿名内部类、静态内部类

1. 为什么成员内部类可以无条件访问外部类的成员？

   通过反编译，查看字节码文件可以看出，编译器会默认为成员内部类添加一个指向外部类对象的引用 **this$0**，还会在无参的构造器上添加一个参数，传入外部类的引用，所以没有创建外部类对象，也就无法创建成员内部类对象了。

2. 为什么局部内部类和匿名内部类只能访问局部 final 变量和形参

   为了保证变量**生命周期**的一致性，Java 采用了复制的手段来解决这个问题。对于局部变量，如果这个变量的值可以在编译期间确定，则编译器会在内部类的常量池中添加一个相等的值或者直接嵌入到字节码中；对于形参，编译器会在内部类的构造器中加入此参数，进行 copy。但又为了保证**数据**一致性，所以必须将变量限制为 final 变量。

3. 静态内部类使用场景

   - 内部类需要脱离外部类对象来创建对象，如各种 builder 设计模式
   - 避免内部类使用过程中出现内存泄漏，如 AsyncTask

### 浅拷贝和深拷贝

- 浅拷贝：在拷贝对象时，对于基本数据类型会复制一份，对于引用的变量只是对引用进行拷贝，不对引用对象拷贝
- 深拷贝：在拷贝对象时，对基本数据类型和引用的对象都会进行拷贝

#### Fail-Fast & Fail-Safe

- Fail-Fast

  Fail-Fast 机制，通俗的讲就是在程序设计时先考虑异常情况，一旦发生异常，直接停止并上报。这种机制在日常的编程中随处可见。Java 的集合类也使用了 fail-fast 机制进行设计，一旦使用不当，就会抛出异常。

  典型的场景，在 foreach 循环里进行元素的 remove/add 操作，就会抛出 CME 异常。

  ```java
  Exception in thread "main" java.util.ConcurrentModificationException
  at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
  at java.util.ArrayList$Itr.next(ArrayList.java:859)
  at com.hollis.ForEach.main(ForEach.java:22)
  ```

  异常原理：

  1. `foreach`语法糖经过反编译以后，可以看出是依赖 while 和 Iterator 来实现的。

  2. 分析上面的调用栈，可以看出是`checkForComodification`发生了异常，此方法是在 iterator.next()中调用的，具体实现如下：

     ```java
     final void checkForComodification() {
         if (modCount != expectedModCount)
             throw new ConcurrentModificationException();
     }
     ```

  3. 该方法中由于 modCount 和 expectedModCound 不相等而抛出了异常。

     modCount是ArrayList中的一个成员变量，它表示该集合实际被修改的次数。

     expectedModCount 是 ArrayList中的一个内部类——Itr中的成员变量，该类实现了 Iterator 接口。

     而 remove()方法只对 modCount 进行了操作。

     ```java
     private void fastRemove(int index) {
         modCount++;
         int numMoved = size - index - 1;
         if (numMoved > 0)
             System.arraycopy(elementData, index+1, elementData, index,
                              numMoved);
         elementData[--size] = null; // clear to let GC do its work
     }
     ```

     这就导致 modCount != expectedModCount 而导致异常。

  Java 里 util 包下的集合类都是 fail-fast 机制的。

- Fail-Safe

  为了避免出发 fail-fast 机制，Java 中提供了一些采用 fail-safe 机制的集合类。

  concurrent 包下的集合类都是 fail-safe 机制的，可以在多线程下并发使用，并发修改，同时也可以在 foreach 中进行 add/remove。

  fail-safe集合的所有对集合的修改都是先拷贝一份副本，然后在副本集合上进行的，并不是直接对原集合进行修改。并且这些修改方法，如add/remove都是通过加锁来控制并发的。这种策略叫做 Copy-On-Write。

  CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。CopyOnWrite容器是一种读写分离的思想，读和写不同的容器。

  存在两个问题：

  - 创建 copy 需要额外的空间和时间上的开销
  - 不能保证遍历的是最新的内容

#### 静态分派和动态分派

变量的静态类型：引用类型，不会被改变，在编译期可知

变量的动态类型：实例类型，可变化，在运行期可知

- 静态分派

  应用场景：方法重载（Overload），根据参数的静态类型确定执行哪个重载方法

- 动态分派

  应用场景：方法重写（Override），根据变量的动态类型确定执行哪个方法