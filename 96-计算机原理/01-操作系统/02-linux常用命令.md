---
dg-publish: false
---
# grep
`grep` 是一个在 Linux 和 Unix 系统中常用的命令行工具，用于搜索文本文件中的指定字符串，并将包含该字符串的行输出到屏幕上。

下面是一些 `grep` 命令的用法举例：

1.  搜索文件中的指定字符串并输出包含该字符串的行：
```bash
grep "hello" file.txt
```

上述命令将搜索名为 `file.txt` 的文件中的字符串 "hello"，并输出包含该字符串的行。

2.  搜索多个文件中的指定字符串并输出包含该字符串的行：
```bash
grep "hello" file1.txt file2.txt file3.txt
```

上述命令将搜索名为 `file1.txt`、`file2.txt` 和 `file3.txt` 的文件中的字符串 "hello"，并输出包含该字符串的行。

3.  搜索文件中的指定字符串并输出不包含该字符串的行：
```bash
grep -v "hello" file.txt
```

上述命令将搜索名为 `file.txt` 的文件中的字符串 "hello"，并输出不包含该字符串的行。

4.  搜索文件中的指定字符串并输出包含该字符串的行及其前后若干行：
```bash
grep -C 2 "hello" file.txt
```

上述命令将搜索名为 `file.txt` 的文件中的字符串 "hello"，并输出包含该字符串的行及其前后 2 行。

5.  搜索文件中的指定字符串并输出包含该字符串的行及其行号：
```bash
grep -n "hello" file.txt
```

上述命令将搜索名为 `file.txt` 的文件中的字符串 "hello"，并输出包含该字符串的行及其行号。

6.  搜索文件中的指定字符串并忽略大小写：
```bash
grep -i "hello" file.txt
```

上述命令将搜索名为 `file.txt` 的文件中的字符串 "hello"，并忽略大小写。

# ls 列出目录下文件

## 查看更多信息
```bash
ls -l
```

# more 查看文件信息
# 截取日志
```sh
sed -n "/2023-11-07 08:55:00/,/$(date "+%Y-%m-%d %H:%M:%S")/p" tomcat20231030_181534.log > log.txt
```
