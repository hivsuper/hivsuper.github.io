---
title: Windows 11使用Docker搭建一个简单的Node.js服务  
date: 2024-08-23 00:56:00 +0800  
categories: [Technology]  
tags: [Node.js]  
---
最近接触到一种使用 Node.js 创建服务的方法，该方法支持 HTTPS，个人非常喜欢这种极其轻量级的方案，因此特地记录此文。

> 本文的前提是本地Docker环境已经配置成功，参考[Windows 10安装Docker并使用私钥连接AWS EC2](/posts/Windows-10安装Docker并使用私钥连接AWS-EC2/)
{: .prompt-tip }

## 1. 安装Node.js
```shell
docker pull node:20-alpine
```

## 2. 启动Node.js容器
```shell
docker run -it --privileged=true -p 3000:3000 --name node-dev -v {PATH}\study-nodejs:/tmp/study-nodejs node:20-alpine
```

## 3. 创建服务
在挂载的目录`{PATH}\study-nodejs`创建相应的文件
- index.html  

```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
    <title>Simple Node.js Server</title>  
</head>  
<body>  
    <h1>Welcome to my simple server!</h1>  
    <p>This is a simple HTML page served using Node.js.</p>  
</body>  
</html>
```

- server.js  

```js
const express = require('express');  
const path = require('path');  

const app = express();  
const PORT = 3000;  

// 提供静态文件  
app.use(express.static(path.join(__dirname)));  

// 路由处理  
app.get('/', (req, res) => {  
    res.sendFile(path.join(__dirname, 'index.html'));  
});  

// 启动服务器  
app.listen(PORT, () => {  
    console.log(`Server is running at http://localhost:${PORT}`);  
});
```

## 4. 配置并启动
进入容器 `node-dev` 执行如下命令：  
```
cd /tmp/study-nodejs/
npm init -y
npm install express
node server.js
```
![Commands In Docker](/assets/img/202408/NodeJs-1.png){: width="800" }

## 5. 验证服务
![Commands In Docker](/assets/img/202408/NodeJs-2.png){: width="800" }

## 6. 安装openssl并创建证书
```
apk upgrade --update-cache --available && apk add openssl && rm -rf /var/cache/apk/*
openssl req -nodes -new -x509 -keyout server.key -out server.cert
```
![Commands In Docker](/assets/img/202408/NodeJs-3.png){: width="800" }

## 7. 修改服务
- server.js  

```js
// server.js  
const express = require('express');  
const path = require('path');  
const https = require('https')
const fs = require('fs')

const app = express();  
const PORT = 3000;  

const options = {
    key: fs.readFileSync('server.key'),
    cert: fs.readFileSync('server.cert')
}

// 提供静态文件  
app.use(express.static(path.join(__dirname)));  

// 路由处理  
app.get('/', (req, res) => {  
    res.sendFile(path.join(__dirname, 'index.html'));  
});  

// 启动服务器  
https.createServer(options, app).listen(PORT, () => {  
    console.log(`Server is running at https://localhost:${PORT}`);  
});
```

## 8. 启动服务并验证
```
/tmp/study-nodejs # node server.js 
Server is running at https://localhost:3000
```
![Commands In Docker](/assets/img/202408/NodeJs-4.png){: width="800" }

## 9. 总结
本文介绍了如何使用Docker搭建一个简单的Node.js服务并开启本地HTTPS，完整的源代码可以在[GitHub](https://github.com/hivsuper/learning-journey/tree/master/study-nodejs)中查看。