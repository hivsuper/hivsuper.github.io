---
title: 在Windows Docker中安装Redis   
date: 2021-06-13 00:18:00 +0800  
categories: [CNBlogs]   
tags: [redis, docker]  
---
<a href="https://www.cnblogs.com/hiver/p/14879401.html" target="_blank">点击查看原文</a>  
本文的前提是本地Docker环境已经配置成功，参考[Windows 10安装Docker并使用私钥连接AWS EC2](/posts/Windows-10安装Docker并使用私钥连接AWS-EC2/)
### 1. 参考资料
+ [redis DOCKER OFFICIAL IMAGE](https://hub.docker.com/_/redis)
+ [https://redis.io/topics/config](https://redis.io/topics/config)
+ [https://my.oschina.net/u/4313128/blog/4074047](https://my.oschina.net/u/4313128/blog/4074047)

### 2. 运行`docker pull redis:6.2.4`下载image
<img src="/assets/img/202106/571584-20210613000729666-1683886320.png" width="800" />

### 3. 下载[redis.conf](https://raw.githubusercontent.com/redis/redis/6.0/redis.conf)，并修改默认密码
```
# IMPORTANT NOTE: starting with Redis 6 "requirepass" is just a compatibility
# layer on top of the new ACL system. The option effect will be just setting
# the password for the default user. Clients will still authenticate using
# AUTH <password> as usually, or more explicitly with AUTH default <password>
# if they follow the new protocol: both will work.
#
requirepass 123456
```

### 4. 修改redis.conf配置使宿主机能够访问redis server
```
# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
# internet, binding to all the interfaces is dangerous and will expose the
# instance to everybody on the internet. So by default we uncomment the
# following bind directive, that will force Redis to listen only on the
# IPv4 loopback interface address (this means Redis will only be able to
# accept client connections from the same host that it is running on).
#
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT OUT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#bind 127.0.0.1
bind 0.0.0.0

# Protected mode is a layer of security protection, in order to avoid that
# Redis instances left open on the internet are accessed and exploited.
#
# When protected mode is on and if:
#
# 1) The server is not binding explicitly to a set of addresses using the
#    "bind" directive.
# 2) No password is configured.
#
# The server only accepts connections from clients connecting from the
# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
# sockets.
#
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured, nor a specific set of interfaces
# are explicitly listed using the "bind" directive.
#protected-mode yes
protected-mode no
```

### 5. 启动redis server
运行`docker run -v {CONFIG PATH}:/usr/local/etc/redis --name test-redis -p 6379:6379 redis:6.2.4 redis-server /usr/local/etc/redis/redis.conf`命令，终端将显示成功日志
```
1:C 12 Jun 2021 15:52:32.212 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 12 Jun 2021 15:52:32.212 # Redis version=6.2.4, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 12 Jun 2021 15:52:32.212 # Configuration loaded
1:M 12 Jun 2021 15:52:32.212 * monotonic clock: POSIX clock_gettime
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 6.2.4 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 1
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           https://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

1:M 12 Jun 2021 15:52:32.213 # Server initialized
1:M 12 Jun 2021 15:52:32.213 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1:M 12 Jun 2021 15:52:32.214 * Ready to accept connections
1:M 12 Jun 2021 15:58:48.100 * DB saved on disk
```
或者使用`docker run -v {CONFIG PATH}:/usr/local/etc/redis --name test-redis -p 6379:6379 -d redis:6.2.4 redis-server /usr/local/etc/redis/redis.conf`在后台启动redis server

### 6. 使用redis-cli连接redis server
执行`docker ps`找到正在运行的redis container  
<img src="/assets/img/202106/571584-20210613001507647-1827989846.png" width="800" />
运行`docker exec -it {CONTAINER ID} redis-cli -a {PASSWORD}`  
<img src="/assets/img/202106/571584-20210613001714129-2064783819.png" width="800" />
若不打开持久化模式，每次redis关闭时数据将丢失，加上`--appendonly yes`即可开启持久化，同时`-v {DATA PATH}:/data`可把数据挂载到指定的volume  
`docker run -v {CONFIG PATH}:/usr/local/etc/redis -v {DATA PATH}:/data --name test-redis -p 6379:6379 -d redis:6.2.4 redis-server /usr/local/etc/redis/redis.conf --appendonly yes`  
<img src="/assets/img/202106/571584-20210613114552128-766378054.png" width="800" />

### 7. 关闭redis server
执行`docker ps`找到正在运行的redis container，然后运行`docker kill {CONTAINER ID}`  
<img src="/assets/img/202106/571584-20210613010915385-94375020.png" width="800" />

### 8. 重启redis server
运行`docker ps -a`找到被关闭的redis container，然后运行`docker start {CONTAINER ID}`重新启动

### 9. 宿主机访问成功
![](/assets/img/202106/571584-20210613020455123-1997480408.png)