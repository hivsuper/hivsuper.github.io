---
title: Redis Learning Notes  
date: 2023-05-15 11:28:00 +0800  
categories: [Technology]  
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


## [3 Cache Problems and Mitigation Approaches](https://www.pixelstech.net/article/1586522853-What-is-cache-penetration-cache-breakdown-and-cache-avalanche)

### Cache penetration(缓存穿透)
Cache penetration is a scenario where the data to be searched doesn't exist at DB and the returned empty result is not cached as well and hence every search for the key will hit the DB eventually.
The mitigation plans:
1. If there is no data for the key in DB, just return an empty result and cache it for a short period of time.
1. Using Bloom filter or add a verification layer similar. Bloom filter is similar to hbase set which can be used to check whether a key exists in the data set. If the key exists, go to the cache layer or DB layer, if it doesn't exists in the data set, then just return.
If the searched key has high repeat rate, then can adopt the first solution. Otherwise if the searched key has low repeat rate and the searched keys are too many, can adopt the second solution to filter most of them first.

### Cache breakdown(缓存击穿)
Cache breakdown is a scenario where the cached data expires and at the same time there are lots of search on the expired data which suddenly cause the searches to hit DB directly and increase the load to the DB layer dramatically.
The mitigation plans:
1. Place a mutex lock in concurrency environment. When some threads are searching the key and updating the cache, any other request should wait until the lock gets released.
1. Asynchronously update the cached data through a worker thread so that the hot data will never expire.

### Cache avalanche(缓存雪崩)
Cache avalanche is a scenario where lots of cached data expire at the same time or the cache service is down and all of a sudden all searches of these data will hit DB and cause high load to the DB layer and impact the performance.
The mitigation plans:
1. If Redis is used, use redis clusters to ensure that some cache server instance is in service at any point of time.
1. Adjust the expiration time for different keys so that they will not expire at the same time.
1. Never expire hot data.