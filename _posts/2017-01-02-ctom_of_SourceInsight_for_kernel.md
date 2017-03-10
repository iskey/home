---
date: 2017-01-02 15:32:00+00:00
layout: post
title: c.tom of SourceInsight for kernel
categories: SourceInsight 
tags: kernel SourceInsight
---
```

;For Linux Kernel Code
SYSCALL_DEFINE0(name) long sys_##name(void)
SYSCALL_DEFINE1(name, t1, v1) long sys_##name(t1 v1)
SYSCALL_DEFINE2(name, t1, v1, t2, v2) long sys_##name(t1 v1, t2 v2)
SYSCALL_DEFINE3(name, t1, v1, t2, v2, t3, v3) long sys_##name(t1 v1, t2 v2, t3 v3)
SYSCALL_DEFINE4(name, t1, v1, t2, v2, t3, v3, t4, v4) long sys_##name(t1 v1, t2 v2, t3 v3, t4 v4)
SYSCALL_DEFINE5(name, t1, v1, t2, v2, t3, v3, t4, v4, t5, v5) long sys_##name(t1 v1, t2 v2, t3 v3, t4 v4, t5 v5)
__init
__initdata
__initconst
__exitdata
__exit_call
__ref
__refdata
__refconst
__init_refok
__initdata_refok
__exit_refok
__exitused
__exit
__devinit
__devinitdata
__devinitconst
__devexit
__devexitdata
__devexitconst
__used
__must_check
__compiler_offsetof(a,b)
__always_inline
__attribute__
__user
__kernel
__safe
__force
__nocast
__iomem
__chk_user_ptr(x)
__chk_io_ptr(x)
__builtin_warning
__acquires(x)
__releases(x)
__acquire(x)
__release(x)
__cond_lock(x,c)
__percpu
likely(x)
unlikely(x)
__read_mostly
____cacheline_aligned
____cacheline_aligned_in_smp
__cacheline_aligned
__cacheline_aligned_in_smp
____cacheline_internodealigned_in_smp
__cacheline_internodealigned_in_smp
__rcu
__refconst
__refdata
__reg_oper0
__reg_oper1
__bitwise
__bitwise__
__aligned_u64
__aligned_be64
__aligned_le64
__naked
__noreturn
__pure
__always_inline
DRIVER_ATTR(_name, _mode, _show, _store) struct driver_attribute driver_attr_##_name
CLASS_ATTR(_name, _mode, _show, _store) struct class_attribute class_attr_##_name
LIST_HEAD(name) struct list_head name
DEFINE_MUTEX(mutexname) struct mutex mutexname
DEFINE_TIMER(_name, _function, _expires, _data) struct timer_list _name
DECLARE_WORK(n, f) struct work_struct n
DEFINE_SPINLOCK(x) spinlock_t x
EXPORT_SYMBOL(sym)
EXPORT_SYMBOL_GPL(sym)
EXPORT_SYMBOL_GPL_FUTURE(sym)
DEFINE_IDA(name) struct ida name
DECLARE_RWSEM(name) struct rw_semaphore name
DECLARE_BITMAP(name,bits) unsigned long name[bits/32]
DEFINE_RWLOCK(x) rwlock_t x
 
```.