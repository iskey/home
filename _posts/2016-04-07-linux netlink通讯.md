---
date: 2016-04-07 13:12:00+00:00
layout: post
title: linux netlink通讯
categories: kernel
tags: kernel netlink
---

最近需要使用netlink来完成内核层和用户层的通讯，搜集了一个网上的例子，暂时摘抄在此，并计划再接下来再补充一些东西：

**ToDoList:**

> - [x] 重写netlink内核实现，把处理函数放在内核线程里。
- [ ] 使用net_namespace机制下的general netlink接口写一个列子。 

# 摘录版本
---

## 内核代码
```
#include <linux/init.h>
#include <linux/module.h>
#include <linux/timer.h>
#include <linux/time.h>
#include <linux/types.h>
#include <net/sock.h>
#include <net/netlink.h>

#define NETLINK_TEST 25
#define MAX_MSGSIZE 1024
int stringlength(char *s);
void sendnlmsg(int pid, char * message);
int err;
struct sock *nl_sk = NULL;

void sendnlmsg(int pid, char *message)
{
    struct sk_buff *skb_1;
    struct nlmsghdr *nlh;
    int len = NLMSG_SPACE(MAX_MSGSIZE);
    int slen = 0;
    if(!message || !nl_sk)
    {
        return ;
    }
    skb_1 = alloc_skb(len,GFP_KERNEL);
    if(!skb_1)
    {
        printk(KERN_ERR "my_net_link:alloc_skb_1 error\n");
    }
    slen = stringlength(message);
    nlh = nlmsg_put(skb_1,0,0,0,MAX_MSGSIZE,0);

    NETLINK_CB(skb_1).pid = 0;
    NETLINK_CB(skb_1).dst_group = 0;

    message[slen]= '\0';
    memcpy(NLMSG_DATA(nlh),message,slen+1);
    printk("my_net_link:send message '%s'.\n",(char *)NLMSG_DATA(nlh));

    netlink_unicast(nl_sk,skb_1,pid,MSG_DONTWAIT);

}

int stringlength(char *s)
{
    int slen = 0;

    for(; *s; s++){
        slen++;
    }

    return slen;
}

void nl_data_ready(struct sk_buff *__skb)
 {
    struct sk_buff *skb;
    struct nlmsghdr *nlh;
    char str[100];
    struct completion cmpl;
    int i=2;
    skb = skb_get(__skb);
    if(skb->len >= NLMSG_SPACE(0))
    {
        nlh = nlmsg_hdr(skb);

        memcpy(str, NLMSG_DATA(nlh), sizeof(str));
        printk("Message received from pid-%d:%s\n",nlh->nlmsg_pid, str) ;
        while(i--)
        {
            init_completion(&cmpl);
            wait_for_completion_timeout(&cmpl,3 * HZ);
            sendnlmsg(nlh->nlmsg_pid, "I am from kernel!");
        }
        kfree_skb(skb);
    }

 }

// Initialize netlink
int netlink_init(void)
{
    nl_sk = netlink_kernel_create(&init_net, NETLINK_TEST, 1,
                                 nl_data_ready, NULL, THIS_MODULE);

    if(!nl_sk){
        printk(KERN_ERR "my_net_link: create netlink socket error.\n");
        return 1;
    }

    printk("my_net_link_3: create netlink socket ok.\n");

    return 0;
}

static void netlink_exit(void)
{
    if(nl_sk != NULL){
        sock_release(nl_sk->sk_socket);
    }

    printk("my_net_link: self module exited\n");
}

module_init(netlink_init);
module_exit(netlink_exit);

MODULE_AUTHOR("frankzfz");
MODULE_LICENSE("GPL");
```

##　用户空间代码
```
#include <sys/stat.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <string.h>
#include <asm/types.h>
#include <linux/netlink.h>
#include <linux/socket.h>
#include <errno.h>

#define NETLINK_TEST 25
#define MAX_PAYLOAD 1024 // maximum payload size

int main(int argc, char* argv[])
{
    int state;
    struct sockaddr_nl src_addr, dest_addr;
    struct nlmsghdr *nlh = NULL;
    struct iovec iov;
    struct msghdr msg;
    int sock_fd, retval;
    int state_smg = 0;
    
    // Create a socket
    sock_fd = socket(AF_NETLINK, SOCK_RAW, NETLINK_TEST);
    if(sock_fd == -1){
        printf("error getting socket: %s", strerror(errno));
        return -1;
    }

    // To prepare binding
    memset(&msg,0,sizeof(msg));
    memset(&src_addr, 0, sizeof(src_addr));
    src_addr.nl_family = AF_NETLINK;
    src_addr.nl_pid = getpid(); // self pid
    src_addr.nl_groups = 0; // multi cast

    retval = bind(sock_fd, (struct sockaddr*)&src_addr, sizeof(src_addr));
    if(retval < 0){
        printf("bind failed: %s", strerror(errno));
        close(sock_fd);
        return -1;
    }

    // To prepare recvmsg
    nlh = (struct nlmsghdr *)malloc(NLMSG_SPACE(MAX_PAYLOAD));
    if(!nlh){
        printf("malloc nlmsghdr error!\n");
        close(sock_fd);
        return -1;
    }

    memset(&dest_addr,0,sizeof(dest_addr));
    dest_addr.nl_family = AF_NETLINK;
    dest_addr.nl_pid = 0;
    dest_addr.nl_groups = 0;

    nlh->nlmsg_len = NLMSG_SPACE(MAX_PAYLOAD);
    nlh->nlmsg_pid = getpid();
    nlh->nlmsg_flags = 0;
    strcpy(NLMSG_DATA(nlh),"Hello you iskey!");

    iov.iov_base = (void *)nlh;
    iov.iov_len = NLMSG_SPACE(MAX_PAYLOAD);
//    iov.iov_len = nlh->nlmsg_len;

    memset(&msg, 0, sizeof(msg));
    msg.msg_name = (void *)&dest_addr;
    msg.msg_namelen = sizeof(dest_addr);
    msg.msg_iov = &iov;
    msg.msg_iovlen = 1;

    state_smg = sendmsg(sock_fd,&msg,0);
    if(state_smg == -1)
    {
        printf("get error sendmsg = %s\n",strerror(errno));
    }
    printf("send netlink message ok.\n");

    memset(nlh,0,NLMSG_SPACE(MAX_PAYLOAD));   
    // Read message from kernel
    while(1){
        printf("In while recvmsg\n");
        state = recvmsg(sock_fd, &msg, 0);
        if(state<0)
        {
            printf("state<1");
        }
        printf("Received message: %s\n",(char *) NLMSG_DATA(nlh));
    }

    close(sock_fd);

    return 0;
}
```

# 内核线程版本
---
## 内核代码
```
#include <linux/init.h>
#include <linux/module.h>
#include <linux/timer.h>
#include <linux/time.h>
#include <linux/types.h>
#include <net/sock.h>
#include <net/netlink.h>
#include "linux/list.h"
#include <linux/kthread.h>

#define NETLINK_TEST 25
#define MAX_MSGSIZE 1024
int stringlength(char *s);
void sendnlmsg(int pid, char * message);
int err;
struct sock *nl_sk = NULL;

struct _my_user_client_{
    int pid;
    unsigned char *msg;
    struct list_head list; 
};

//struct list_head client_head;
LIST_HEAD(client_head);
spinlock_t g_cli_list_lock= SPIN_LOCK_UNLOCKED;
struct task_struct *g_thread[10];

int client_thread(void *data)
{
    struct list_head *plist, *list_n;
    struct _my_user_client_ *my_client;
    struct completion cmpl;
    
    init_completion(&cmpl);
    daemonize("iskey_cli");

    while(!kthread_should_stop())
    //while(1)
    {        
        spin_lock(&g_cli_list_lock);
        list_for_each_safe(plist, list_n, &client_head){
            my_client= (struct _my_user_client_ *)list_entry(plist, struct _my_user_client_, list);
            printk(KERN_ERR "kernel received msg:%s pid:%d.\n", my_client->msg, my_client->pid);
            sendnlmsg(my_client->pid, "I am from kernel!");
            list_del(plist);
            kfree(my_client);
        }
        spin_unlock(&g_cli_list_lock);
        schedule_timeout_interruptible(msecs_to_jiffies(1));
        wait_for_completion_timeout(&cmpl,3 * HZ);
    }
    printk(KERN_ERR "### thread end.\n");

    return 0;
}

void sendnlmsg(int pid, char *message)
{
    struct sk_buff *skb_1;
    struct nlmsghdr *nlh;
    int len = NLMSG_SPACE(MAX_MSGSIZE);
    int slen = 0;
    if(!message || !nl_sk)
    {
        return ;
    }
    skb_1 = alloc_skb(len,GFP_KERNEL);
    if(!skb_1)
    {
        printk(KERN_ERR "my_net_link:alloc_skb_1 error\n");
    }
    slen = stringlength(message);
    nlh = nlmsg_put(skb_1,0,0,0,MAX_MSGSIZE,0);

    NETLINK_CB(skb_1).pid = 0;
    NETLINK_CB(skb_1).dst_group = 0;

    message[slen]= '\0';
    memcpy(NLMSG_DATA(nlh),message,slen+1);
    printk("my_net_link:send message '%s' pid:%d.\n",(char *)NLMSG_DATA(nlh), pid);

    netlink_unicast(nl_sk,skb_1,pid,MSG_DONTWAIT);

}

int stringlength(char *s)
{
    int slen = 0;

    for(; *s; s++){
        slen++;
    }

    return slen;
}

void nl_data_ready(struct sk_buff *__skb)
 {
    struct sk_buff *skb;
    struct nlmsghdr *nlh;
    char str[100];
    struct completion cmpl;
    int i=2;
    skb = skb_get(__skb);
    if(skb->len >= NLMSG_SPACE(0))
    {
        nlh = nlmsg_hdr(skb);

        memcpy(str, NLMSG_DATA(nlh), sizeof(str));

        struct _my_user_client_ *client= kmalloc(sizeof(struct _my_user_client_), GFP_KERNEL);
        unsigned char *t_data= kmalloc(sizeof(str),GFP_KERNEL);
        memcpy(t_data, str, sizeof(str));
        client->msg= t_data;
        client->pid= nlh->nlmsg_pid;
        spin_lock(&g_cli_list_lock);
        list_add(&(client->list),&client_head);
        spin_unlock(&g_cli_list_lock);
        
        printk("Message received from pid-%d:%s\n",nlh->nlmsg_pid, str) ;
//        while(i--)
//        {
//            init_completion(&cmpl);
//            wait_for_completion_timeout(&cmpl,3 * HZ);
//            sendnlmsg(nlh->nlmsg_pid, "I am from kernel!");
//        }
        kfree_skb(skb);
    }

 }

// Initialize netlink
int netlink_init(void)
{
    nl_sk = netlink_kernel_create(&init_net, NETLINK_TEST, 1,
                                 nl_data_ready, NULL, THIS_MODULE);

    if(!nl_sk){
        printk(KERN_ERR "my_net_link: create netlink socket error.\n");
        return 1;
    }

    int cpu=0;
    
//    for_each_present_cpu(cpu){
        printk(KERN_ERR "cpu id is %d.\n", cpu);
        g_thread[cpu]= kthread_create(client_thread, &cpu, "netlink_thread%d", cpu);
        if(IS_ERR(g_thread[cpu])){
            printk(KERN_ERR "client_thread start error.\n");
            return 2;
        }

        printk(KERN_ERR "netlink_thread :%s\n", g_thread[0]->comm);
        
        wake_up_process(g_thread[cpu]);
//    }
    
    printk("my_net_link_3: create netlink socket ok.\n");

    return 0;
}

static void netlink_exit(void)
{
    if(nl_sk != NULL){
        sock_release(nl_sk->sk_socket);
    }

    int cpu= 0;
//    for_each_present_cpu(cpu){
        kthread_stop(g_thread[cpu]);
//    }
    
    printk("my_net_link: self module exited\n");
}

module_init(netlink_init);
module_exit(netlink_exit);

MODULE_AUTHOR("frankzfz");
MODULE_LICENSE("GPL");
```

##　用户空间代码
```
#include <sys/stat.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <string.h>
#include <asm/types.h>
#include <linux/netlink.h>
#include <linux/socket.h>
#include <errno.h>

#define NETLINK_TEST 25
#define MAX_PAYLOAD 1024 // maximum payload size


int main(int argc, char* argv[])
{
    int state;
    struct sockaddr_nl src_addr, dest_addr;
    struct nlmsghdr *nlh = NULL;
    struct iovec iov;
    struct msghdr msg;
    int sock_fd, retval;
    int state_smg = 0;
    
    // Create a socket
    sock_fd = socket(AF_NETLINK, SOCK_RAW, NETLINK_TEST);
    if(sock_fd == -1){
        printf("error getting socket: %s", strerror(errno));
        return -1;
    }

    // To prepare binding
    memset(&msg,0,sizeof(msg));
    memset(&src_addr, 0, sizeof(src_addr));
    src_addr.nl_family = AF_NETLINK;
    src_addr.nl_pid = getpid(); // self pid
    src_addr.nl_groups = 0; // multi cast

    retval = bind(sock_fd, (struct sockaddr*)&src_addr, sizeof(src_addr));
    if(retval < 0){
        printf("bind failed: %s", strerror(errno));
        close(sock_fd);
        return -1;
    }

    // To prepare recvmsg
    nlh = (struct nlmsghdr *)malloc(NLMSG_SPACE(MAX_PAYLOAD));
    if(!nlh){
        printf("malloc nlmsghdr error!\n");
        close(sock_fd);
        return -1;
    }

    memset(&dest_addr,0,sizeof(dest_addr));
    dest_addr.nl_family = AF_NETLINK;
    dest_addr.nl_pid = 0;
    dest_addr.nl_groups = 0;

    nlh->nlmsg_len = NLMSG_SPACE(MAX_PAYLOAD);
    nlh->nlmsg_pid = getpid();
    nlh->nlmsg_flags = 0;
    strcpy(NLMSG_DATA(nlh),"Hello you iskey!");

    iov.iov_base = (void *)nlh;
    iov.iov_len = NLMSG_SPACE(MAX_PAYLOAD);
//    iov.iov_len = nlh->nlmsg_len;

    memset(&msg, 0, sizeof(msg));
    msg.msg_name = (void *)&dest_addr;
    msg.msg_namelen = sizeof(dest_addr);
    msg.msg_iov = &iov;
    msg.msg_iovlen = 1;

    state_smg = sendmsg(sock_fd,&msg,0);
    if(state_smg == -1)
    {
        printf("[user] get error sendmsg = %s\n",strerror(errno));
    }
    printf("[user] send netlink message ok.\n");

    memset(nlh,0,NLMSG_SPACE(MAX_PAYLOAD));   
    // Read message from kernel
    //while(1){
        printf("[user] receiving netlink msg...\n");
        state = recvmsg(sock_fd, &msg, 0);
        if(state<0)
        {
            printf("[user] state<1");
        }
        printf("[user] Received message: %s\n",(char *) NLMSG_DATA(nlh));
    //}

    close(sock_fd);

    return 0;
}
```
