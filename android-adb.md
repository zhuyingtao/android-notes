# ADB 命令

![](https://camo.githubusercontent.com/8049cd7a063e674ffb75114f3038945d5b7dc870/687474703a2f2f692e696d6775722e636f6d2f314f7839696c722e706e673f31)

## 1. 概述

adb 是android debug bridge 的缩写，从字面意思就可以看出，adb 是连接 PC 端与客户端之间的“桥梁”，通过它，我们可以从 PC 端对客户端进行调试。

## 2. 基本原理

adb 的运行原理是 PC 端的 adb server 与手机端的守护进程 adbd 建立连接，然后 PC 端的 adb client 通过 adb server 转发命令，adbd 接收命令后解析运行。

所以如果 adbd 以普通权限执行，有些需要 root 权限才能执行的命令无法直接用 adb xxx 执行。这时可以 adb shell 然后 su 后执行命令，也可以让 adbd 以 root 权限执行，这个就能随意执行高权限命令了。
## 3. 基本用法
### 3.1. 命令语法
adb 命令的基本语法如下：
```sh
adb [-d|-e|-s <serialNumber>] <command>
```
如果只有一个设备/模拟器连接时，可以省略掉 `[-d|-e|-s <serialNumber>]` 这一部分，直接使用 `adb <command>`。

在配置好环境变量的前提下，直接输入`adb`或者`adb help`可以列出所有的命令选项。

### 3.2. 常用命令

adb命令分为两种，分别是 **adb** 命令和 **adb shell** 命令。

- adb 命令是指 adb 这个程序自带的一些命令。
- adb shell 命令是指 android 设备自带的命令。这些命令存放在设备的 `/system/bin` 目录下，由一些 linux 通用的命令（如 cat/ls/ps）和 android 特有的命令（如 am/pm）组成。

#### adb 命令

---

- 查看 adb 版本

```sh
adb version
```

- 输出所有连接的设备

```sh
adb devices
```

- 启动 adb server (一般无需手动执行此命令，在运行 adb 命令时若发现 adb server 没有启动会自动调起)

```sh
adb start-server
```

| 参数        | 用途              |
| --------- | --------------- |
| -P <port> | 指定端口号，默认端口为5037 |

- 停止 adb server

```sh
adb kill-server
```

- 获取设备 root 权限

```sh
adb root
```

#### adb shell 命令

----

- 查看手机中应用的包名

```sh
adb shell pm list packages [-f] [-d] [-e] [-s] [-3] [-i] [-u] [--user USER_ID][FILTER]
```
| 参数         | 用途                |
| ---------- | :---------------- |
| 无          | 显示所有应用            |
| -f         | 显示应用关联的 apk 文件    |
| -d         | 只显示 disabled 应用   |
| -e         | 只显示 enabled 应用    |
| -s         | 只显示系统应用           |
| -3         | 只显示第三方应用          |
| -i         | 显示应用的 installer   |
| -u         | 包含已卸载应用           |
| `<FILTER>` | 包名包含`<FILTER>`字符串 |

- 安装应用

```sh
adb install [-lrtsdg] <path_to_apk>
```
| 参数   | 含义                                       |
| ---- | ---------------------------------------- |
| -l   | 将应用安装到保护目录 /mnt/asec                     |
| -r   | 允许覆盖安装                                   |
| -t   | 允许安装 AndroidManifest.xml 里 application 指定 `android:testOnly="true"` 的应用 |
| -s   | 将应用安装到 sdcard                            |
| -d   | 允许降级覆盖安装                                 |
| -g   | 授予所有运行时权限                                |

`adb install`内部原理简介：

1. `adb push` apk 到 /data/local/tmp目录下。
2. 调用`pm install`安装apk。
3. 删除/data/local/tmp目录下的 apk。

- 卸载应用

```sh
adb uninstall [-k] <package_name>
```

| 参数   | 含义             |
| ---- | -------------- |
| -k   | 卸载应用但保留数据和缓存目录 |

- 清除数据和缓存

```sh
adb shell pm clear <package_name>
```

##### dumpsys

可通过dumpsys命令查询系统服务的运行状态(对象的成员变量属性值)。对于分析调试问题，dumpsys 非常好用，可以避免每次修改都要加 log 重新编译。

dumpsys 支持查询的服务列表：

| 服务名       | 类名                   | 功能         |
| :----------- | :--------------------- | :----------- |
| activity     | ActivityManagerService | AMS相关信息  |
| package      | PackageManagerService  | PMS相关信息  |
| window       | WindowManagerService   | WMS相关信息  |
| input        | InputManagerService    | IMS相关信息  |
| power        | PowerManagerService    | PMS相关信息  |
| batterystats | BatterystatsService    | 电池统计信息 |
| battery      | BatteryService         | 电池信息     |
| alarm        | AlarmManagerService    | 闹钟信息     |
| dropbox      | DropboxManagerService  | 调试相关     |
| procstats    | ProcessStatsService    | 进程统计     |
| cpuinfo      | CpuBinder              | CPU          |
| meminfo      | MemBinder              | 内存         |
| gfxinfo      | GraphicsBinder         | 图像         |
| dbinfo       | DbBinder               | 数据库       |

直接 dumpsys 某个服务打出的信息量巨大，可以选择性的加一些参数进行过滤，可通过 `-h ` 命令来查看帮助信息

```bash
▶ adb shell dumpsys activity -h
Activity manager dump options:
  [-a] [-c] [-p PACKAGE] [-h] [WHAT] ...
  WHAT may be one of:
    a[ctivities]: activity stack state
    r[recents]: recent activities state
    b[roadcasts] [PACKAGE_NAME] [history [-s]]: broadcast state
    broadcast-stats [PACKAGE_NAME]: aggregated broadcast statistics
    i[ntents] [PACKAGE_NAME]: pending intent state
    p[rocesses] [PACKAGE_NAME]: process state
    o[om]: out of memory management
    perm[issions]: URI permission grant state
    prov[iders] [COMP_SPEC ...]: content provider state
    provider [COMP_SPEC]: provider client-side state
    s[ervices] [COMP_SPEC ...]: service state
    allowed-associations: current package association restrictions
    as[sociations]: tracked app associations
    lmk: stats on low memory killer
    lru: raw LRU process list
    binder-proxies: stats on binder objects and IPCs
    settings: currently applied config settings
    service [COMP_SPEC]: service client-side state
    package [PACKAGE_NAME]: all state related to given package
    all: dump all activities
    top: dump the top activity
  WHAT may also be a COMP_SPEC to dump activities.
  COMP_SPEC may be a component name (com.foo/.myApp),
    a partial substring in a component name, a
    hex object identifier.
  -a: include all available server state.
  -c: include client state.
  -p: limit output to given package.
  --checkin: output checkin format, resetting data.
  --C: output checkin format, not resetting data.
  --proto: output dump in protocol buffer format.
  --autofill: dump just the autofill-related state of an activity
```

使用示例：

```shell
adb shell dumpsys activity s <package> // 查看 app 的所有 service 状态
adb shell dumpsys activity b <package> // 查看 app 的所有 broadcast 状态
adb shell dumpsys activity top // 查看UI界面信息
```



- 查看内存情况

```shell
adb shell dumpsys meminfo
```

| Item | 全称                  | 含义     | 等价                         |
| :--- | :-------------------- | :------- | :--------------------------- |
| USS  | Unique Set Size       | 物理内存 | 进程独占的内存               |
| PSS  | Proportional Set Size | 物理内存 | PSS= USS+ 按比例包含共享库   |
| RSS  | Resident Set Size     | 物理内存 | RSS= USS+ 包含共享库         |
| VSS  | Virtual Set Size      | 虚拟内存 | VSS= RSS+ 未分配实际物理内存 |

故内存的大小关系：VSS >= RSS >= PSS >= USS

- 与应用交互，使用 `am <command>`命令，常用的<command>命令有：

| command                           | 用途                         |
| --------------------------------- | -------------------------- |
| `start [options] <INTENT>`        | 启动 `<INTENT>` 指定的 Activity |
| `startservice [options] <INTENT>` | 启动 `<INTENT>` 指定的 Service  |
| `broadcast [options] <INTENT>`    | 发送 `<INTENT>` 指定的广播        |
| `force-stop <packagename>`        | 停止 `<packagename>` 相关的进程   |

`<INTENT>`对应的选项如下：

| 参数               | 含义                                       |
| ---------------- | ---------------------------------------- |
| `-a <ACTION>`    | 指定 action，比如 `android.intent.action.VIEW` |
| `-c <CATEGORY>`  | 指定 category，比如 `android.intent.category.APP_CONTACTS` |
| `-n <COMPONENT>` | 指定完整 component ，比如 `com.example.app/.ExampleActivity` |

- 复制手机里的文件到 PC 端

```sh
adb pull <phone_path> [pc_path]
```

- 复制 PC 端的文件到手机

```shell
adb push <pc_path> <phone_path>
```

- 查看日志

```sh
adb logcat [<option>]...[<filter-spec>]...
```

##### 按级别过滤日志

Android 的日志分为如下几个优先级（priority）：

- V —— Verbose（最低，输出得最多）
- D —— Debug
- I —— Info
- W —— Warning
- E —— Error
- F —— Fatal
- S —— Silent（最高，啥也不输出）

按某级别过滤日志则会将该级别及以上的日志输出。

比如，命令：

```
adb logcat *:W
```

会将 Warning、Error、Fatal 和 Silent 日志输出。

##### 按 tag 和级别过滤日志

`<filter-spec>` 可以由多个 `<tag>[:priority]` 组成。

比如，命令：

```
adb logcat ActivityManager:I MyApp:D *:S
```

表示输出 tag `ActivityManager` 的 Info 以上级别日志，输出 tag `MyApp` 的 Debug 以上级别日志，及其它 tag 的 Silent 级别日志（即屏蔽其它 tag 日志）。

| 参数   | 含义        |
| ---- | --------- |
| -v   | 指定日志输出的格式 |
| -c   | 清空日志      |

- 查看设备信息

```sh
adb shell getprop [<property_key>]
```

```sh
adb shell getprop | grep <property>
```

- 获取设备屏幕信息

```shell
adb shell wm [size|density]
```

- 重启手机

```sh
adb reboot [recovery|bootloader]
```

- am

此命令是 **activity manager** 的缩写，用来执行各种系统操作，如启动 Activity、强行停止进程、广播 intent、修改设备屏幕属性等。命令格式如下：

```shell
adb shell am <command>
```

可用的 am 命令：

| 命令                              | 功能                      | 实现方法                   |
| :-------------------------------- | :------------------------ | :------------------------- |
| am start `[options`]  <INTENT>    | 启动Activity              | startActivityAsUser        |
| am startservice <INTENT>          | 启动Service               | startService               |
| am stopservice <INTENT>           | 停止Service               | stopService                |
| am broadcast <INTENT>             | 发送广播                  | broadcastIntent            |
| am kill <PACKAGE>                 | 杀指定后台进程            | killBackgroundProcesses    |
| am kill-all                       | 杀所有后台进程            | killAllBackgroundProcesses |
| am force-stop <PACKAGE>           | 强杀进程                  | forceStopPackage           |
| am hang                           | 系统卡住                  | hang                       |
| am restart                        | 重启                      | restart                    |
| am bug-report                     | 创建bugreport             | requestBugReport           |
| am dumpheap <pid> <file>          | 进程pid的堆信息输出到file | dumpheap                   |
| am send-trim-memory <pid> <level> | 收紧进程的内存            | setProcessMemoryTrimLevel  |
| am monitor                        | 监控                      | MyActivityController.run   |

am命令实的实现方式在Am.java，最终几乎都是调用`ActivityManagerService`相应的方法来完成。

- pm

此命令是 package manager 的缩写，可执行的操作有：

| 命令                            | 功能              | 实现方法                              |
| :------------------------------ | :---------------- | :------------------------------------ |
| list packages                   | 列举app包信息     | PMS.getInstalledPackages              |
| install `[options`] <PACKAGE>   | 安装应用          | PMS.installPackageAsUser              |
| uninstall `[options`] <PACKAGE> | 卸载应用          | IPackageInstaller.uninstall           |
| enable `<包名或组件名`>         | enable            | PMS.setEnabledSetting                 |
| disable `<包名或组件名`>        | disable           | PMS.setEnabledSetting                 |
| hide <PACKAGE>                  | 隐藏应用          | PMS.setApplicationHiddenSettingAsUser |
| unhide <PACKAGE>                | 显示应用          | PMS.setApplicationHiddenSettingAsUser |
| get-install-location            | 获取安装位置      | PMS.getInstallLocation                |
| set-install-location            | 设置安装位置      | PMS.setInstallLocation                |
| path <PACKAGE>                  | 查看App路径       | PMS.getPackageInfo                    |
| clear <PACKAGE>                 | 清空App数据       | AMS.clearApplicationUserData          |
| get-max-users                   | 最大用户数        | UserManager.getMaxSupportedUsers      |
| force-dex-opt <PACKAGE>         | dex优化           | PMS.forceDexOpt                       |
| dump <PACKAGE>                  | dump信息          | AM.dumpPackageStateStatic             |
| trim-caches `<目标size`>        | 紧缩cache目标大小 | PMS.freeStorageAndNotify              |

卸载系统应用（/system/app）

```shell
adb shell pm uninstall -k --user 0 package-name
```

- wm

  ```shell
  adb shell wm size //查看手机分辨率
  adb shell wm density //查看手机 dpi
  adb shell wm size reset //恢复默认分辨率
  adb shell wm 1080x1920 //设置手机分辨率为 1080*1920
  adb shell wm density 320 //设置手机 dpi 为 320
  ```

  

