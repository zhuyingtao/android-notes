### 1. 概述

ANR全称`Application Not Responding`，即应用未响应。简单来说就是主线程在特定的时间内没有做完特定的事情。

##### ANR 类型

- Input事件（包括按键和触摸事件）超过5s没有被处理完
- Service处理超时，前台20s，后台200s
- BroadcastReceiver处理超时，前台10s，后台60s
- ContentProvider执行超时，10s内没有处理完

### 2. ANR 分析

##### ANR 产生原因

- 主线程有耗时操作，如有复杂的layout布局，IO操作等
- 主线程被子线程同步锁block
- 主线程被Binder对端block
- Binder被占满导致主线程无法和SystemServer通信
- 得不到系统资源（CPU/RAM/IO）

##### ANR 日志

发生ANR的过程中，系统会自动的获取并打印日志信息，从这些log中我们可以获取ANR的类型，CPU的情况（CPU使用率过高有可能是CPU饥饿导致了ANR；CPU使用率过低说明主线程被block了），每种ANR类型在规定时间内执行的具体操作等等。

- 发生KeyDispatchTimeout类型的ANR日志中出现的关键字：Reason: Input dispatching timed out xxxx
- 发生ServiceTimeout类型的ANR日志中出现的关键字：Timeout executing service:/executing serviceXXX
- 发生BroadcastTimeout类型的ANR日志中出现的关键字：Timeout of broadcast XXX/Receiver during timeout:XXX/Broadcast of XXX
- 发生ProcessContentProviderPublishTimedOutLocked类型的ANR日志中出现的关键字：timeout publishing content providers

log输出以外，你会发现各个应用进程和系统进程的函数堆栈信息都输出到了一个/data/anr/traces.txt的文件中，这个文件是分析ANR原因的关键文件。

##### ANR 分析

分析ANR大致分为以下几个步骤:

1. 确定ANR发生时间,关键字:**am_anr**,**ANR in**
2. 查看ANR发生时打印的trace,文件目录:**/data/anr/traces.txt**
3. 查看系统耗时关键字:**binder_sample**,**dvm_lock_sample**,**am_lifecycle_sample**,**binder thread**
4. 结合源码和以上的信息进行分析

### 3. ANR 避免

为了避免 ANR 发生的概率，基本的思路就是将IO操作在工作线程来处理，减少其他耗时操作和错误操作

1：UI线程尽量只做跟UI相关的工作
2：耗时的工作（比如数据库操作，I/O，连接网络或 者别的有可能阻碍UI线程的操作） 把它放入单独的线程处理
3：尽量用Handler来处理UIthread和别的thread之间的交互