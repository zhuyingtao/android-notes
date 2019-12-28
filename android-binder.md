## Android Binder 机制

Binder作为Android系统提供的一种IPC机制，无论从系统开发还是应用开发，都是Android系统中最重要的组成，也是最难理解的一块知识点。

### 1. 概述

Android系统中，每个应用程序是由Android的`Activity`，`Service`，`Broadcast`，`ContentProvider`这四大组件的中一个或多个组合而成，这四组件所涉及的多进程间的通信底层都是依赖于Binder IPC机制。所以要想理解这些组件之间的原理，必须要理解 Binder 通信机制。

#### Android 为什么选择 Binder？

Android 基于 Linux 内核的，可以使用 Linux 原有的一些 IPC 机制，比如管道、消息队列、共享内存、socket 等方式，但 Android 还是采用了 Binder 作为主要机制，有以下几方面的原因。

1. 性能方面

   管道模式/消息队列采用存储-转发方式，至少需要两次内存拷贝过程。socket 传输效率低，开销大。共享内存虽然无需拷贝，但控制复杂，难以使用。Binder 只需要有一次内存拷贝过程即可，性能高效。

2. 安全方面

   传统的 IPC 通信方式没有任何措施，基本依靠上层协议，无法确认对方可靠的身份。Android 为每个应用程序分配了自己的 UID，进程 UID 可以用来鉴别身份，安全性高。

3. 稳定方面

   Binder 基于 C/S 架构，架构清晰、职责明确又相互独立，稳定性更好。共享内存控制复杂，难以使用。

### 2. 原理

#### 2.1 Linux IPC 原理

从进程角度来看Linux 传统 IPC 机制

![img](https://pic3.zhimg.com/80/v2-38e2ea1d22660b237e17d2a7f298f3d6_hd.jpg)

两个进程间通信，是利用进程间可共享的内核空间来完成底层通信工作的，进程内的用户空间采用系统调用（ioctl等方法）跟内核空间进行交互。这里涉及到一些进程通信的基本概念：

- 进程隔离

  进程隔离，就是进程与进程间的内存是不共享的。

- 用户空间/内核空间

  操作系统核心是内核，独立于普通的应用程序，可以访问受保护的内存空间，也可以访问底层硬件设备。为了保护用户进程不能直接操作内核，保证内核的安全，操作系统从逻辑上将虚拟空间划分为用户空间和内核空间。内核空间是系统内核运行的内存空间，用户空间是用户程序运行的空间。为了保证安全性，它们之间是隔离的。

- 系统调用：用户态/内核态

  用户程序不可避免需要访问内核资源，比如文件操作、网络访问等等。为了突破隔离限制，就需要借助系统调用来实现。系统调用是用户空间访问内核空间的唯一方式，保证了所有资源访问都是在内核的控制下进行的，避免了用户程序对系统资源的越权访问。

  当进程执行系统调用而进入内核代码执行时，则处于内核态，此时CPU 处于特权级最高的（0 级）内核代码执行。担当进程执行应用代码时，则处于用户态。此时 CPU 处于特权级最低的（3 级）用户代码执行。

理解了上面的概念后，再来具体讨论传统的进程通信方式：

![img](https://pic1.zhimg.com/80/v2-aab2affe42958a659ea8a517ffaff5a0_hd.jpg)

这种方式有两个问题：

1. 性能低，一次数据传递需要 2 次数据拷贝。
2. 接收数据的缓存区由数据接收进程提供，接收进程不知道需要多大的内存空间来存放传递的数据，解决方法只能是尽可能大的开辟内存空间或者先通过调用 API 获取数据大小。这两种做法浪费空间或时间。

#### 2.2 Binder IPC 原理

#### 动态内核可加载模块

跨进程通信是需要内核空间做支持的。Binder 并不是 Linux 系统内核的一部分，那怎么办呢？得益于 Linux 的动态内核可加载模块机制（Loadable Kernel Module, LKM）：模块是具有独立功能的程序，它可以被单独编译但是不能独立运行，它在运行时被链接到内核作为内核的一部分。这样，Android 系统就可以动态添加一个内核模块运行在内核空间，用户进程之间通过这个内核模块作为桥梁来实现通信。

> 在 Android 系统中，这个运行在内核空间，负责各个用户进程通过 Binder 实现通信的内核模块就叫 **Binder 驱动**（Binder Dirver）。

#### 内存映射

有了 Binder 驱动，那 Android 进程间是如何实现通信的呢？这里还要再提另一个概念：内存映射。

Binder IPC 机制中涉及到的内存映射通过 mmap() 来实现，mmap() 是操作系统中一种内存映射的方法。内存映射简单的讲就是将用户空间的一块内存区域映射到内核空间。映射关系建立后，用户对这块内存区域的修改可以直接反应到内核空间；反之内核空间对这段区域的修改也能直接反应到用户空间。

内存映射能减少数据拷贝次数，实现用户空间和内核空间的高效互动。两个空间各自的修改能直接反映在映射的内存区域，从而被对方空间及时感知。也正因为如此，内存映射能够提供对进程间通信的支持。

#### Binder IPC 实现原理

![img](https://pic4.zhimg.com/80/v2-cbd7d2befbed12d4c8896f236df96dbf_hd.jpg)

一次完整的 Binder IPC 过程通常是这样的：

1. 首先 Binder 驱动在内核空间创建一个数据接收缓存区；
2. 接着在内核空间开辟一块内核缓存区，建立**内核缓存区**和**内核中数据接收缓存区**之间的映射关系，以及**内核中数据接收缓存区**和**接收进程用户空间地址**的映射关系；
3. 发送方进程通过系统调用 copy_from_user() 将数据 copy 到内核中的**内核缓存区**，由于内核缓存区和接收进程的用户空间存在内存映射，因此也就相当于把数据发送到了接收进程的用户空间，这样便完成了一次进程间的通信。

#### Binder 通信模型

Binder通信采用 C/S 架构，从组件视角来说，包含Client、Server、ServiceManager以及Binder驱动，其中ServiceManager用于管理系统中的各种服务。ServiceManager 和 Binder 驱动由系统提供，而 Client、Server 由应用程序来实现。Client、Server 和 ServiceManager 均是通过系统调用 open、mmap 和 ioctl 来访问设备文件 /dev/binder，从而实现与 Binder 驱动的交互来间接的实现跨进程通信。

![ServiceManager](http://gityuan.com/images/android-arch/IPC-Binder.jpg)

可以看出无论是注册服务和获取服务的过程都需要ServiceManager，此处的Service Manager是指Native层的ServiceManager（C++）。ServiceManager是整个Binder通信机制的大管家，是Android进程间通信机制Binder的守护进程，要掌握Binder机制，首先需要了解系统是如何首次[启动Service Manager](http://gityuan.com/2015/11/07/binder-start-sm/)。当Service Manager启动之后，Client端和Server端通信时都需要先[获取Service Manager](http://gityuan.com/2015/11/08/binder-get-sm/)接口，才能开始通信服务。

图中Client/Server/ServiceManage之间的相互通信都是基于Binder机制。既然基于Binder机制通信，那么同样也是C/S架构，则图中的3大步骤都有相应的Client端与Server端。

1. **[注册服务(addService)](http://gityuan.com/2015/11/14/binder-add-service/)**：Server进程要先注册Service到ServiceManager。该过程：Server是客户端，ServiceManager是服务端。
2. **[获取服务(getService)](http://gityuan.com/2015/11/15/binder-get-service/)**：Client进程使用某个Service前，须先向ServiceManager中获取相应的Service。该过程：Client是客户端，ServiceManager是服务端。
3. **使用服务**：Client根据得到的Service信息建立与Service所在的Server进程通信的通路，然后就可以直接与Service交互。该过程：client是客户端，server是服务端。

#### Binder 通信过程

总结 Binder 通信过程：

1. 首先，一个进程使用 BINDER_SET_CONTEXT_MGR 命令通过 Binder 驱动将自己注册成为 ServiceManager；
2. Server 通过驱动向 ServiceManager 中注册 Binder（Server 中的 Binder 实体），表明可以对外提供服务。驱动为这个 Binder 创建位于内核中的实体节点以及 ServiceManager 对实体的引用，将名字以及新建的引用打包传给 ServiceManager，ServiceManger 将其填入查找表。
3. Client 通过名字，在 Binder 驱动的帮助下从 ServiceManager 中获取到对 Binder 实体的引用，通过这个引用就能实现和 Server 进程的通信。

Client、Server、ServiceManager、Binder 驱动这几个组件在通信过程中扮演的角色就如同互联网中服务器（Server）、客户端（Client）、DNS域名服务器（ServiceManager）以及路由器（Binder 驱动）之前的关系。

![img](https://pic4.zhimg.com/80/v2-67854cdf14d07a6a4acf9d675354e1ff_hd.jpg)

#### Binder 通信中的代理模式

![img](https://pic2.zhimg.com/80/v2-13361906ecda16e36a3b9cbe3d38cbc1_hd.jpg)

#### Binder 的完整定义

现在我们可以对 Binder 做个更加全面的定义了：

- 从进程间通信的角度看，Binder 是一种进程间通信的机制；
- 从 Server 进程的角度看，Binder 指的是 Server 中的 Binder 实体对象；
- 从 Client 进程的角度看，Binder 指的是对 Binder 代理对象，是 Binder 实体对象的一个远程代理
- 从传输过程的角度看，Binder 是一个可以跨进程传输的对象；Binder 驱动会对这个跨越进程边界的对象对一点点特殊处理，自动完成代理对象和本地对象之间的转换。

### 3. 实现

通常我们在做开发时，实现进程间通信用的最多的就是 AIDL。当我们定义好 AIDL 文件，在编译时编译器会帮我们生成代码实现 IPC 通信。借助 AIDL 编译以后的代码能帮助我们进一步理解 Binder IPC 的通信原理。但是无论是从可读性还是可理解性上来看，编译器生成的代码对开发者并不友好。为了便于理解，下面来手动编写代码来实现跨进程调用，实际上就是实现 AIDL 转化为 java 代码的过程。

#### 3.1 各个类的职责描述

在正式编码实现跨进程调用之前，先介绍下实现过程中用到的一些类。了解了这些类的职责，有助于我们更好的理解和实现跨进程通信。

- **IBinder** : IBinder 是一个接口，代表了一种跨进程通信的能力。只要实现了这个借口，这个对象就能跨进程传输。
- **IInterface** : IInterface 代表的就是 Server 进程对象具备什么样的能力（能提供哪些方法，其实对应的就是 AIDL 文件中定义的接口）
- **Binder** : Java 层的 Binder 类，代表的其实就是 Binder 本地对象。BinderProxy 类是 Binder 类的一个内部类，它代表远程进程的 Binder 对象的本地代理；这两个类都继承自 IBinder, 因而都具有跨进程传输的能力；实际上，在跨越进程的时候，Binder 驱动会自动完成这两个对象的转换。
- **Stub** : AIDL 的时候，编译工具会给我们生成一个名为 Stub 的静态内部类；这个类继承了 Binder, 说明它是一个 Binder 本地对象，它实现了 IInterface 接口，表明它具有 Server 承诺给 Client 的能力；Stub 是一个抽象类，具体的 IInterface 的相关实现需要开发者自己实现。

#### 3.2 实现过程讲解

一次跨进程通信必然会涉及到两个进程，在这个例子中 RemoteService 作为服务端进程，提供服务；ClientActivity 作为客户端进程，使用 RemoteService 提供的服务。

![img](https://pic2.zhimg.com/80/v2-7ca457119bd700a5acf7f69bb0c07e51_hd.jpg)

服务端实现：

根据前面介绍过的 IInterface，它代表的是服务端具备什么样的能力。因此我们定义一个 BookManager 接口继承自 IInterface，标明服务端具备的能力。

```java
/**
 * 这个类用来定义服务端 RemoteService 具备什么样的能力
 */
public interface BookManager extends IInterface {

    void addBook(Book book) throws RemoteException;
}
```

只定义服务端具备什么样的能力是不够的，既然是跨进程调用，那么接下来我们得实现一个跨进程调用对象 Stub。Stub 继承 Binder, 说明它是一个 Binder 本地对象；实现 IInterface 接口，表明具有 Server 承诺给 Client 的能力；Stub 是一个抽象类，具体的 IInterface 的相关实现需要调用方自己实现。

```java
public abstract class Stub extends Binder implements BookManager {

    ...

    public static BookManager asInterface(IBinder binder) {
        if (binder == null)
            return null;
        IInterface iin = binder.queryLocalInterface(DESCRIPTOR);
        if (iin != null && iin instanceof BookManager)
            return (BookManager) iin;
        return new Proxy(binder);
    }

    ...

    @Override
    protected boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
        switch (code) {

            case INTERFACE_TRANSACTION:
                reply.writeString(DESCRIPTOR);
                return true;

            case TRANSAVTION_addBook:
                data.enforceInterface(DESCRIPTOR);
                Book arg0 = null;
                if (data.readInt() != 0) {
                    arg0 = Book.CREATOR.createFromParcel(data);
                }
                this.addBook(arg0);
                reply.writeNoException();
                return true;

        }
        return super.onTransact(code, data, reply, flags);
    }

    ...
}
```

首先说明 asInterface(IBinder binder)

当 Client 端在创建和服务端的连接，调用 bindService 时需要创建一个 ServiceConnection 对象作为入参。在 ServiceConnection 的回调方法 onServiceConnected 中 会通过这个 asInterface(IBinder binder) 拿到 BookManager 对象，这个 IBinder 类型的入参 binder 是驱动传给我们的，正如你在代码中看到的一样，方法中会去调用 binder.queryLocalInterface() 去查找 Binder 本地对象，如果找到了就说明 Client 和 Server 在同一进程，那么这个 binder 本身就是 Binder 本地对象，可以直接使用。否则说明是 binder 是个远程对象，也就是 BinderProxy。因此需要我们创建一个代理对象 Proxy，通过这个代理对象来是实现远程访问。

接下来就需要实现这个代理类 Proxy，既然是代理类自然需要实现 BookManager 接口

```java
public class Proxy implements BookManager {

    ...

    public Proxy(IBinder remote) {
        this.remote = remote;
    }

    @Override
    public void addBook(Book book) throws RemoteException {

        Parcel data = Parcel.obtain();
        Parcel replay = Parcel.obtain();
        try {
            data.writeInterfaceToken(DESCRIPTOR);
            if (book != null) {
                data.writeInt(1);
                book.writeToParcel(data, 0);
            } else {
                data.writeInt(0);
            }
            remote.transact(Stub.TRANSAVTION_addBook, data, replay, 0);
            replay.readException();
        } finally {
            replay.recycle();
            data.recycle();
        }
    }

    ...
}
```

我们看看 addBook() 的实现；在 Stub 类中，addBook(Book book) 是一个抽象方法，Server 端需要去实现它。

- 如果 Client 和 Server 在同一个进程，那么直接就是调用这个方法。
- 如果是远程调用，Client 想要调用 Server 的方法就需要通过 Binder 代理来完成，也就是上面的 Proxy。

在 Proxy 中的 addBook() 方法中首先通过 Parcel 将数据序列化，然后调用 remote.transact()。正如前文所述 Proxy 是在 Stub 的 asInterface 中创建，能走到创建 Proxy 这一步就说明 Proxy 构造函数的入参是 BinderProxy，即这里的 remote 是个 BinderProxy 对象。最终通过一系列的函数调用，Client 进程通过系统调用陷入内核态，Client 进程中执行 addBook() 的线程挂起等待返回；驱动完成一系列的操作之后唤醒 Server 进程，调用 Server 进程本地对象的 onTransact()。最终又走到了 Stub 中的 onTransact() 中，onTransact() 根据函数编号调用相关函数（在 Stub 类中为 BookManager 接口中的每个函数中定义了一个编号，只不过上面的源码中我们简化掉了；在跨进程调用的时候，不会传递函数而是传递编号来指明要调用哪个函数）；我们这个例子里面，调用了 Binder 本地对象的 addBook() 并将结果返回给驱动，驱动唤醒 Client 进程里刚刚挂起的线程并将结果返回。

这样一次跨进程调用就完成了。

### 参考文章

1. [写给 Android 应用工程师的 Binder 原理剖析](https://zhuanlan.zhihu.com/p/35519585)