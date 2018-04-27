---
layout: post
title: "iptables时间规则匹配"
date: 2018-04-27 19:25:48 +0800
comments: true
categories: iptables
---

1、每天固定时间段匹配

``` java

iptables -I FORWARD -s 172.17.1.132 -d 192.168.1.119 -m time --timestart 09:40 --timestop 09:59 -j DROP

```

2、按周固定时间段匹配

```
iptables -I FORWARD -s 172.17.1.132 -d 192.168.1.119 -m time --timestart 09:40 --timestop 09:59 --weekdays Wed,Thu -j DROP

```
3、按固定日期匹配，注这里比较特殊，可以看见下面的时间是17点不是9点，是因为时区的原因，要差8小时。

```
iptables -I FORWARD -s 172.17.1.132 -d 192.168.1.119 -m time --datestart 2014-3-19T17:40:08 --datestop 2014-3-19T17:59:50 -j DROP

```

附上帮助说明：

```
time match options:
    --datestart time     Start and stop time, to be given in ISO 8601
    --datestop time      (YYYY[-MM[-DD[Thh[:mm[:ss]]]]])
    --timestart time     Start and stop daytime (hh:mm[:ss])
    --timestop time      (between 00:00:00 and 23:59:59)
[!] --monthdays value    List of days on which to match, separated by comma
                         (Possible days: 1 to 31; defaults to all)
[!] --weekdays value     List of weekdays on which to match, sep. by comma
                         (Possible days: Mon,Tue,Wed,Thu,Fri,Sat,Sun or 1 to 7
                         Defaults to all weekdays.)
    --localtz/--utc      Time is interpreted as UTC/local time
```
