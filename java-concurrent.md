## Java 并发编程

### Thread 类

Java 中线程的状态可分为五种：New, Runnable, Running, Blocked, Dead

![img](assets/java-concurrent/820406-20160504003349169-1489389267.png)

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

![img](assets/java-concurrent/061046391107893.jpg)

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

### Callable/Future/FutureTask

Java 创建线程的两种方式：1. 继承 Thread类 2. 实现 Runnable 接口。这两种方式都有一个缺点：在执行完任务后无法获取执行结果，只能通过共享变量和线程间通信来实现，比较麻烦。

后来 Java 提供了 Callable 和 Future，它们可以在执行完任务后获得结果。

##### Callable

```java
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

怎么使用呢？一般是配合 ExecutorService 来使用的。

```java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
Future<?> submit(Runnable task);
```

##### Future

Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果。

```java
public interface Future<V> {
    // 取消任务，参数表示是否允许取消正在执行的任务
    boolean cancel(boolean mayInterruptIfRunning);
    // 任务是否被取消成功
    boolean isCancelled();
    // 任务是否已经完成
    boolean isDone();
    // 获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回
    V get() throws InterruptedException, ExecutionException;
    // 同上，并且加了一个等待时间，如果在等待时间内还没获取到结果，则直接返回 null
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

##### FutureTask

FutureTask 是 Future 接口的具体实现

```java
public class FutureTask<V> implements RunnableFuture<V>

public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```

可以看出 RunnableFuture 继承了 Runnable 接口和 Future 接口，而 FutureTask 实现了 RunnableFuture 接口。所以它既可以作为 Runnable 被线程执行，又可以作为 Future 得到 Callable 的返回值。

FutureTask 提供了2 个构造器：

```java
public FutureTask(Callable<V> callable) {
}
public FutureTask(Runnable runnable, V result) {
}
```

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

### 悲观锁与乐观锁

#### 悲观锁

总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。

Java中`synchronized`和`ReentrantLock`等独占锁就是悲观锁思想的实现。

悲观锁适用于写比较多的场景。

#### 乐观锁

总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据。乐观锁适用于多读的应用类型，这样可以提高吞吐量。

Java中`java.util.concurrent.atomic`包下面的原子变量类就是使用了乐观锁的一种实现方式**CAS**实现的。

乐观锁适用于写比较少的情况下（多读场景），省去了锁的开销，加大了系统的吞吐量。

#### CAS

CAS 全称是 compare and swap，它是一种原子操作，同时 CAS 是一种乐观机制。java.util.concurrent 包很多功能都是建立在 CAS 之上，如 ReenterLock 内部的 AQS，各种原子类，其底层都用 CAS来实现原子操作。

无锁编程，即不使用锁的情况下实现多线程之间的变量同步，也就是在没有线程被阻塞的情况下实现变量的同步，所以也叫非阻塞同步（Non-blocking Synchronization）。**CAS算法**涉及到三个操作数

- 需要读写的内存值 V
- 旧的预期值 A
- 拟写入的新值 B

当且仅当 V 的值等于 A时，CAS通过硬件级别的原子方式用新值B来更新V的值，否则不会执行任何操作（比较和替换是一个原子操作）。一般情况下是一个**自旋操作**，即**不断的重试**。

Java 并没有直接实现 CAS，CAS 相关的实现是通过 C++ 内联汇编的形式实现的。Java 代码需通过 JNI 才能调用，位于 unsafe.cpp。

CAS 问题：

- ABA 问题

  加入 version 字段来记录更改的版本，避免并发操作带来的问题。

- 自选问题

  给定限定的自旋次数，防止进入死循环。但是，如果 CAS 一直操作不成功，会造成长时间原地自旋，给 CPU 带来大的执行开销。

#### CAS 与 synchronized 的使用场景

- 对于资源竞争较少（线程冲突较轻）的情况，使用synchronized同步锁进行线程阻塞和唤醒切换以及用户态内核态间的切换操作额外浪费消耗cpu资源；而CAS基于硬件实现，不需要进入内核，不需要切换线程，操作自旋几率较少，因此可以获得更高的性能。

- 对于资源竞争严重（线程冲突严重）的情况，CAS自旋的概率会比较大，从而浪费更多的CPU资源，效率低于synchronized。

CAS适用于写比较少的情况下（多读场景，冲突一般较少），synchronized适用于写比较多的情况下（多写场景，冲突一般较多）

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

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    .....
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
    ...
}
```

下面解释下一下构造器中各个参数的含义：

- corePoolSize：核心池的大小，这个参数跟后面讲述的线程池的实现原理有非常大的关系。在创建了线程池后，默认情况下，线程池中并没有任何线程，而是等待有任务到来才创建线程去执行任务，除非调用了prestartAllCoreThreads()或者prestartCoreThread()方法，从这2个方法的名字就可以看出，是预创建线程的意思，即在没有任务到来之前就创建corePoolSize个线程或者一个线程。默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；

- maximumPoolSize：线程池最大线程数，这个参数也是一个非常重要的参数，它表示在线程池中最多能创建多少个线程；

- keepAliveTime：表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；

- unit：参数keepAliveTime的时间单位，有7种取值，在TimeUnit类中。

- workQueue：一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对线程池的运行过程产生重大影响，一般来说，这里的阻塞队列有以下几种选择：

  ```java
  ArrayBlockingQueue;
  LinkedBlockingQueue;
  SynchronousQueue;
  ```

- threadFactory：线程工厂，主要用来创建线程；

- handler：表示当拒绝处理任务时的策略。

在ThreadPoolExecutor类中有几个非常重要的方法：

```java
execute()
submit()
shutdown()
shutdownNow()
```

- execute()方法实际上是Executor中声明的方法，在ThreadPoolExecutor进行了具体的实现，这个方法是ThreadPoolExecutor的核心方法，通过这个方法可以向线程池提交一个任务，交由线程池去执行。
- submit()方法是在ExecutorService中声明的方法，在AbstractExecutorService就已经有了具体的实现，在ThreadPoolExecutor中并没有对其进行重写，这个方法也是用来向线程池提交任务的，但是它和execute()方法不同，它能够返回任务执行的结果，去看submit()方法的实现，会发现它实际上还是调用的execute()方法，只不过它利用了Future来获取任务执行结果。
- shutdown()和shutdownNow()是用来关闭线程池的。

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

### 































