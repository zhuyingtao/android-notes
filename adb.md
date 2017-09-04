# ADB 命令

### 1. 概述

adb 是android debug bridge 的缩写，从字面意思就可以看出，adb 是连接 PC 端与客户端之间的“桥梁”，通过它，我们可以从 PC 端对客户端进行调试。

### 2. 基本原理

adb 的运行原理是 PC 端的 adb server 与手机端的守护进程 adbd 建立连接，然后 PC 端的 adb client 通过 adb server 转发命令，adbd 接收命令后解析运行。

所以如果 adbd 以普通权限执行，有些需要 root 权限才能执行的命令无法直接用 adb xxx 执行。这时可以 adb shell 然后 su 后执行命令，也可以让 adbd 以 root 权限执行，这个就能随意执行高权限命令了。
### 3. 基本用法
#### 3.1 命令语法
adb 命令的基本语法如下：
```sh
adb [-d|-e|-s <serialNumber>] <command>
```
如果只有一个设备/模拟器连接时，可以省略掉 [-d|-e|-s <serialNumber>] 这一部分，直接使用 `adb <command>`。

在配置好环境变量的前提下，直接输入`adb`或者`adb help`可以列出所有的命令选项。

#### 3.2 常用命令

adb命令分为两种，分别是 **adb** 命令和 **adb shell** 命令。

- adb 命令是指 adb 这个程序自带的一些命令。
- adb shell 命令是指 android 设备自带的命令。这些命令存放在设备的 `/system/bin` 目录下，由一些 linux 通用的命令（如 cat/ls/ps）和 android 特有的命令（如 am/pm）组成。

##### 3.2.1 adb 命令
```sh
- adb devices  // 输出设备列表
- adb start-server  // 启动 adb server (一般无需手动执行此命令，在运行 adb 命令时若发现 adb server 没有启动会自动调起)
    - -P <port> 指定端口号，默认端口为5037
- adb kill-server  // 停止 adb server
- adb version  // 查看 adb 版本
- adb root  // 获取手机 root 权限
```
##### ADB shell 命令

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

- 查看 activity

```sh
adb shell dumpsys activity activities
```

- 查看 service

```sh
adb shell dumpsys activity services [<package_name>]
```

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

- 重启手机

```sh
adb reboot [recovery|bootloader]
```
