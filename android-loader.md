### Loader

#### 概述

Loader 是在 Android 3.0 中引入的，目的是使在 activity 或 fragment 中异步加载数据更加方便简单，其具有以下特征：

- 可用于每个 `Activity` 和 `Fragment`。
- 支持异步加载数据。
- 监控其数据源并在内容变化时传递新结果。
- 在某一配置更改后重建加载器时，会自动重新连接上一个加载器的游标。 因此，它们无需重新查询其数据。

#### 类和接口

在应用中使用 Loader 时，可能会涉及到多个类和接口。 下表汇总了这些类和接口：

| 类/接口                            | 说明                                       |
| ------------------------------- | ---------------------------------------- |
| `LoaderManager`                 | 一种与 `Activity` 或 `Fragment` 相关联的的抽象类，用于管理一个或多个 `Loader` 实例。 这有助于应用管理与 `Activity` 或 `Fragment` 生命周期相关联的、运行时间较长的操作。它最常见的用法是与 `CursorLoader`一起使用，但应用可自由写入其自己的加载器，用于加载其他类型的数据。 每个 Activity 或片段中只有一个 `LoaderManager`。但一个 `LoaderManager` 可以有多个加载器。 |
| `LoaderManager.LoaderCallbacks` | 一种回调接口，用于客户端与 `LoaderManager` 进行交互。例如，您可使用 `onCreateLoader()` 回调方法创建新的加载器。 |
| `Loader`                        | 一种执行异步数据加载的抽象类。这是加载器的基类。 您通常会使用 `CursorLoader`，但您也可以实现自己的子类。加载器处于活动状态时，应监控其数据源并在内容变化时传递新结果。 |
| `AsyncTaskLoader`               | 提供 `AsyncTask` 来执行工作的抽象加载器。              |
| `CursorLoader`                  | `AsyncTaskLoader` 的子类，它将查询 `ContentResolver` 并返回一个 `Cursor`。此类采用标准方式为查询游标实现 `Loader` 协议。它是以 `AsyncTaskLoader` 为基础而构建，在后台线程中执行游标查询，以免阻塞应用的 UI。使用此加载器是从 `ContentProvider` 异步加载数据的最佳方式，而不用通过片段或 Activity 的 API 来执行托管查询。 |

#### 使用方法

使用 Loader 的应用通常包括：

- `Activity` 或 `Fragment`。
- `LoaderManager` 的实例。
- 一个 `CursorLoader`，用于加载由 `ContentProvider` 支持的数据。您也可以实现自己的 `Loader` 或 `AsyncTaskLoader` 子类，从其他源中加载数据。
- 一个 `LoaderManager.LoaderCallbacks` 实现。您可以使用它来创建新加载器，并管理对现有加载器的引用。
- 一种显示加载器数据的方法，如 `SimpleCursorAdapter`。
- 使用 `CursorLoader` 时的数据源，如 `ContentProvider`。



1. **启动 Loader**

`LoaderManager` 可在 `Activity` 或 `Fragment` 内管理一个或多个 `Loader` 实例。每个 Activity 或片段中只有一个 `LoaderManager`。

通常，您会在 Activity 的 `onCreate()` 方法或片段的`onActivityCreated()` 方法内初始化 `Loader`。

```java
// Prepare the loader.  Either re-connect with an existing one,
// or start a new one.
getLoaderManager().initLoader(0, null, this);
```

`initLoader()` 方法采用以下参数：

- 用于标识加载器的唯一 ID。在此示例中，ID 为 0。
- 在构建时提供给加载器的可选参数（在此示例中为 `null`）。
- `LoaderManager.LoaderCallbacks` 实现， `LoaderManager` 将调用此实现来报告加载器事件。在此示例中，本地类实现 `LoaderManager.LoaderCallbacks` 接口，因此它会传递对自身的引用 `this`。

2. **使用 LoadManager 回调**

`LoaderManager.LoaderCallbacks` 是一个支持客户端与 `LoaderManager` 交互的回调接口。

`LoaderManager.LoaderCallbacks` 包括以下方法：

- `onCreateLoader()`：针对指定的 ID 进行实例化并返回新的 `Loader`


- `onLoadFinished()` ：将在先前创建的加载器完成加载时调用


- `onLoaderReset()`：将在先前创建的加载器重置且其数据因此不可用时调用

##### onCreateLoader

当您尝试访问加载器时（例如，通过 `initLoader()`），该方法将检查是否已存在由该 ID 指定的加载器。 如果没有，它将触发 `LoaderManager.LoaderCallbacks` 方法 `onCreateLoader()`。在此方法中，您可以创建新加载器。 通常，这将是 `CursorLoader`，但您也可以实现自己的 `Loader` 子类。

##### onLoadFinished

当先前创建的加载器完成加载时，将调用此方法。该方法必须在为此加载器提供的最后一个数据释放之前调用。 此时，您应移除所有使用的旧数据（因为它们很快会被释放），但不要自行释放这些数据，因为这些数据归其加载器所有，其加载器会处理它们。

##### onLoaderReset

此方法将在先前创建的加载器重置且其数据因此不可用时调用。 通过此回调，您可以了解何时将释放数据，因而能够及时移除其引用。



#### 参考资料

1. <https://developer.android.com/guide/components/loaders.html?hl=zh-cn#app>
2. <http://blog.csdn.net/yanbober/article/details/48861457>

