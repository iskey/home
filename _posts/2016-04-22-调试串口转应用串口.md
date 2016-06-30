---
date: 2016-04-22 20:36:00+00:00
layout: post
title: 调试串口转应用串口
categories: 串口
tags: 串口
---

## 调试串口转应用串口


```
#include "stdio.h"
#include "sys/ioctl.h"
#include "fcntl.h"

#include "termios.h"
#include "unistd.h"

#define ECHOFLAGS (ECHO | ECHOE | ECHOK | ECHONL)

int main()
{
    int fd;
    int ret;
    char buf[100];
    struct termios term;
    
    fd= open("/dev/pts/0",O_RDONLY);
    if(fd<=0){
        printf("Error. open /dev/pts/0.\n");
        goto err;
    }
    ioctl(fd, TIOCCONS);
    close(fd);

    fd= open("/dev/ttyS0",O_RDONLY);
    if(fd<=0){
    fd= open("/dev/ttyS0",O_RDONLY);
    if(fd<=0){
        printf("Error. open /dev/console.\n");
        goto err;
    }

    ret= tcgetattr(fd, &term);
    if(-1== ret){
        printf("tcgetattr error.\n");
        goto err;
    }
    cfmakeraw(&term);
    term.c_lflag&= ~ECHOFLAGS;
    tcsetattr(fd, TCSAFLUSH, &term);
    tcsetattr(fd, TCSANOW, &term);
    if(-1== ret){
        printf("tcsetattr error.\n");
        goto err;
    }

    ret= read(fd, buf, 1);
    if(ret== -1){
        perror("read error.\n");    
        goto err;
    }
    
    while(*buf!='\n')
    {
        ret= read(fd, buf, 1);
        if(ret== -1){
            perror("read error1.\n");
            goto err;
        }
        printf("%02x\n",*buf);
    }
    ioctl(fd, TIOCCONS);
    close(fd);

    return 0;
err:
    return -1;
}
```
