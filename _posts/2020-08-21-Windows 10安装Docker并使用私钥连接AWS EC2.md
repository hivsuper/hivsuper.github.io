---
title: Windows 10安装Docker并使用私钥连接AWS EC2  
date: 2020-08-21 22:41:00 +0800  
categories: [CNBlogs]  
tags: [docker]  
---
<a href="https://www.cnblogs.com/hiver/p/13543739.html" target="_blank">点击查看原文</a> 
------------------------------------------------------------------------

2025-08-17 添加Docker image的导出导入方法

------------------------------------------------------------------------
### 1. 参考资料
+ [ubuntu docker 开启ssh](https://blog.csdn.net/qq_27068845/article/details/77015432)
+ [docker - 容器里安装ssh](https://www.cnblogs.com/sunshine-2015/p/6384471.html)
+ [保存对容器的修改](https://www.docker.org.cn/book/docker/docer-save-changes-10.html)
+ [How to copy files from host to Docker container?](https://stackoverflow.com/questions/22907231/how-to-copy-files-from-host-to-docker-container)

### 2. 下载Docker安装程序，确认Hyper-V已经开启
[docker for windows](https://docs.docker.com/docker-for-windows/install/)

### 3. 在PowerShell运行`docker version`确认是否安装成功
![](/assets/img/202008/571584-20200821215636042-54529853.png)
若不成功则重启操作系统后重试

### 4. 安装Ubuntu
在PowerShell运行`docker run -it ubuntu`命令，输出以下信息表示安装成功
```
 Unable to find image 'ubuntu:latest' locally
 latest: Pulling from library/ubuntu
 54ee1f796a1e: Pull complete
 f7bfea53ad12: Pull complete
 46d371e02073: Pull complete
 b66c17bbf772: Pull complete
 Digest: sha256:31dfb10d52ce76c5ca0aa19d10b3e6424b830729e32a89a7c6eee2cda2be67a5
 Status: Downloaded newer image for ubuntu:latest
```

### 5. <a href="javascript:;" name="5" style="text-decoration:none;color:inherit">在PowerShell运行`docker images`查看image</a>
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              4e2eef94cd6b        16 hours ago        73.9MB
```

### 6. 在PowerShell启动ubuntu
* 运行`docker run -it --privileged=true -p 10022:22 ubuntu`命令进入ubuntu系统(注：可使用--name自定义名称)
* 运行`lsb_release -a`确认系统(注：若lsb_release命令不存在则可使用`cat /etc/issue`)
![](/assets/img/202008/571584-20200821220711165-954935248.png)

### 7. 安装工具
依次运行以下命令
> apt-get update  
 apt-get install vim  
 apt-get install openssh-server  

### 8. 创建私钥，添加内容后修改权限
![](/assets/img/202008/571584-20200821221938853-816927086.png)
运行`ssh -i ~/aws/key {ec2.user}@{ec2.id.address}`连接EC2

### 9. Docker image

#### 9.1 保存
* 在PowerShell运行`docker ps -l`查看Container
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
2e48d1d6cd44        2cdf64bb985d        "/bin/bash"         21 minutes ago      Up 21 minutes       0.0.0.0:10022->22/tcp   ecstatic_nash
```
* 在PowerShell运行`docker commit {CONTAINER ID} {image.name}`(例如`docker commit 2e48d1d6cd44 test/ubuntu`)
* 在PowerShell运行`docker images`查看image
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
test/ubuntu         latest              38416deebfd4        52 seconds ago      263MB
ubuntu              latest              4e2eef94cd6b        16 hours ago        73.9MB
```

#### 9.2 导出
* 在PowerShell运行`docker save -o {file.name} {REPOSITORY}:{TAG}`导出image

#### 9.3 删除
* [在PowerShell运行`docker images`查看image](#5)
* 在PowerShell运行`docker image rm {IMAGE ID}`或`docker image rm {REPOSITORY}`删除对应的docker image

#### 9.4 导入
* 在PowerShell运行`docker load -i {file.name}`导入image

### 10. 关闭docker进程
* 关闭特定进程：在PowerShell运行`docker kill {CONTAINER ID}`
* 关闭所有：在PowerShell运行`docker stop $(docker ps -a -q)`命令

### 11. Docker container
#### 11.1 重启已经关闭的container并打开bash
* `docker container ls -a`找到container信息
* `docker start {CONTAINER ID}`或者`docker start {NAMES}`启动
* `docker exec -it {CONTAINER ID} bash`或者`docker exec -it {NAMES} bash`打开容器终端

#### 11.2 删除container
* `docker container ls -a`找到container信息
* `docker rm {CONTAINER ID}`或`docker rm {NAMES}`删除对应的container

### 12. 容器与主机文件互传
* 启动容器(例如`docker run -it --privileged=true -p 10022:22 test/ubuntu`)
* 查看正在运行的container id `docker ps`
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
7bc4886a6824        test/ubuntu         "/bin/bash"         13 seconds ago      Up 12 seconds       0.0.0.0:10022->22/tcp   mystifying_panini
```
* 主机->容器 `docker cp {source.file.path} {CONTAINER ID}:{target.file.path}`(例如`docker cp .\key 7bc4886a6824:/tmp`)
* 容器->主机 `docker cp {CONTAINER ID}:{source.file.path} {target.file.path}`(例如`docker cp 7bc4886a6824:/tmp/key ~\Desktop`) 

### 13. 挂载主机文件夹
* 启动容器时使用`-v {host.path}:{container.path}`参数
例如
```
PS C:\docker> docker run -it --privileged=true -p 10032:22 --name test-terraform -v /c/my-notes:/tmp/my-notes test/ubuntu
root@666666666666:/# cd /tmp/my-notes/
root@666666666666:/tmp/my-notes# ls
aws  github  ide  vpn
```
这时已经可以在容器内读写该文件夹中的内容

### 14. 查看docker日志
* `docker ps`找到container信息
* `docker logs {CONTAINER ID}`或`docker logs {NAMES}`查看container日志
