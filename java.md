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