---
date: 2015-01-23 03:46:31+00:00
layout: post
title: 设计模式之单例模式
categories: 设计模式
tags: 单例模式  创建型模式
---

单例模式又称单件模式，是指整个程序的生命周期内，任一时刻，都只允许一个实例存在。

单例模式一般包含两层意思：
> * 全局访问
> * 全程唯一

一般，我们很容易把单例模式和全局变量混淆，觉得使用全局变量就是单例模式。
仔细分析，我们就会认识到，全局变量只实现了单例模式的**全局访问**特性，但是并不能保证**全局唯一**特性。

况且，全局变量在许多的代码规范中，都是不建议使用的。

在C语言中，单例模式一般设计如下：

```
struct Singleton{};

int get_instance()
{
    static Singleton *global_instance= NULL;
    
    if(NULL== global_instance){
        global_instance= malloc(sizeof(Singleton));
    }
    
    return global_instance;
}
```

但是放到多线程环境下，如上的实例就不能保证，在整个生命周期都是全局唯一了，多个线程有可能对同时进入初始化过程，所以需要加上多线程互斥访问机制，所以就可以写成如下：

```
pthread_mutex_t singleton_lock=PTHREAD_MUTEX_INITIALIZER;
struct Singleton{};

int get_instance()
{
    static struct Singleton *global_instance= NULL;
    
    pthread_mutex_lock(&singleton_lock);
    if(NULL== global_instance){
        global_instance= malloc(sizeof(Singleton));
        global_instance->lock= PTHREAD_MUTEX_INITIALIZER;
    }
    pthread_mutex_unlock(&singleton_lock);
    
    return global_instance;
}
```

现在不会出现多个线程同时初始化单件实例的情况了，但是随之而来的就是每次获取全局实例，都要执行一遍加锁、解锁的过程。
这时"饿汉写法"的弊端，如果单例的访问频次比较高，就需要另外新建一个执行初始化的函数，也就是所谓的“懒汉模式”。
