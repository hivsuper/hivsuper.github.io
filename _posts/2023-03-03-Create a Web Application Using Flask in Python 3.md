---
title: Create a Web Application Using Flask in Python 3  
date: 2023-03-03 21:23:00 +0800  
categories: [Technology, Python Learning Journey]  
tags: [python, docker]  
---
## Overview
This article is based on [awesome-compose](https://github.com/docker/awesome-compose/tree/master/flask) by adding some S3 operations to code.
<details><summary markdown="span">[app.py](https://github.com/hivsuper/learning-journey/tree/master/demos/aws-demo/flask/app/app.py)</summary>

```
from http import HTTPStatus
from logging.config import dictConfig

import boto3
from flask import Flask

app = Flask(__name__)
dictConfig({
    'version': 1,
    'formatters': {
        'console_fmt': {
            'format': '%(asctime)s %(levelname)s service_name="' + __name__ + '" ' +
                      ' class="%(name)s" event_description="%(message)s"'
        }
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'formatter': 'console_fmt',
            'stream': 'ext://sys.stdout',
            'level': 'INFO',
        }
    },
    'root': {
        'level': 'INFO',
        'handlers': ['console']
    },
    # Disable any existing non-root loggers to prevent any unexpected impact
    'disable_existing_loggers': False
})


@app.route('/', methods=['GET'])
def health_check():
    app.logger.info("call health_check")
    return "Welcome!"


@app.route('/s3-bucket/<name>', methods=['POST'])
def create_s3_bucket(name):
    app.logger.info("create %s", name)
    s3 = boto3.client('s3')
    return s3.create_bucket(Bucket=name)


@app.route('/s3-bucket/<name>', methods=['GET'])
def get_s3_bucket(name):
    app.logger.info("get %s", name)
    s3 = boto3.client('s3')
    return s3.get_bucket_location(Bucket='my-2023-test-bucket')


@app.route('/s3-bucket/<name>', methods=['DELETE'])
def delete_s3_bucket(name):
    app.logger.info("delete %s", name)
    s3 = boto3.client('s3')
    return s3.delete_bucket(Bucket=name)


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)

```
</details>

## References
- [Boto3 Docs 1.26.83 documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html)
- [Try Docker Compose](https://docs.docker.com/compose/gettingstarted/)

## Prerequisites
+ Docker Desktop is [installed](https://docs.docker.com/desktop/install/windows-install/)

## Compose the Docker image
Run `docker compose up -d` in the `flask` folder. It failed when doing it with `Docker version 19.03.13, build 4484c46d9d` on my computer but upgrading to the latest `Docker Desktop` with an engine `Docker version 20.10.22, build 3a2c30b` then rerun succeeded.
```
PS C:\flask\> docker compose up -d
Command "compose up" not available in current context (default)
```
Below is the successful log:
```
PS C:\flask> docker compose up -d
[+] Building 5.4s (16/16) FINISHED
 => [internal] load build definition from Dockerfile                                                                                                                                                                                                                               0.0s
 => => transferring dockerfile: 32B                                                                                                                                                                                                                                                0.0s
 => [internal] load .dockerignore                                                                                                                                                                                                                                                  0.0s
 => => transferring context: 2B                                                                                                                                                                                                                                                    0.0s
 => resolve image config for docker.io/docker/dockerfile:1.4                                                                                                                                                                                                                       2.9s
 => CACHED docker-image://docker.io/docker/dockerfile:1.4@sha256:9ba7531bd80fb0a858632727cf7a112fbfd19b17e94c4e84ced81e24ef1a0dbc                                                                                                                                                  0.0s
 => [internal] load build definition from Dockerfile                                                                                                                                                                                                                               0.0s
 => [internal] load .dockerignore                                                                                                                                                                                                                                                  0.0s
 => [internal] load metadata for docker.io/library/alpine:3.7                                                                                                                                                                                                                      2.1s
 => [internal] load build context                                                                                                                                                                                                                                                  0.0s
 => => transferring context: 1.81kB                                                                                                                                                                                                                                                0.0s
 => [builder 1/7] FROM docker.io/library/alpine:3.7@sha256:8421d9a84432575381bfabd248f1eb56f3aa21d9d7cd2511583c68c9b7511d10                                                                                                                                                        0.0s
 => CACHED [builder 2/7] RUN apk add --no-cache         uwsgi-python3         python3                                                                                                                                                                                              0.0s
 => CACHED [builder 3/7] WORKDIR /app                                                                                                                                                                                                                                              0.0s
 => CACHED [builder 4/7] COPY requirements.txt /app                                                                                                                                                                                                                                0.0s
 => CACHED [builder 5/7] RUN --mount=type=cache,target=/root/.cache/pip     pip3 install -r requirements.txt                                                                                                                                                                       0.0s
 => CACHED [builder 6/7] RUN addgroup -S python && adduser -S 999 -G python                                                                                                                                                                                                        0.0s
 => [builder 7/7] COPY . /app                                                                                                                                                                                                                                                      0.0s
 => exporting to image                                                                                                                                                                                                                                                             0.1s
 => => exporting layers                                                                                                                                                                                                                                                            0.0s
 => => writing image sha256:cb79200b9e16f7b1ba1f35992e8c22102bb85d546bea41ba6cd0ac26e0cab443                                                                                                                                                                                       0.0s
 => => naming to docker.io/library/flask-web                                                                                                                                                                                                                                       0.0s
[+] Running 1/1
 - Container flask-web-1  Started                                                                                                                       0.4s
PS C:\flask>
```
The following image is built by `docker compose`:
```
REPOSITORY                                                                                  TAG                     IMAGE ID       CREATED          SIZE
flask-web                                                                                   latest                  f98f208ab295   30 minutes ago   124MB
```
And the `flask-web-1` web application can be accessed with this call
```
curl --location --request GET 'http://127.0.0.1:8000'
```
Run `docker compose stop` to stop `flask-web-1`
```
PS C:\flask> docker compose stop
[+] Running 1/0
 - Container flask-web-1  Stopped                                                                                                                      0.0s
```
