---
title: Initialization Log  
date: 2023-03-01 17:18:00 +0800  
categories: [Github Pages]  
tags: [jekyll-theme-chirpy]  
---
## 1. Overview
Because my operating system for daily use is `Windows 10` and to isolate `Jekyll` environment to other development tools, I determine to set up it in Docker container. This article will introduce:
+ `Jekyll` installation in docker container
+ GitHub Pages with `jekyll-theme-chirpy`

## 2. Prerequisites
+ Docker Desktop is [installed](https://docs.docker.com/desktop/install/windows-install/)

## 3. Create GitHub repository
Refer to [Getting Started](https://chirpy.cotes.page/posts/getting-started/#option-1-using-the-chirpy-starter) to create [hivsuper.github.io](https://github.com/hivsuper/hivsuper.github.io) with the Chirpy Starter as template, then clone it to local.

## 4. Set up `Jekyll` environment

### 4.1 Create docker container
```
docker run -it --privileged=true -p 8080:4000 --name github -v {HOST_PATH}/hivsuper.github.io:{DOCKER_PATH}/hivsuper.github.io ubuntu
```
+ Map the 8080 port of host machine to 4000 in docker container
+ Mount the hivsuper.github.io folder where the codes locate to docker container
 
### 4.2 Install Jekyll
1. Perform the actions on [Jekyll Installation](https://jekyllrb.com/docs/installation/ubuntu/).
> apt-get update  
apt-get install ruby-full build-essential zlib1g-dev  
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc  
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc  
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc  
source ~/.bashrc  
gem install jekyll bundler  
2. Install Dependencies by running `bundle`
```
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# bundle
```

## 5. Configure for GitHub Pages
1. Crop a squre image via [iloveimg](https://www.iloveimg.com/zh-cn/crop-image)
2. Create a round corner image via [image-round](https://www.dute.org/image-round) and create favicons with [favicon generator](https://realfavicongenerator.net/), then follow [tutorial](https://chirpy.cotes.page/posts/customize-the-favicon/) to place them. 
3. [Usage](https://chirpy.cotes.page/posts/getting-started/#usage) to modify files. See [#1](https://github.com/hivsuper/hivsuper.github.io/pull/1) for more details.

## 6. Deploy
After Installing `giscus`, get `data-repo`, `data-repo-id`, `data-category` and `data-category-id` according to [giscus guide](https://vuepress-theme-hope.github.io/v2/comment/guide/giscus.html) and configure them in [#2](https://github.com/hivsuper/hivsuper.github.io/pull/2). Run `bundle exec jekyll s --host 0.0.0.0` to review pages
```
root@ffffffffffff:{DOCKER_PATH}/hivsuper.github.io# bundle exec jekyll s --host 0.0.0.0
...
    Server address: http://0.0.0.0:4000/
  Server running... press ctrl-c to stop.
```
Now http://127.0.0.1:8080/ is accessible in browser in the host machine. If all looks good, push all the changes and merge them. Workflow in GitHub Action will be triggered automatically and check it from https://hivsuper.github.io/.
