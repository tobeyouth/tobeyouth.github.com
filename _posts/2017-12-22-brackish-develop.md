---
layout: post
title: "开发了一个小网站"
modified: 2017-12-22 14:17:45 +0800
tags: [python, mysql, docker, flask, supervisor]
category: blog
---

最近空余时间一直在开发一个小网站，用到了以下的一些技术栈

- docker
- python
- flask
- mysql
- supervisor
- nginx
- scary

在开发的过程中碰到了不少大大小小的坑，这里权当个记录，如果以后再碰到这样的坑，可以快速的跳出来。


### 项目的结构

首先来说一下项目的结构，初衷是想抓点儿美女图片做个图片站。说到抓数据，首先想到的就是 python 的爬虫框架
`Scary`，事实上也确实使用了 `Scary`。

数据抓回来后，还需要有个网站去展示，所以还需要做个简单的小网站。因为工作中也写一点儿 Python，并且不太喜欢写
nodejs，所以自然而然网站就选择了用 Python 进行开发。

web 框架方面选择了 Flask，原因就是相比 Django
来说比较简单，虽然功能也比较薄弱吧，但是一个小破网站也不需要太多的功能，只要能快速的搭建起来就好了。

所以整个项目被分成了两个部分

- 爬虫
- 展示

因为考虑到爬虫只能算是一个简单功能，所以将爬虫做了单独的部署，也方便之后下线。


### docker

说到开发，在开发 Python 项目时，比较传统的是使用 `virtualenv`，也就是给通过改变环境变量的方式给 Python
虚拟出一个运行环境来。但是考虑到日后如果想要迁移到云服务器上的话，还需要自己手动去安装 mysql 等一系列的依赖，不如直接用
docker 来个一步到到位的部署。

所以还是决定用 docker 来做本地的开发和之后的部署。

第一次接触 docker，也没怎么看文档，直接上手就开始搞，网上的 docker 教程一抓一大把，看了几个之后慢慢的有了一点儿理解。

在Docker 的世界里有 image 和 container 两个概念。

image 就相当于一个手册，Docker 将通过不同的 image 来构建不同的 Container。或者换一个角度，也可以把 image
理解成面向对象中的 class, container 就是通过这个 image 生成出来的实例。

通过不同的 image 我们构建不同的 container，我们可以依赖他人的 image 来生成自己的 image，再通过自己的 image 来生成自己的
container，最终我们的应用就跑在我们生成出来的这个 container 中。


对于我这种只想简单开发和部署的人来说，docker像是一个配置白皮书，只要写好 Dockerfile，就可以方便的将本地开发的环境迁移到任何机器上，安装依赖执行初始化脚本，都可以一键执行完。

既然是一键执行，那一定事先有个配置文件，这个配置文件就是 Dockerfile。
Dockerfile 的作用就是生成 image, 然后就可以根据这个 image 来生成 container，运行应用了。

这里简单的把我的 Dockerfile 列出来，可以对照着这个 Dockerfile 写一份自己的 Dockerfile

```
FROM ubuntu
MAINTAINER tobeyouth <tobeyouth@gmail.com>


# copy from p0bailey/docker-flask
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update

RUN apt-get install -y uwsgi
RUN apt-get install -y python-pip 
RUN apt-get install -y python
RUN apt-get install -y uwsgi-plugin-python
RUN apt-get install -y nginx 
RUN apt-get install -y supervisor
RUN apt-get install -y mysql-server
RUN apt-get install -y curl
RUN apt-get install -y vim

COPY nginx/flask.conf /etc/nginx/sites-available/
COPY supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY app /var/www/app

# install deps
ADD requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt

RUN mkdir -p /var/log/nginx/app /var/log/uwsgi/app /var/log/supervisor \
    && rm /etc/nginx/sites-enabled/default \
    && ln -s /etc/nginx/sites-available/flask.conf /etc/nginx/sites-enabled/flask.conf \
    && echo "daemon off;" >> /etc/nginx/nginx.conf \
    && chown -R www-data:www-data /var/www/app \
    && chown -R www-data:www-data /var/log \
    && RUN mkdir /var/run/mysqld \
    && chown -R mysql:mysql /var/run/mysqld


# node env
RUN apt-get install -y nodejs

CMD ["/usr/bin/supervisord"]
```

首先是 `FROM` 这句，这表示我的运行环境将依赖于一个名字叫 `ubuntn` 的 image, 在这个 image 构建的 container
基础上，会继续添加我自己的一些设置，已完成我对应用运行环境的需求。

`MAINTAINER`就表示了这个 Dockerfile 的作者信息，出了事故可以找作者背锅。

接下来的 `ENV` 就是设计个环境变量, 设置这个环境变量是什么意思，其实我也不知道，去 github
上看了一下，貌似这个是个默认设置，那就这样吧。

在接下来主要就是 `RUN` 和 `COPY` 这些命令，跟名字一样，就是执行一些命令和 copy 一些文件到 container 中。

最后是 `CMD`，这个就是执行初始化的命令，比如我这里执行的就是 `supervisord`，至于什么是 `supervisord`，接下来会再说一下。

写好自己的 Dockerfile 后，需要执行一下`docker build {image_name}:{image_tag} {image_path}` 来生成自己的
image，经过漫长的 build 过程之后，如果不出什么错误，就可以执行 `docker run` 来创建自己的 container 了。
`docker run`
有很多参数，具体可以看[这里](https://docs.docker.com/engine/reference/commandline/run/#options)，当然也可以写个简单的小脚本来执行，比如我的这个

```
#!/bin/bash

IMAGE_NAME='brackish'
CWD=`pwd`
CONTAINER_PORT=80
VIEW_PORT=80

docker run -i -t -d -v ${CWD}/app:/var/www/app -p ${CONTAINER_PORT}:${VIEW_PORT} ${IMAGE_NAME}
```

`docker run` 加了 `-d` 参数，执行之后会输出一个 container_id，这个就是所创建的 container 的 id 了，之后还需要通过这个 id
删除或者访问 container 。不过不小心忘了也没事儿，还可以通过

```
docker container ls -a
```
来查看所有的 container。
