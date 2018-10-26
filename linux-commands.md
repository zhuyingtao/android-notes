
## [常用命令](http://www.runoob.com/linux/linux-command-manual.html)
- 文件管理
    - 文件操作 [mv](http://www.runoob.com/linux/linux-comm-mv.html) | [rm](http://www.runoob.com/linux/linux-comm-rm.html) | [cp](http://www.runoob.com/linux/linux-comm-cp.html) | [touch](http://www.runoob.com/linux/linux-comm-touch.html)
    - 文件查找 [find](http://www.runoob.com/linux/linux-comm-find.html) | [locate](http://www.runoob.com/linux/linux-comm-locate.html) | [which](http://www.runoob.com/linux/linux-comm-which.html) | [whereis](http://www.runoob.com/linux/linux-comm-whereis.html) | type ---[区别](http://www.ruanyifeng.com/blog/2009/10/5_ways_to_search_for_files_using_the_terminal.html)
    - 文件内容查看 [cat](http://www.runoob.com/linux/linux-comm-cat.html) | [more](http://www.runoob.com/linux/linux-comm-more.html) | [less](http://www.runoob.com/linux/linux-comm-less.html) | head | tail --- [区别](https://my.oschina.net/junn/blog/304868)
    - 文件属性更改 [chmod](http://www.runoob.com/linux/linux-comm-chmod.html) | [chown](http://www.runoob.com/linux/linux-comm-chown.html) | [chgrp](http://www.runoob.com/linux/linux-comm-chgrp.html) | [chattr](http://www.runoob.com/linux/linux-comm-chattr.html)
    - 文件类型 [file](http://www.runoob.com/linux/linux-comm-file.html)
    - 窗口打开文件夹 [nautilus](http://www.jianshu.com/p/3d1e527419cd)
- 内容匹配
  - [awk](http://www.runoob.com/linux/linux-comm-awk.html)
  - [grep](http://www.runoob.com/linux/linux-comm-grep.html)
- 链接 [ln](http://www.runoob.com/linux/linux-comm-ln.html)
- 备份压缩 [tar](http://www.runoob.com/linux/linux-comm-tar.html)
- 系统管理
  - ps
  - [top](http://www.runoob.com/linux/linux-comm-top.html)
  - 显示用户登录信息 [w](http://www.runoob.com/linux/linux-comm-w.html)
  - 显示内存状况 [free](http://www.runoob.com/linux/linux-comm-free.html)
  - 环境变量 printenv | set
- 软件工具
    - [apt-get](http://man.linuxde.net/apt-get)
    - [dpkg](http://man.linuxde.net/dpkg)

## 技巧
- [命令链接操作符](https://linux.cn/article-2469-1.html)

## [Shell脚本编程](http://www.runoob.com/linux/linux-shell.html)

## 快捷键
| 描述        | 快捷键          |
| --------- | ------------ |
| 打开终端      | Ctrl+Alt+T   |
| 复制        | Ctrl+Shift+C |
| 粘贴        | Ctrl+Shift+V |
| 新建标签页     | Ctrl+Shift+T |
| 关闭标签页     | Ctrl+Shift+W |
| 跳转到行首     | Ctrl+A       |
| 跳转到行尾     | Ctrl+E       |
| 向前跳一个字符   | Ctrl+F       |
| 向后跳一个字符   | Ctrl+B       |
| 向前跳一个单词   | Alt+F        |
| 向后跳一个单词   | Alt+B        |
| 删除光标前所有字符 | Ctrl+U       |
| 删除光标后所有字符 | Ctrl+K       |



#### 重定向

重定向输出结果：`>` 重写，`>>` 追加

重定向输入结果：`<` 重写

文件描述符：0 - 标准输入，1 - 标准输出，2 - 错误输出

```bash
ls -l /bin/usr > ls-output.txt  // 重定向标准输出
ls -l /usr/bin >> ls-output.txt // 追加重定向标准输出
ls -l /bin/usr 2> ls-error.txt	// 重定向标准错误
ls -l /bin/usr > ls-output.txt 2>&1 // 重定向标准输出和错误到同一个文件
ls -l /bin/usr &> ls-output.txt // 同上
ls -l /bin/usr 2> /dev/null // 处理不需要的输出
cat < lazy_dog.txt
```

管道线： `command1 | command2` ，一个命令的标准输出可以通过管道送至另一个命令的标准输入

```sh
 ls -l /usr/bin | sort | less
```

- 花括号展开

```shell
mkdir {2017..2019}-0{1..9} {2017..2019}-{10..12}
```

#### 启动文件

- 非登录shell

`/etc/bash.bashrc` -- > `~/.bashrc`

- 登录shell

`/etc/profile` --> `~/.bash_profile`或`~/.bash_login`或`~/.profile`

`~/.bashrc`是最重要的启动文件，因为它几乎总是被读取，无论是非登录shell还是登录shell