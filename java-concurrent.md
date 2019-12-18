## Java 并发编程

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
- 进行比较的值 A
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































