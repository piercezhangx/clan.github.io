---
layout: default
title: 异步事件的IO复用处理机制
---

事件驱动模型采用IO复用（select, poll, epoll, ...）时往往也需要处理异步事
件（一般通过信号）；异步IO可以采用信号通知机制；有些事件处理模型采用pipe来
实现事件的通知。

Linux 2.6提供了几个新的系统调用可以让异步事件处理通过IO复用机制来处理，分别为：

 - eventfd (>=2.6.22)
 - signalfd (>=2.6.22)
 - timerfd_create (>=2.6.25)

其中eventfd可以取代pipe，节约一个fd；signalfd 和 timerfd_create 可以分别让信
号和定时器事件通过IO复用来处理。

{{ page.date | date_to_string }}
