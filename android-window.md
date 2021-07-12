## Android Window 机制

#### 什么是 Window

window 机制就是为了管理屏幕上 view 的显示以及触摸事件的传递问题

- 管理 view 的显示次序，比如多个 view 重叠的显示，多个应用小窗的显示
- 管理触摸事件的传递，屏幕接收到触摸事件后，通过 window 机制进行下一步传递

在 Android 中，每个 view 树都可以看成一个 window。dialog、popupWindow 都是通过 WindowManager 添加到屏幕上的，所以是一个单独的 window。

window 本身是一个抽象的概念，view 树是 window 的存在形式，window 是 view 树的载体。

#### Window 的相关属性

##### type

window 的 type 属性，决定 window 的显示次序，其值为 Z-Order，代表 window 的高度

- 应用程序窗口：应用程序窗口一般位于最底层，Z-Order在1-99
- 子窗口：子窗口一般是显示在应用窗口之上，Z-Order在1000-1999
- 系统级窗口：系统级窗口一般位于最顶层，不会被其他的 window 遮住，如 Toast，Z-Order在2000-2999。**如果要弹出自定义系统级窗口需要动态申请权限**。

Z-Order越大，window越靠近用户，也就显示越高，高度高的window会覆盖高度低的window。