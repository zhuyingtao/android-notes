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

