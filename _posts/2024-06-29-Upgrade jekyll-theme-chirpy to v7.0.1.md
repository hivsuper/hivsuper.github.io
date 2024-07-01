---
title: Upgrade jekyll-theme-chirpy to v7.0.1  
date: 2024-06-29 21:19:00 +0800  
categories: [Github Pages]  
tags: [jekyll-theme-chirpy]  
---
## 1. Overview
The document outlines the process of upgrading `jekyll-theme-chirpy` to <u>v7.0.1</u> from starter.

## 2. Prerequisites
+ The Github Pages project has been set up with `jekyll-theme-chirpy` [v5.5.2](/posts/Initialization-Log/)

## 3. Update the changes
- Compare the [changes](https://github.com/cotes2020/chirpy-starter/compare/v5.5.2...v7.0.1?diff=split&w=) between <u>v5.5.2</u> and <u>v7.0.1</u>
- Update files accordingly and [commit](https://github.com/hivsuper/hivsuper.github.io/commit/dfbae45d1ec603dda17ccaa2bdab1cbaa7a5ace6)

## 4. Upgrade
> The local container is running on `Ubuntu 22.04.3 LTS` which contains `Ruby-3.0.0` by default, nevertheless the `jekyll-theme-chirpy` v7.0.1 requires higher version. Here introduce `rvm` to upgrade `Ruby`.
{: .prompt-warning }

### 4.1 Install rvm
Run the commands below
```shell
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# apt-get install software-properties-common
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# apt-add-repository -y ppa:rael-gc/rvm
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# apt-get update
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# apt-get install rvm
```
### 4.2 Upgrade Ruby
> Need to run `/bin/bash --login` if seeing "RVM is not a function" error below
![rvm-default](/assets/img/202406/rvm-default.png){: width="800" }
{: .prompt-tip }
```shell
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# rvm install "ruby-3.3.3"
...
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# rvm alias create default 3.3.3
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# /bin/bash --login
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# rvm default
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# ruby -v
ruby 3.3.3 (2024-06-12 revision f1c7b6f435) [x86_64-linux]
```
### 4.3 Upgrade jekyll-theme-chirpy
```shell
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# bundle update jekyll-theme-chirpy
Fetching gem metadata from https://rubygems.org/.........
Resolving dependencies...
Resolving dependencies...
...
Fetching jekyll-theme-chirpy 7.0.1
Installing jekyll-theme-chirpy 7.0.1
Bundler attempted to update jekyll-theme-chirpy but its version stayed the same
Bundle updated!
```
## 5. Deploy in new terminal
```shell
root@ffffffffffff:/# cd {DOCKER_PATH}/hivsuper.github.io
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# /bin/bash --login
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# rvm default
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# bundle exec jekyll s --host 0.0.0.0
...
    Server address: http://0.0.0.0:4000/
  Server running... press ctrl-c to stop.
```

## 6. References
- [Upgrade Guide](https://github.com/cotes2020/jekyll-theme-chirpy/wiki/Upgrade-Guide)
- [Setting the default Ruby](https://rvm.io/rubies/default)
- [RVM package for Ubuntu](https://github.com/rvm/ubuntu_rvm)
