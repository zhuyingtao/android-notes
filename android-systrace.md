## Systrace

### Systrace 简介

Systrace 是 Android 4.1 中新增的性能数据采样和分析工具。它可帮助开发者收集 Android 关键子系统（如 SurfaceFlinger/SystemServer/Kernel/Input/Display 等 Framework 部分关键模块、服务，View系统等）的运行信息，从而帮助开发者更直观的分析系统瓶颈，改进性能。

Systrace 的功能包括跟踪系统的 I/O 操作、内核工作队列、CPU 负载以及 Android 各个子系统的运行状况等。在 Android 平台中，它主要由3部分组成：

- **内核部分**：Systrace 利用了 Linux Kernel 中的 ftrace 功能。所以，如果要使用 Systrace 的话，必须开启 kernel 中和 ftrace 相关的模块。
- **数据采集部分**：Android 定义了一个 Trace 类。应用程序可利用该类把统计信息输出给ftrace。同时，Android 还有一个 atrace 程序，它可以从 ftrace 中读取统计信息然后交给数据分析工具来处理。
- **数据分析工具**：Android 提供一个 systrace.py（ python 脚本文件，位于 Android SDK目录/platform-tools/systrace 中，其内部将调用 atrace 程序）用来配置数据采集的方式（如采集数据的标签、输出文件名等）和收集 ftrace 统计数据并生成一个结果网页文件供用户查看。 从本质上说，Systrace 是对 Linux Kernel中 ftrace 的封装。应用进程需要利用 Android 提供的 Trace 类来使用 Systrace。

### Systrace 使用方法

使用 Systrace 流程一般是这样的：

- 手机准备好你要进行抓取的界面
- 点击开始抓取(命令行的话就是开始执行命令)
- 手机上开始操作(不要太长时间)
- 设定好的时间到了之后，会将生成 Trace.html 文件，使用 **Chrome** 将这个文件打开进行分析

抓取的话一般有两种方式：

1. 手机【开发者选项】中，进入【系统跟踪】，点击录制开始。
2. 使用命令行方式。命令行形式比较灵活，速度也比较快，一次性配置好后，以后再使用就会很快出结果。

所以推荐使用命令行形式。Systrace 工具在 Android-SDK 目录下的 platform-tools 里面,下面是简单的使用方法

```bash
$ cd android-sdk/platform-tools/systrace
$ python systrace.py
```

抓取结束后，会生成对应的 Trace.html 文件，注意这个文件只能被 Chrome 打开。不论使用那种工具，在抓取之前都可以选择参数，下面说一下这些参数的意思：

一般来说比较常用的是

1. -o : 指示输出文件的路径和名字
2. -t : 抓取时间
3. -b : 指定 buffer 大小
4. -a : 指定 app 包名

另外说一下 category tags

- **gfx** - Graphics
- **input** - Input
- **view** - View
- webview - WebView
- **wm** - Window Manager
- **am** - Activity Manager
- audio - Audio
- video - Video
- camera - Camera
- hal - Hardware Modules
- res - Resource Loading
- **dalvik** - Dalvik VM
- rs - RenderScript
- **sched** - CPU Scheduling
- **freq** - CPU Frequency
- **membus** - Memory Bus Utilization
- **idle** - CPU Idle
- **disk** - Disk input and output
- **load** - CPU Load
- **sync** - Synchronization Manager
- **workq** - Kernel Workqueues Note: Some trace categories are not supported on all devices. Tip: If you want to see the names of tasks in the trace output, you must include the sched category in your command parameters.

我们一般会把这个命令配置成Alias，配置如下：

```bash
alias st-start='python /sdk/platform-tools/systrace/systrace.py'  
alias st-start-gfx-trace = ‘st-start -t 8 gfx input view sched freq wm am hwui workq res dalvik sync disk load perf hal rs idle mmc’
```

### Systrace 分析

#### 线程状态查看

Systrace 会用不同的颜色来标识不同的线程状态，在每个方法上面都会有对应的线程状态来标识目前线程所处的状态。通过查看线程状态我们可以知道目前的瓶颈是什么, 是 CPU 执行慢还是因为 Binder 调用, 又或是进行 IO 操作, 又或是拿不到 CPU 时间片。

线程状态主要有下面几个

- 绿色：运行中，`Running`
- 蓝色：可运行，`Runnable`
- 白色：休眠中，`Sleeping`，可能是因为线程在互斥锁上被阻塞
- 橙色：不可中断的睡眠态 IO Block，`Uninterruptible Sleep|Block I/O`，线程在 I/O 上被阻塞
- 紫色：不可中断的睡眠态，`Uninterruptible Sleep`，线程在另一个内核操作（通常是内存管理）上被阻塞，一般是进入了内核态

#### 进程唤醒信息分析

Systrace 会标识出一个非常有用的信息，可以帮助我们进行跨进程调用相关的分析。

一个进程被唤醒的信息往往比较重要，知道他被谁唤醒，那么我们也就知道了他们之间的调用等待关系，如果出现一段比较长的 sleep 情况，然后被唤醒，那么我们就可以去看是谁唤醒了这个线程，对应的就可以查看唤醒者的信息，看看为什么唤醒者这么晚才唤醒。

#### 信息区数据解析



