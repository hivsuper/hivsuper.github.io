---
title: Docker Handbook  
date: 2024-01-13 22:04:00 +0800  
categories: [Uncategorized]  
tags: [docker]  
---
Refer to the earlier [article](/posts/Windows-10安装Docker并使用私钥连接AWS-EC2/) to create docker in `Windows OS`.  
### How to Create User And Add to the New User Group?
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