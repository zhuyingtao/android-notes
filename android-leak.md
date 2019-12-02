### 内存泄漏实例

- 单例模式导致内存泄漏

  传入的 context，如果是 activity 就会造成泄漏

- 非静态内部类/匿名内部类导致内存泄漏

  非静态内部类/匿名内部类都会隐式持有外部类的引用，当它们的生命周期比外部类更长时，就有可能发生内存泄漏。

  AsyncTask/Thread

- Timer 和 TimerTask 导致内存泄漏

  Activity 销毁要立即 cancel 掉 Timer 和 TimerTask，以避免发生内存泄漏

- Handler 导致内存泄漏

  在 handler 中执行延时任务，持有 activity 引用

- 静态变量导致内存泄漏

  static Context mContext;

  mContext=this;

- 未取消注册或回调导致内存泄漏

- 资源未关闭或释放导致内存泄漏

- WebView 导致内存泄漏

### 内存泄漏检测

- Android Profiler
- Android Lint
- StrictMode
- LeakCanary