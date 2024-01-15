---
title: Docker Handbook  
date: 2024-01-13 22:04:00 +0800  
categories: [Uncategorized]  
tags: [docker]  
---
Refer to the earlier [article](/posts/Windows-10安装Docker并使用私钥连接AWS-EC2/) to create docker in `Windows OS`.  
### Create User and Add to the New User Group
`useradd -s /bin/bash -m {USER}` to create user
```
root@fcec697d2b29:/# useradd -s /bin/bash -m github
```
`sudo groupadd {GROUP}` to create user group
```
root@fcec697d2b29:/# sudo groupadd docker
```
`usermod -aG {GROUP} {USER}` to add an user to user group
```
root@fcec697d2b29:/# usermod -aG docker github
```
`su {USER}` to switch user
```
root@fcec697d2b29:/# su github
github@fcec697d2b29:/$ pwd
/
github@fcec697d2b29:/$
```

### Fix "Ports are not available" Error
If the port is unavailable when starting a docker container
```
PS C:\Users\Super Li> docker start test-mysql
Error response from daemon: Ports are not available: exposing port TCP 0.0.0.0:3306 -> 0.0.0.0:0: listen tcp 0.0.0.0:3306: bind: An attempt was made to access a socket in a way forbidden by its access permissions.
Error: failed to start containers: test-mysql
PS C:\Users\Super Li> netstat -aon | findstr :
```
Run `netstat -aon` to find if the port is occupied.
```
PS C:\Users\Super Li> netstat -aon | findstr 3306
  TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING       20940
  TCP    127.0.0.1:3306         127.0.0.1:14007        ESTABLISHED     20940
  TCP    127.0.0.1:14007        127.0.0.1:3306         ESTABLISHED     28732
  TCP    [::]:3306              [::]:0                 LISTENING       20940
  TCP    [::1]:3306             [::]:0                 LISTENING       23288
```
1. If the port is occupied by any process, kill it and retry.
1. If the port isn't occupied, restart `winnat` as administrator and retry.

```
PS C:\Windows\system32> net stop winnat

Windows NAT Driver 服务已成功停止。

PS C:\Windows\system32> net start winnat

Windows NAT Driver 服务已经启动成功。

PS C:\Windows\system32>
```