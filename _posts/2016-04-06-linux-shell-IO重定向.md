---
date: 2016-04-06 15:48:00+00:00
layout: post
title: linux shell I/O重定向
categories: 脚本
tags: shell
---

在命令行中，经常会用到shell命令的重定向操作，最常用的做法是重定向到一个文件：

```
$ls > t.txt
```
把当前目录的文件列表重定向到t.txt文件，这里的**>**符号的意思就是把```ls```命令的标准输出重定向到t.txt文件。
熟悉linux的同学都知道，linux 有三个标准句柄，

```
0: stdin
1: stdout
2: stderr
```
所以如果我们想要截获命令行的所有输出标准输出信息的话，就应该采用如下写法：

```
$ls >t.txt 2>&1
```
有次，移植python的时候，遇到些问题，需要看看configure的流程，看到了另外一种写法：

```
exec 5>> config.log
exec 6>> Setup.config
echo "hello"> &5
echo "test config"> &6
```
这样就可以实现向不同的文件写入详细的过程，而不会污染configure执行过程的输出信息了。
更巧妙的是，如果你是再改造一个已经混乱不堪，输出非常乱的脚本，你可以把标准输出重定向到文件，而自己用一个重定向的句柄来接管标准输出，如下：

```
echo "first I am in stdout"
exec 3>> t.log
exec 4>&1
exec 5>&2
exec 1>&3
exec 2>&3
echo "seconf I am not in stdout"
echo "third I am in stdout">&4
echo "fourth I am in stderr">&5
```

题外话：
> 在调试复杂脚本的时候，很难跟踪流程，可以使用trap命令来很方便的跟踪流程

```
trap 'echo “before execute line:$LINENO, a=$a,b=$b,c=$c”' DEBUG
```

>  trap捕捉到信号之后，可以有三种反应方式：
    >> 1. 执行一段程序来处理这一信号
2. 接受信号的默认操作
3. 忽视这一信号

