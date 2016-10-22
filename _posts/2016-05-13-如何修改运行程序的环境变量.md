---
date: 2016-5-13 20:31:00+00:00
layout: post
title: 如何修改运行程序的环境变量
categories: linux编程
tags: 环境变量
---

linux修改环境变量的API有`putenv()`,`getenv()`,`setenv()`。如果在程序已经运行的情况下，尝试修改环境可以通过gdb来实现。

```
(gdb) attach process_id
(gdb) call putenv ("LD_PRELOAD=/mylib.so")
(gdb) detach
```
