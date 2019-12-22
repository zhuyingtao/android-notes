### JVM

<img src="/Users/zyt/Library/Application Support/typora-user-images/image-20191119001446096.png" alt="image-20191119001446096" style="zoom: 50%;" />



[](https://www.zhihu.com/question/24304289)

- 程序计数器

  程序计数器，保存的是程序当前执行的指令的地址，类似于汇编语言中的 PC 寄存器。

  虽然JVM中的程序计数器并不像汇编语言中的程序计数器一样是物理概念上的CPU寄存器，但是JVM中的程序计数器的功能跟汇编语言中的程序计数器的功能在逻辑上是等同的。

  程序计数器是每个线程私有的。

- Java 栈

  Java栈中存放的是一个个的栈帧，每个栈帧对应一个被调用的方法。

- 本地方法栈

  与 Java 栈类似。区别在于本地方法栈是执行 native 方法的。

- 堆

  存储对象。线程共享。

- 方法区

  存储每个类的信息。还有一个重要区域是运行时常量池。