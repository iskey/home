---
date: 2016-04-12 10:10:00+00:00
layout: post
title: Makefile常用函数
categories: makefile
tags: makefile
---

[TOC]

# Makefile常用函数 #

---

## 字符串处理函数


### 字符串替换函数-subst

- cmd：`$(subst <from>,<to>,<text>)`
- 功能：把`<text>`中的`<from>`替换为`<to>`

### 模式字符串替换函数-patsubst

- cmd：`$(patsubst <pattern>,<replacement>,<text>)`
- 例子：`$(patsubst %.c,%.o,c.c a.c)`
- 说明：与`$(var:<pattern>=<replacement>)`相同

### 去空格格函数-strip

- cmd：`$(strip <string>)`
- 例子：`$(strip a b c)`

### 查找字符串函数-findstring

- cmd：`$(findstring <find>,<in>)`
- 功能：在字符串`<in>`中查找`<find>`
- 返回：如果找到则返回`<find>`,否则返回空字符串

### 过滤函数-filter

- cmd：`$(filter <pattern...>,<text>)`
- 功能：以`<pattern>`模式过滤`<text>`字符串中的单词，保留符合`<pattern>`的单词，可以有多个模式
- 返回：返回符合模式`<pattern>`的字符串
- 例子：
		sources:=foo.c bar.c baz.s ugh.h
		foo:$(sources)
		cc $(filter %.c %.s,$(sources)) -o foo

### 反过滤函数-filter-out

- cmd：`$(filter-out <pattern...>,<text>)`
- 功能：以`<pattern>`模式过滤`<text>`字符串的单词，去除符合模式`<pattern>`的单词，可以有多个模式
- 返回：返回不符合模式`<pattern>`的字符串
- 示例：
		objects= main1.o main2.o foo.o bar.o
		mains= main1.o main2.o

		$(filter-out $(mains),$(objects))

### 排序函数-sort

- cmd：`$(sort <list>)`
- 功能：给字符串`<list>`中的单词排序（升序）
- 返回：返回排序后的字符串
- 示例：`$(sort foo bar lose)`
- 备注：sort函数会去除`<list>`中相同的单词

### 取单词函数-word

- cmd：`$(word <n>,<text>)`
- 功能：取字符串`<text>`中的第`<n>`个单词
- 返回：返回第`<n>`个字符串，如果`<n>`比`<text>`的单词个数大，则返回空字符串

### 取单词串函数-wordlist

- cmd：`$(wordlist <s>,<e>,<text>)`
- 功能：从`<text>`中取从`<s>`到`<e>`的单词串，`<s>`和`<e>`是一个数字
- 返回：如果`<s>`大于`<text>`的长度，则返回空字符串
- 示例：`$(worllist 2,3,foo bar baz)`

### 单词个数统计函数-words

- cmd：`$(words <text>)`
- 功能：统计`<text>`中字符串中的单词个数

## 文件名操作函数

### 取目录函数-dir

- cmd：`$(dir <names...>)`
- 功能：从文件名序列`<names>`中取出目录部分

### 取文件函数-notdir

- cmd：`$(notdir <names...>)`
- 功能：从文件名序列`<names>`中取出文件名部分

### 取后缀函数-suffix

- cmd：`$(suffix <names>)`
- 功能：从文件名序列`<names...>`中取出各个文件名的后缀

### 取前缀函数-basename

- cmd：`$(basename <names>)`
- 功能：从文件名序列`<names...>`中取出各个文件名的前缀
- 返回：返回各个文件前缀，如果文件没有前缀，则返回空字符串

### 加后缀函数-addsuffix

- cmd：`$(addsuffix <suffix>,<names...>)`
- 功能：把后缀`<suffix>`加到`<names>`中的每个单词后面

### 加前缀函数-addprefrix

- cmd：`$(addprefix <prefix>,<names...>)`
- 功能：把前缀`<prefix>`加到`<names>`中的每个单词前面

### 连接函数-join

- cmd：`$(join <list1>,<list2>)`
- 功能：把`<list1>`中的单词对应的加到`<list2>`的单词后面

### 取目录文件函数-wildcard

- cmd：`$(wildcard <pattern>)`
- 功能：取目录下所有匹配`<pattern>`的文件，生成一个以空格间隔的文件名列表
- 示例：`$(wildcard *.c ./foo/*.c)`

## foreach函数

- cmd：`$(foreach <var>,<list>,<text>)`
- 功能：把参数`<list>`中的单词逐一取出放到参数`<var>`所指定的变量中，然后再执行`<text>`所包含的表达式
- 示例：
		names=a b c d
		files:=$(foreach iskey,$(names),$(iskey).o)

## if函数

- cmd：`$(if <condition>,<then-part>,<else-part>)`
- 功能：与GNU的make所支持的条件语句ifeq相似
- 返回：如果`<condition>`为真，则返回`<then-part>`部分，否则返回`<else-part>`部分

## call函数

- cmd：`$(call <expression>,<parm1>,<parm2>,<parm3>...) `
- 功能：当make执行这个函数时，`$(1),$(2),$(3)`等，会被参数`$(parm1),$(parm2),$(parm3)`替换
- 示例：
		reverse=$(1) $(2)
		foo= $(call reverse,a,b)
		
## origin函数

- cmd：`$(origin <variable>)`
- 功能：检查变量是在哪里定义的

## shell函数

- cmd：`$(Shell cat a.txt)`
- 功能：执行shell命令

## makefile自定义函数

	define make_target_dirs
		@mkdir -p　$(dir $2)
	endef

	all：
		$(call make_target_dirs,a,b,c)

## 使用shell命令

- shell脚本在target中才有效，其它地方都被忽略掉了
- make把每一行shell脚本当做一个独立的单元，他们在单独的进程中运行
- make在调用shell之前先进行预处理，会展开所有的Makefile变量和函数，这些变量和函数都以$开头
- make预处理时，所有以$开头的，它都不会放过，要想使用shell自己的变量，应该以$$开头，另外需要注意，shell自己的变量不需要括号
