## Preference

### SharedPreferences

- SharedPreferences是个单例，所以任意Context拿到的都是同一个实例。

- SharedPreferences在实例化的时候会把SharedPreferences对应的xml文件内容全部读取到内存。

- 对于非多进程兼容的SharedPreferences的读操作是从内存读取的，不涉及IO操作。写入的时候由于内存已经保存了完整的xml数据，然后新写入的数据也会同步更新到内存，所以无论是用commit还是apply都不会影响立即读取。

- 除非你需要关心xml是否写入文件成功，否则你应该在所有调用commit的地方改用apply。

- 我们需要对SharedPreferences在包装一层内存缓存来提高性能吗？完全不需要，因为SharedPreferences本身已经做了内存缓存。

#### commit 与 apply

相同点：

- 二者都是提交 Prefrence 修改数据
- 二者都是原子过程

不同点：

对于 commit()

- 会返回执行结果
- 是同步执行的

对于 apply()

- 不会返回执行结果
- 是异步执行

commit 与 apply 的声明：

```java
/**
  * Commit your preferences changes back from this Editor to the
  * {@link SharedPreferences} object it is editing.  This atomically
  * performs the requested modifications, replacing whatever is currently
  * in the SharedPreferences.
  *
  * <p>Note that when two editors are modifying preferences at the same
  * time, the last one to call commit wins.
  *
  * <p>If you don't care about the return value and you're
  * using this from your application's main thread, consider
  * using {@link #apply} instead.
  *
  * @return Returns true if the new values were successfully written
  * to persistent storage.
  */
  boolean commit();

/**
 * <p>Unlike {@link #commit}, which writes its preferences out
 * to persistent storage synchronously, {@link #apply}
 * commits its changes to the in-memory
 * {@link SharedPreferences} immediately but starts an
 * asynchronous commit to disk and you won't be notified of
 * any failures.  If another editor on this
 * {@link SharedPreferences} does a regular {@link #commit}
 * while a {@link #apply} is still outstanding, the
 * {@link #commit} will block until all async commits are
 * completed as well as the commit itself.
 *
 * <p>As {@link SharedPreferences} instances are singletons within
 * a process, it's safe to replace any instance of {@link #commit} with
 * {@link #apply} if you were already ignoring the return value.
 *
 * <p>You don't need to worry about Android component
 * lifecycles and their interaction with <code>apply()</code>
 * writing to disk.  The framework makes sure in-flight disk
 * writes from <code>apply()</code> complete before switching
 * states.
 *
 * <p class='note'>The SharedPreferences.Editor interface
 * isn't expected to be implemented directly.  However, if you
 * previously did implement it and are now getting errors
 * about missing <code>apply()</code>, you can simply call
 * {@link #commit} from <code>apply()</code>.
 */
 void apply();
```

commit 与 apply 的具体实现：

```java
public boolean commit() {
    MemoryCommitResult mcr = commitToMemory();
    SharedPreferencesImpl.this.enqueueDiskWrite(mcr, null /* sync write on this thread okay */);
    try {
        mcr.writtenToDiskLatch.await();
    } catch (InterruptedException e) {
        return false;
    }
    notifyListeners(mcr);
    return mcr.writeToDiskResult;
}

public void apply() {
    final MemoryCommitResult mcr = commitToMemory();
    final Runnable awaitCommit = new Runnable() {
        public void run() {
            try {
                mcr.writtenToDiskLatch.await();
            } catch (InterruptedException ignored) {
            }
        }
    };
    QueuedWork.add(awaitCommit);
    Runnable postWriteRunnable = new Runnable() {
        public void run() {
            awaitCommit.run();
            QueuedWork.remove(awaitCommit);
        }
    };
    SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);
    // Okay to notify the listeners before it's hit disk
    // because the listeners should always get the same
    // SharedPreferences instance back, which has the
    // changes reflected in memory.
    notifyListeners(mcr);
}

private void enqueueDiskWrite(final MemoryCommitResult mcr,final Runnable postWriteRunnable) {
    final Runnable writeToDiskRunnable = new Runnable() {
        public void run() {
            synchronized (mWritingToDiskLock) {
                writeToFile(mcr);
            }
            synchronized (SharedPreferencesImpl.this) {
                mDiskWritesInFlight--;
            }
            if (postWriteRunnable != null) {
                postWriteRunnable.run();
            }
        }
    };
    final boolean isFromSyncCommit = (postWriteRunnable == null);
    // Typical #commit() path with fewer allocations, doing a write on
    // the current thread.
    if (isFromSyncCommit) {
        boolean wasEmpty = false;
        synchronized (SharedPreferencesImpl.this) {
            wasEmpty = mDiskWritesInFlight == 1;
        }
        if (wasEmpty) {
            writeToDiskRunnable.run();
            return;
        }
		QueuedWork.singleThreadExecutor().execute(writeToDiskRunnable);
}
```

这两个方法都是首先修改内存中缓存的mMap的值，然后将数据写到磁盘中。它们的主要区别是commit会等待写入磁盘后再返回，而apply则在调用写磁盘操作后就直接返回了，**但是这时候可能磁盘中数据还没有被修改。**



[SP vs MMKV vs DataStore](https://www.jianshu.com/p/e2113f501cf9)

[MMKV 使用与原理](https://www.jianshu.com/p/13b889028326)

##### 重点阅读：

https://mp.weixin.qq.com/s/w3uZR6us1MMVYzfzD1PCiQ