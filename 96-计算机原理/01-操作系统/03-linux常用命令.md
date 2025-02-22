---
dg-publish: false
---

# grep
Q: `grep` 是一个在 Linux 和 Unix 系统中常用的命令行工具，用于搜索文本文件中的指定字符串，并将包含该字符串的行输出到屏幕上。

A:下面是一些 `grep` 命令的用法举例：

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

# whatis
在 Linux 系统中，`whatis` 数据库是一个包含命令和程序简短描述的数据库，主要用于提供手册页（man page）标题的快速查询。`whatis` 命令可以用来查找命令或程序的简要说明。

### 如何使用 `whatis` 命令

1. **基本用法**：
```
   whatis <command>
```
   例如：
```
   whatis ls
```
   输出：
```
   ls (1)               - list directory contents
```

2. **更新 `whatis` 数据库**：
   `whatis` 数据库通常需要手动更新，以确保它包含最新的手册页信息。可以使用 `makewhatis` 或 `mandb` 命令来更新数据库。

   - 使用 `makewhatis`：
```
     sudo makewhatis
```

   - 使用 `mandb`：
```
     sudo mandb
```

### `whatis` 数据库的作用
- **快速查询**：通过 `whatis` 命令，用户可以快速查询到命令或程序的简要描述，而不需要查看完整的手册页。
- **帮助学习**：对于新手来说，`whatis` 命令可以帮助他们快速了解系统中可用的命令及其用途。

### 示例
假设你想知道 `grep` 命令的用途，你可以运行：
```
whatis grep
```
输出可能是：
```
grep (1)               - print lines that match patterns
```

这表明 `grep` 命令用于打印匹配特定模式的行。

总之，`whatis` 数据库和命令是 Linux 系统中一个非常有用的工具，帮助用户快速获取命令和程序的简要信息。

# apropos 
- search the manual page names and descriptions
在 whatis 数据库中查找字符串
和 `man -k` 是等价的

```bash
[root@PrdCwgxApp01 ~]# apropos grep
bzgrep (1)           - search possibly bzip2 compressed files for a regular expression
egrep (1)            - print lines matching a pattern
fgrep (1)            - print lines matching a pattern
grep (1)             - print lines matching a pattern
msggrep (1)          - pattern matching on message catalog
pgrep (1)            - look up or signal processes based on name and other attributes
xzegrep (1)          - search compressed files for a regular expression
xzfgrep (1)          - search compressed files for a regular expression
xzgrep (1)           - search compressed files for a regular expression
zgrep (1)            - search possibly compressed files for a regular expression
zipgrep (1)          - search files in a ZIP archive for lines matching a pattern
```

# awk
`awk` 是一个强大的文本处理工具，广泛用于 Unix 和 Linux 系统中。它可以用于对文本文件进行扫描和处理，特别适合处理结构化数据（如列状数据）。`awk` 命令的名字来源于其三位作者的姓氏首字母：Alfred Aho、Peter Weinberger 和 Brian Kernighan。

### `awk` 的基本语法
```sh
awk 'pattern {action}' filename
```

- **pattern**：模式，用于匹配输入文件中的行。
- **action**：动作，对匹配的行执行的操作。
- **filename**：要处理的文件名。

### `awk` 的常用功能和示例

1. **打印文件的所有行**：
```sh
   awk '{print}' filename
```
   这条命令将打印文件中的所有行。

2. **打印文件的特定列**：
```sh
   awk '{print $1, $3}' filename
```
   这条命令将打印文件中的第一列和第三列。`$1` 表示第一列，`$3` 表示第三列。

3. **基于模式匹配的行**：
```sh
   awk '/pattern/ {print}' filename
```
   这条命令将打印包含“pattern”字符串的所有行。

4. **计算文件中的某列的总和**：
```sh
   awk '{sum += $1} END {print sum}' filename
```
   这条命令将计算第一列的总和，并在处理完所有行后打印总和。

5. **条件处理**：
```sh
   awk '$3 > 50 {print $1, $3}' filename
```
   这条命令将打印第三列值大于50的行的第一列和第三列。

6. **使用内置变量**：
   - `NR`：当前记录数（行号）。
   - `NF`：当前记录中的字段数。
```sh
   awk '{print NR, $0}' filename
```
   这条命令将打印行号和该行的内容。

7. **使用 BEGIN 和 END 块**：
```sh
   awk 'BEGIN {print "Start"} {print $0} END {print "End"}' filename
```
   这条命令将在处理文件之前打印 "Start"，然后打印文件的每一行，最后在处理完文件后打印 "End"。

### 一个更复杂的示例
假设有一个名为 `data.txt` 的文件，其内容如下：
```
John Doe  30
Jane Doe  25
Alice    28
Bob      22
```

你可以使用 `awk` 来完成各种任务：

- **打印所有名字和年龄**：
```sh
  awk '{print $1, $2, $3}' data.txt
```

- **只打印年龄大于25的人**：
```sh
  awk '$3 > 25 {print $1, $2, $3}' data.txt
```

- **计算平均年龄**：
```sh
  awk '{sum += $3; count++} END {print "Average age:", sum/count}' data.txt
```

总之，`awk` 是一个非常强大且灵活的文本处理工具，适用于各种文本处理任务。通过组合不同的模式和动作，你可以用 `awk` 实现复杂的数据处理和分析。

借助 **`-v`选项** ，可以将外部值（并非来自stdin）传递给awk：
```bash
[root@PrdCwgxApp01 ~]# VAR=100
[root@PrdCwgxApp01 ~]# echo | awk '{print $VAR}'

[root@PrdCwgxApp01 ~]# echo | awk -v var=$VAR '{print var}'
100
```

更多awk 参考： [awk详细用法](https://wangchujiang.com/linux-command/c/awk.html)

# base64

编码字符串

```bash
printf foo|base64
```

编码文件

```bash
base64 file
```

解码

```bash
printf Zm9v|base64 -d
```

解码文件

```bash
base64 -d file
```

# at(batch)
在指定的时间运行任务，，可以使用 `at` 命令。`at` 命令允许你指定具体的时间和日期来执行任务。

## 基本用法
 要在今天下午 3 点运行一个脚本 `myscript.sh`，可以这样做：

```sh
echo "/path/to/myscript.sh" | at 15:00
```

 交互式方式

你也可以直接输入 `at` 命令，进入交互式模式，然后输入你希望执行的命令。结束输入后，按 `Ctrl + D` 提交任务。

```sh
at 15:00
at> /path/to/myscript.sh
at> <Ctrl+D>退出交互模式
```

## 时间格式

`at` 命令支持多种时间格式，包括：

- **HH:MM**: 例如 `15:00` 表示今天的下午 3 点。
- **midnight**: 表示今天的午夜。
- **noon**: 表示今天的中午 12 点。
- **teatime**: 表示今天的下午 4 点。
- **now + time**: 例如 `now + 1 hour` 表示从现在起 1 小时后。
- **MMDDYY** 或 **MM/DD/YY**: 例如 `12/31/23` 表示 2023 年 12 月 31 日。
- **relative time**: 例如 `now + 2 days` 或 `tomorrow + 2 hours`。

假设你要在明天早上 9 点运行一个脚本 `backup.sh`，你可以使用以下命令：

```sh
echo "/path/to/backup.sh" | at 09:00 tomorrow
```

## 查看任务

要查看已提交但尚未执行的任务，可以使用 `atq` 命令：

```sh
atq
```

### 删除任务

```sh
atrm <job_number>
```

**batch用法和at基本一致 但是不能指定时间 而是在系统负载较低时自动运行**

# cd  
# 