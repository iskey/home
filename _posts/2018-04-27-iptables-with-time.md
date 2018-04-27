---
layout: post
title: "重定向外部连接到本地环回地址"
date: 2018-04-27 17:12:37 +0800
comments: true
categories: iptables
---

背景：在本地`localhost`搭建了一个Httpserver，监听在`4000`端口，现在想通过`eth0`在不改动代码的情况下对外发布服务。

最先想到的就是端口映射，`rinetd`服务，使用也很简单，直接配置外部ip到环回地址的映射就可以了:


``` java
root@lfgphicpra39095:/usr1/# cat /etc/rinetd.conf
......
#
# forwarding rules come here
#
# you may specify allow and deny rules after a specific forwarding rule
# to apply to only that forwarding rule
#
# bindadress    bindport  connectaddress  connectport

**10.252.64.154  4000 127.0.0.1 4000**

```

但是这样一来，一旦eth0的地址发生了改变，就需要再修改配置文件。


另一种方法就是通过iptables就行重定向：

```
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 4000 -j DNAT --to-destination 127.0.0.1
```

配置后，外部访问不通，通过抓包分析，协议栈没有回复`syc+ack`报文，看来是路由的问题。

内核协议栈会丢弃路由 对于源地址或目的地址为`loopback`地址的，内核协议栈的认为这是一个`martian packet`，直接丢弃。

```
route_localnet – BOOLEAN: Do not consider loopback addresses as martian source or destination while routing. This enables the use of 127/8 for local routing purposes (default FALSE).


```

这个特性是对每个网卡设备生效的，所以只需要在`eth0`上开启环回地址路由就可以了

```
echo 1 > /proc/sys/net/ipv4/conf/eth0/route_localnet
```
