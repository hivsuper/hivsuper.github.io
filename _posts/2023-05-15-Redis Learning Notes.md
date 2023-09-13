---
title: Redis Learning Notes  
date: 2023-05-15 11:28:00 +0800  
categories: [Technology, Redis Learning Journey]  
tags: [redis]  
---
This is Series II of [Redis学习记录](https://www.cnblogs.com/hiver/p/7403979.html).  

## References
+ [Redis为什么这么快？](https://juejin.cn/post/7065842156694405156)

## [4 Reasons Why Single-Threaded Redis is So Fast](https://levelup.gitconnected.com/4-reasons-why-single-threaded-redis-is-so-fast-414e0106f921)
Redis’ performance can be attributed to 4 primary factors
1. In-memory data storage  
Redis is an in-memory key-value store. Accessing the RAM is several order of magnitude faster than accessing the disk directly, thus, allowing Redis to be much faster than other data store.
1. Optimised data structure  
Without worrying persisting the data, data in Redis can be stored more efficiently for fast retrieval via different data structures.
1. Single-threaded architecture
- Minimises the CPU consumption due to threads creation or destruction
- Minimises the CPU consumption due to context switching
- Reduces lock overhead as multi-threaded applications require locks for thread synchronisation which are bug-prone
- Able to make use of various “thread-unsafe” commands such as Lpush
1. Non-blocking IO  
An I/O multiplexing module monitors multiple sockets simultaneously and only returns sockets that are readable. The ready-to-read sockets are pushed to a single-threaded event loop and are handled by the corresponding handlers using the Reactor Pattern.

## [Why does Redis 6 Introduce Multiple-Threaded?](https://www.cnblogs.com/javastack/p/15303036.html)
1. Only one CPU can be used in Single-Threaded mode
1. QPS can't be improved more
Therefore, Redis 6 introduces threaded I/O. Nevertheless, there are still limitations because the author prefers redis cluster than Multiple-Threaded
- Multiple-Threaded is only used for I/O whereas EventLoop is still Single-Threaded
- I/O thread can only READ or Write and event thread keeps waiting until all I/O threads get done