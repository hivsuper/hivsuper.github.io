---
title: 在Windows Docker中安装MySQL  
date: 2021-06-13 03:17:00 +0800  
categories: [CNBlogs]   
tags: [mysql, docker]  
---
<a href="https://www.cnblogs.com/hiver/p/14879516.html" target="_blank">点击查看原文</a>  
本文的前提是本地Docker环境已经配置成功，参考[Windows 10安装Docker并使用私钥连接AWS EC2](https://hivsuper.github.io/posts/Windows-10安装Docker并使用私钥连接AWS-EC2/)
### 1. 参考资料
- [GRANT Statement](https://dev.mysql.com/doc/refman/8.0/en/grant.html)
- [How to fix "Host '172.18.0.1' is not allowed to connect" with MySQL Docker](https://www.jeffgeerling.com/blog/2017/how-fix-host-1721801-not-allowed-connect-mysql-docker)
- [mysql DOCKER OFFICIAL IMAGE](https://hub.docker.com/_/mysql?tab=description&page=5&ordering=last_updated)

### 2. 运行`docker pull mysql:5.7.34`下载image
```
5.7.34: Pulling from library/mysql
69692152171a: Pull complete
1651b0be3df3: Pull complete
0f86c95aa242: Pull complete
37ba2d8bd4fe: Pull complete
497efbd93a3e: Pull complete
a023ae82eef5: Pull complete
e76c35f20ee7: Pull complete
e887524d2ef9: Pull complete
ccb65627e1c3: Pull complete
Digest: sha256:a682e3c78fc5bd941e9db080b4796c75f69a28a8cad65677c23f7a9f18ba21fa
Status: Downloaded newer image for mysql:5.7.34
docker.io/library/mysql:5.7.34
d53b544bd61131e960f6983acfb3392a37283b5d469ce782c1f0210d71d7fadc
```

### 3. 启动MySQL
```
docker run -v {VOLUME PATH}:/var/lib/mysql --name test-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7.34
```
{VOLUME PATH}为数据持久化的位置，初始化时该文件夹必须为空
<img src="/assets/img/202106/571584-20210613115934679-1617487219.png" width="800" />

### 4. 宿主机连接
![](/assets/img/202106/571584-20210613031019599-2102506754.png)

### 5. 创建新用户和数据库
```
CREATE USER '{USERNAME}'@'%' IDENTIFIED BY '{PASSWORD}';
create database `{DATABASE}`
GRANT ALL ON {DATABASE}.* TO '{USERNAME}'@'%';
```
![](/assets/img/202106/571584-20210613031119978-1741252661.png)

### 6. 用新用户登录
`docker exec -it {CONTAINER ID} mysql -u {USERNAME} -p`
<img src="/assets/img/202106/571584-20210613031632535-1984418504.png" width="800" />