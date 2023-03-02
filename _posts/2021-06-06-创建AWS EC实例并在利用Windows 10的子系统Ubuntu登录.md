---
title: 创建AWS EC实例并在利用Windows 10的子系统Ubuntu登录  
date: 2021-06-06 22:25:00 +0800  
categories: [Chinese, CNBlogs]   
tags: [linux, cloud]  
---
<a href="https://www.cnblogs.com/hiver/p/14856667.html" target="_blank">点击查看原文</a>  
由于Windows 10升级后无法与VirtualBox兼容，而docker的配置又太过麻烦，今天终于下决心注册一个AWS的帐号。本文介绍：
* 创建AWS帐号
* 安装Windows子系统
* 利用Windows Terminal+子系统登录AWS EC2实例

### 1. 创建及配置AWS帐号和EC2实例

#### 1.1 注册及AWS配置帐号
根据[AWS入门 – 开通海外账户及巧用免费套餐](https://zhuanlan.zhihu.com/p/67478818)教程在AWS官方注册了自己的帐号，选择了基本（Basic）的服务的类型，激活12个月的免费使用期。注册成功需要填个人信息和绑定银行卡，为了避免使用的服务超过限制，打开账单的监控。
<img src="/assets/img/202106/571584-20210606211650085-1637187395.png" width = "800" />

#### 1.2 进入EC2主界面开始创建EC2实例
<img src="/assets/img/202106/571584-20210606221836097-1699370161.png" width = "800" />

#### 1.3 点击“启动新实例”进入配置页面，选择想要的AMI
<img src="/assets/img/202106/571584-20210606212223414-149907117.png" width = "800" />

#### 1.4 选择免费套餐的实例类型
<img src="/assets/img/202106/571584-20210606212308234-76222150.png" width = "800" />

#### 1.5 使用默认的实例配置和存储，暂不添加标签，创建自定义的安全组
注：这里选择“所有流量”做为测试，实际中应该自定义
<img src="/assets/img/202106/571584-20210606212653848-539862405.png" width = "800" />

#### 1.6 审核无问题后启动（若当前无密钥则选择创建新的密钥，并下载到本地环境）
<img src="/assets/img/202106/571584-20210606213208622-185832803.png" width = "800" />

#### 1.7 创建成功后可在列表中找到该新实例
<img src="/assets/img/202106/571584-20210606213617679-1618039588.png" width = "800" />

### 2. 安装Windows子系统
根据[适用于 Linux 的 Windows 子系统安装指南](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10#step-4---download-the-linux-kernel-update-package)安装Ubuntu子系统以及Windows 终端

### 3. 利用Windows Terminal+子系统登录AWS EC2实例
#### 3.1 打开实例复制公网IP和DNS
<img src="/assets/img/202106/571584-20210606221455298-1024582666.png" width = "800" />

#### 3.2 在Windows Terminal中测试AWS EC2的连通性

```
PS C:\Users\test> ping ec2-xx-xx-xx-xx.xx-xx-x.compute.amazonaws.com

正在 Ping ec2-xx-xx-xx-xx.xx-xx-x.compute.amazonaws.com [xx.xx.xx.xx] 具有 32 字节的数据:
来自 xx.xx.xx.xx 的回复: 字节=32 时间=48ms TTL=44
来自 xx.xx.xx.xx 的回复: 字节=32 时间=48ms TTL=44
来自 xx.xx.xx.xx 的回复: 字节=32 时间=47ms TTL=44
来自 xx.xx.xx.xx 的回复: 字节=32 时间=48ms TTL=44

xx.xx.xx.xx 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 47ms，最长 = 48ms，平均 = 47ms
PS C:\Users\test> ping xx.xx.xx.xx

正在 Ping xx.xx.xx.xx 具有 32 字节的数据:
来自 xx.xx.xx.xx 的回复: 字节=32 时间=48ms TTL=44
来自 xx.xx.xx.xx 的回复: 字节=32 时间=48ms TTL=44
来自 xx.xx.xx.xx 的回复: 字节=32 时间=48ms TTL=44
来自 xx.xx.xx.xx 的回复: 字节=32 时间=50ms TTL=44

xx.xx.xx.xx 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 48ms，最长 = 50ms，平均 = 48ms
```

#### 3.3 在Window Terminal中打开Ubuntu子系统，修改下载好的密钥文件权限，利用密钥登录EC2
注：参考 [Windows SSH: Permissions for 'private-key' are too open](https://superuser.com/questions/1296024/windows-ssh-permissions-for-private-key-are-too-open)
```
cp ec2_vpn_secret_600.pem ~/.ssh/ec2_vpn_secret.pem
chmod 600 ~/.ssh/ec2_vpn_secret.pem
ssh -i "~/.ssh/ec2_vpn_secret.pem" xxxx@ec2-xx.xx.xx.xx.xx-xx-x.compute.amazonaws.com
```
登录成功
<img src="/assets/img/202106/571584-20210606220925005-1161620357.png" width = "800" />