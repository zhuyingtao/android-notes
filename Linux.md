

## 常用命令
- 查看开机时长 **top**,**last reboot**,**w**,**uptime**

### grep
- 用法

        grep [OPTIONS] PATTERN [FILE...]
        grep [OPTIONS] [-e PATTERN]...  [-f FILE]...  [FILE...]
- 参数

        -i  匹配时忽略字母的大小写
        -e  多模式匹配，匹配任意一个PATTERN
        -E  支持扩展正则表达式,例如'service|activity'
        -r  递归搜索，搜索当前目录和子目录
        -n  列出所有的匹配的文本行，并显示行号

### find
- 用法
        find [OPTION]... [查找路径] [查找条件] [处理动作]
