#### 概述

ANR全称`Application Not Responding`，意思就是程序未响应。如果一个应用无法响应用户的输入，系统就会弹出一个ANR对话框。

#### 出现场景

- 主线程被IO操作（从4.0之后网络IO不允许在主线程中）阻塞。
- 主线程中存在耗时的计算
- 主线程中错误的操作，比如Thread.wait或者Thread.sleep等

Android系统会监控程序的响应状况，一旦出现下面两种情况，则弹出ANR对话框

- 应用在**5秒**内未响应用户的输入事件（如按键或者触摸）
- BroadcastReceiver未在**10秒**内完成相关的处理

#### 如何避免

基本的思路就是将IO操作在工作线程来处理，减少其他耗时操作和错误操作

- 使用[AsyncTask](http://droidyue.com/blog/2014/11/08/bad-smell-of-asynctask-in-android/)处理耗时IO操作。
- 使用Thread或者HandlerThread时，调用Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND)设置优先级，否则仍然会降低程序响应，因为默认Thread的优先级和主线程相同。
- 使用[Handler](http://droidyue.com/blog/2014/12/28/in-android-handler-classes-should-be-static-or-leaks-might-occur/)处理工作线程结果，而不是使用Thread.wait()或者Thread.sleep()来阻塞主线程。
- Activity的onCreate和onResume回调中尽量避免耗时的代码
- BroadcastReceiver中onReceive代码也要尽量减少耗时，建议使用IntentService处理。

#### 如何定位

如果开发机器上出现问题，我们可以通过查看`/data/anr/traces.txt`即可，最新的ANR信息在最开始部分。