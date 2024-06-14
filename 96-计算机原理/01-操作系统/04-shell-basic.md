# 导航
pwd 
ls
cd

## ls
```bash
[root@VM-0-11-opencloudos bin]# ls -l /bin
lrwxrwxrwx 1 root root 7 Jan  8 04:59 /bin -> usr/bin
``` 
ls -l 一图解释 #flashcard 
![](assets/ls-l一图解释.png)
<!--ID: 1718096044579-->
# 文件
echo
参数用空格来分隔 如果我们的参数里本身就带空格怎么办呢 用转义字符
 echo hello\ world
 或者用引号
 echo "hello world"
 当我们创建文件 目录时，这个知识点比较有用
 mkdir my photos 
 因为上面的命令会创建两个目录
 