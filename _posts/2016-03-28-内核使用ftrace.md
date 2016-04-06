---
date: 2016-03-28 23:44:00+00:00
layout: post
title: 内核使用ftrace
categories: kernel
tags: ftrace kernel
---

```
#!/bin/sh
ROOT_DEBUGFS="/debug"
ROOT_FTRACE="${ROOT_DEBUGFS}/tracing"
#mount -t debugfs nodev ${ROOT_DEBUGFS}
echo 0 > ${ROOT_FTRACE}/trace
echo $$ > ${ROOT_FTRACE}/set_ftrace_pid
echo 1 > ${ROOT_FTRACE}/tracing_enabled
echo 1 > ${ROOT_FTRACE}/tracing_on
# can set other filtering here
echo function > ${ROOT_FTRACE}/current_tracer
echo start_trace_marker > ${ROOT_FTRACE}/trace_marker
exec $*
echo end_trace_marker > ${ROOT_FTRACE}/trace_marker
echo 0 > ${ROOT_FTRACE}/tracing_on
echo 0 > ${ROOT_FTRACE}/tracing_enabled
```
