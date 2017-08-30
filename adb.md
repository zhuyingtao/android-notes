# ADB 命令

### 基本原理

adb 的运行原理是 PC 端的 adb server 与手机端的守护进程 adbd 建立连接，然后 PC 端的 adb client 通过 adb server 转发命令，adbd 接收命令后解析运行。

所以如果 adbd 以普通权限执行，有些需要 root 权限才能执行的命令无法直接用 adb xxx 执行。这时可以 adb shell 然后 su 后执行命令，也可以让 adbd 以 root 权限执行，这个就能随意执行高权限命令了。
### 基本用法
#### 命令语法
ADB 命令的基本语法如下：
```sh
adb [-d|-e|-s <serialNumber>] <command>
```
如果只有一个设备/模拟器连接时，可以省略掉 [-d|-e|-s <serialNumber>] 这一部分，直接使用 adb <command>。

#### 常用命令
##### ADB 命令
```sh
- adb devices  输出设备列表
- adb start-server  启动 adb server (一般无需手动执行此命令，在运行 adb 命令时若发现 adb server 没有启动会自动调起)
    - -P <port> 指定端口号，默认端口为5037
- adb kill-server  停止 adb server
- adb version  查看 adb 版本
- adb root  获取手机 root 权限
```
##### ADB shell 命令
```sh
- adb shell pm list packages [-f] [-d] [-e] [-s] [-3] [-i] [-u] [--user USER_ID][FILTER]
```
| 参数       | 用途              |
| -------- | :-------------- |
| 无        | 显示所有应用          |
| -f       | 显示应用关联的 apk 文件  |
| -d       | 只显示 disabled 应用 |
| -e       | 只显示 enabled 应用  |
| -s       | 只显示系统应用         |
| -3       | 只显示第三方应用        |
| -i       | 显示应用的 installer |
| -u       | 包含已卸载应用         |
| <FILTER> | 包名包含<FILTER>字符串 |
