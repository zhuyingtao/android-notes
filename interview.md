## Interview Questions

### Java

- 反射、RTTI
- JVM
  - 内存模型
    - 堆
    - 栈
    - 方法区
      - 常量池
  - 垃圾回收
    - 垃圾检测
      - 引用计数法
      - 可达性分析法
    - 垃圾处理
      - 标记-清除法
      - 复制法
  - 类加载
    - 加载机制
- 并发
  - 锁的类型
    - 可重入锁
  - sychronized 关键字
  - 线程池
  - Callable
  - ThreadLocal
  - Thread
    - run()/start()
  - 进程与线程
- 容器
  - 基本容器
  - List
    - ArrayList/LinkedList
  - Map
    - HashMap 原理
    - ConcurrentHashMap 原理
    - LinkedHashMap 原理
  - concurrent 包中的容器
- 设计模式
  - 单例模式
    - DCL
      - volatile
      - synchronized
- 类
  - Object
    - ==/equals/hashcode
    - wait()/notify()
  - 强/弱/软/虚引用
- 关键字
  - final
  - transient

### Android

- Activity
  - 生命周期
  - startActivity() 原理
- Framework
  - 应用启动流程
  - Handler机制
    - Handler 源码
    - Looper 源码
    - Message 源码
    - MessageQueue 源码
- IPC
  - AIDL
  - Binder 机制
- XML
  
  - include/viewstub/merge
- Event
  - touch 事件传递分发
    - 拦截
    - 消费
- ANR
  - 类型
  - 分析
  - 处理
- View 
  
  - 绘制流程
- 内存
  - 内存泄漏
    - 场景
    - 原因
    - 解决方法
    
    

- 数据结构和算法
  - 常见的排序算法，top k问题
  - Java
    - 类：初始化顺序
    - 集合
      - 常用的集合类
      - HashMap/LinkedHashMap区别
      - HashMap实现原理
    - 类型擦除
    - 单例模式
      - 几种写法
      - double-check 原理
    - 并发编程
      - volatile 关键字用法以及原理
    - Object 类
      - 哪些常用方法
      - wait/notify
- 操作系统与开发工具
  - Linux常用命令
    - 常用的命令
    - grep vim
  - Git
    - [文件状态/工作区](http://www.cnblogs.com/polk6/p/git-fileStatus.html)
    - 常用命令 修改上一次的提交/git rebase
  - 正则表达式
    - 手机号匹配
  - 加密
    - hash算法：MD5/SHA
    - 加密算法：AES/DES
- Android
  - Activity
    - [启动模式和intentFlags](https://blog.csdn.net/singwhatiwanna/article/details/9294285) 应用场景
    - taskAffinity
  - Service
    - Service与Thread关系，为什么不直接在Activity里创建Thread？
    - 如何保证service在后台不被kill 提升优先级/onDestroy重启/onStartCommand返回值
    - 如何与Activity之间通信
  - BroadcastReceiver
    - 静态注册/动态注册
  - 进程间通信
    - 哪些方式
    - AIDL
  - View
    - 绘制流程 onMeasure/onLayout/onDraw
    - 自定义View的方式 自绘控件/组合控件/继承控件
  - 事件分发机制 View/ViewGroup
    - 事件类型
    - 事件分发顺序
    - 涉及的方法
    - onTouch 和 onTouchEvent 区别
  - 异步消息处理机制
    - Handler/Looper/Message
    - 为什么主线程不会因为Looper.loop()里的死循环卡死？
  - 内存泄露
    [内存泄露场景及修复方法](https://wiki.n.miui.com/pages/viewpage.action?pageId=69213399) 泄露场景/如何解决
  - ANR
    [ANR问题分析](https://wiki.n.miui.com/pages/viewpage.action?pageId=66675971) 定义/类型/产生原因/如何分析/解决与避免
  - 动画
    - 动画分类及用法
  - xml
    - permission 和 uses-permission
  - 框架
    - 网络框架
    - MVC/MVP/MVVM
  - 加密
    - base64
    - 对称加密和非对称加密
    - 常用的对称加密算法
  - 扩展
    - Rxjava/Kotlin/Testing
  - 工具
    - MAT
    - Systrace
    - TraceView
    - Profiler

#### 现场编程

- [使用两个栈实现队列](https://www.nowcoder.com/profile/314508/codeBookDetail?submissionId=1227977)
  push()/pop()/健壮性

```java
import java.util.Stack;



public class Solution {

    Stack<Integer> stack1 = new Stack<Integer>();

    Stack<Integer> stack2 = new Stack<Integer>();



    public void push(int node) {

        stack1.push(node);

    }



    public int pop() {

        if(stack1.empty()&&stack2.empty()){

            throw new RuntimeException("Queue is empty!");

        }

        if(stack2.empty()){

            while(!stack1.empty()){

                stack2.push(stack1.pop());

            }

        }

        return stack2.pop();

    }

}
```

