---
date: 2017-03-06 21:12:00+00:00
layout: post
title: user mode linux
categories: kernel
tags: kernel uml
---

以前研究内核，总是要依靠开发板或者自己编译内核．近来留意到了ＵＭＬ，才发现linux可以像进程一样跑起来，而且跑起来不不难，深感惭愧，以下是简单的使用方法介绍．

## 简介

用户模式Linux（User ModeLinux，UML）不同于其他Linux虚拟化项目，UML尽量将它自己作为一个普通的程序。从Linux2.6.9版本起，用户模式Linux（User mode Linux,UML）已随Linux内核源代码一起发布，它存放于arch/um目录下。编译好UML的内核之后，可直接用gdb运行编译好的内核并进行调试。

参考：http://uml.devloop.org.uk/howto.html

## 编译内核

### 以-O2编译内核

```
make ARCH=um defconfig
make menuconfig ARCH=um
make ARCH=um
```

参考：`http://user-mode-linux.sourceforge.net/source.html`

### 以-O0编译内核

参考：`http://blog.chinaunix.net/uid-234303-id-2425015.html`

 

## 根文件系统

### [下载现成的根文件系统](http://fs.devloop.org.uk/)
 
### [制作根文件系统](http://fancymore.com/reading/linux-kernel-debugging.html)

## 调试

```
# gdb ./linux   
(gdb) handle SIGSEGV pass nostop noprint   
(gdb) handle SIGUSR1 pass nostop noprint 
(gdb) start ubd0=/root/srcpkg/filesystem/CentOS6.x-x86-root_fs mem=256m
```

`http://blog.csdn.net/ztz0223/article/details/7874759`

五、常见问题

1、
http://moodlearchive.epfl.ch/2010-2011/mod/forum/discuss.php?d=108781

2、
http://www.lenky.info/archives/2012/04/1471