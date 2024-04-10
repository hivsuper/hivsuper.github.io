---
title: Maven的-pl -am -amd参数学习   
date: 2017-11-17 12:08:00 +0800  
categories: [CNBlogs]   
tags: [java,maven]  
---
<a href="https://www.cnblogs.com/hiver/p/7850954.html" target="_blank">点击查看原文</a>  

------------------------------------------------------------------------

2024-04-10 编辑追加排除模块的方法

------------------------------------------------------------------------
昨天maven的deploy任务需要只选择单个模块并且把它依赖的模块一起打包，第一时间便想到了-pl参数，然后就开始处理，但是因为之前只看了一下命令的介绍，竟然花了近半小时才完全跑通，故记录此文。

假设现有项目结构如下

```
dailylog-parent
|-dailylog-common
|-dailylog-web
```

- 三个文件夹处在同级目录中
- dailylog-web依赖dailylog-common
- dailylog-parent管理dailylog-common和dailylog-web。

根据资料已知：

| 参数	   | 全称	                    | 释义	                                                                                   | 说明                                                 |
|-------|------------------------|---------------------------------------------------------------------------------------|----------------------------------------------------|
| -pl	  | --projects             | Build specified reactor projects instead of all projects                              | 选项后可跟随{groupId}:{artifactId}或者所选模块的相对路径(多个模块以逗号分隔) |
| -am	  | --also-make            | If project list is specified, also build projects required by the list                | 表示同时处理选定模块所依赖的模块                                   |
| -amd  | --also-make-dependents | If project list is specified, also build projects that depend on projects on the list | 表示同时处理依赖选定模块的模块                                    |
| -N    | --Non-recursive        | Build projects without recursive                                                      | 表示不递归子模块                                           |
| -rf   | --resume-from          | Resume reactor from specified project                                                 | 表示从指定模块开始继续处理                                      |

以下是在`maven-3.3.9`中的试验

### 1. 在dailylog-parent目录运行`mvn clean install -pl org.lxp:dailylog-web -am`，结果
```
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for dailylog-parent develop-SNAPSHOT:
[INFO] 
[INFO] dailylog-parent .................................... SUCCESS [  0.266 s]
[INFO] dailylog-common .................................... SUCCESS [  1.937 s]
[INFO] dailylog-web ....................................... SUCCESS [  2.305 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
- dailylog-common成功安装到本地库
- dailylog-parent成功安装到本地库
- dailylog-web成功安装到本地库  

该命令等价于`mvn clean install -pl ../dailylog-web -am`
### 2. 在dailylog-parent目录运行`mvn clean install -pl ../dailylog-common -am`，结果
```
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for dailylog-parent develop-SNAPSHOT:
[INFO] 
[INFO] dailylog-parent .................................... SUCCESS [  0.274 s]
[INFO] dailylog-common .................................... SUCCESS [  1.956 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
- dailylog-common成功安装到本地库
- dailylog-parent成功安装到本地库

### 3. 在dailylog-parent目录运行`mvn clean install -pl ../dailylog-common -amd`，结果
```
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for dailylog-common develop-SNAPSHOT:
[INFO] 
[INFO] dailylog-common .................................... SUCCESS [  2.259 s]
[INFO] dailylog-web ....................................... SUCCESS [  2.045 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
- dailylog-common成功安装到本地库
- dailylog-web成功安装到本地库

<span style="color: rgba(255, 0, 0, 1)">由于dailylog-parent并不依赖dailylog-common模块，故没有被安装</span>
### 4. 在dailylog-parent目录运行`mvn clean install -pl ../dailylog-common,../dailylog-parent -amd`，结果
```
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for dailylog-parent develop-SNAPSHOT:
[INFO] 
[INFO] dailylog-parent .................................... SUCCESS [  0.277 s]
[INFO] dailylog-common .................................... SUCCESS [  2.014 s]
[INFO] dailylog-web ....................................... SUCCESS [  2.032 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
- dailylog-common成功安装到本地库
- dailylog-parent成功安装到本地库
- dailylog-web成功安装到本地库

### 5. 在dailylog-parent目录运行`mvn clean install -N`，结果
```
[INFO] Installing ...\dailylog-parent\pom.xml to ...\.m2\repository\org\lxp\dailylog-parent\develop-SNAPSHOT\dailylog-parent-develop-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
- dailylog-parent成功安装到本地库  

<span style="color: rgba(255, 0, 0, 1)">-N表示不递归，那么dailylog-parent管理的子模块不会被同时安装</span>
### 6. 在dailylog-parent目录运行`mvn clean install -pl ../dailylog-parent -N`，结果
```
[INFO] Installing ...\dailylog-parent\pom.xml to ...\.m2\repository\org\lxp\dailylog-parent\develop-SNAPSHOT\dailylog-parent-develop-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
- dailylog-parent成功安装到本地库

### 7. 在dailylog-parent目录运行`mvn clean install -rf ../dailylog-common`，结果
```
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for dailylog-common develop-SNAPSHOT:
[INFO] 
[INFO] dailylog-common .................................... SUCCESS [  2.264 s]
[INFO] dailylog-web ....................................... SUCCESS [  1.979 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
- dailylog-common成功安装到本地库
- dailylog-web成功安装到本地库

### 8. 在dailylog-parent目录运行`mvn clean install -pl !../dailylog-web,!../dailylog-common`，结果
```
[INFO] Installing ...\dailylog-parent\pom.xml to ...\.m2\repository\org\lxp\dailylog-parent\develop-SNAPSHOT\dailylog-parent-develop-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
- dailylog-web被排除
- dailylog-common被排除
