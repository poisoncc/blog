---
layout: post
title: "docker离线安装"
categories: [docker]
description: ubuntu16.04的docker离线安装
---

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**

> 某些客户现场没有外网，但是需要安装docker以便运行自己的服务，这里是ubuntu16.04，其他版本类似，主要是在家里找好依赖包，带到现场。

# ubuntu16.04下docker的离线安装

## docker依赖包下载

> 16.04只依赖libltdl7

ubuntu的deb包在`https://ubuntu.pkgs.org`中可以搜到

libltdl7的[下载页面](https://ubuntu.pkgs.org/16.04/ubuntu-main-amd64/libltdl7_2.4.6-0.1_amd64.deb.html "下载页面")

## docker安装包下载
docker的安装包需要根据ubuntu的版本到docker的[官网](https://download.docker.com/ "官网")下载

注意ubuntu版本所对应的代号

```
Ubuntu不同版本对应的代号如下
trusty 14.04
xenial 16.04
yakkety 16.10
zesty  17.04
```

docker的[下载页面](https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/ "下载页面")

## docker-compose下载
docker-compose需要去github下载
[下载地址](https://github.com/docker/compose/releases "下载地址")，然后下载`docker-compose-Linux-x86_64`。

## 安装docker环境

安装libltdl7，docker

	sudo dpkg -i libltd7.deb
	sudo dpkg -i docker.deb

docker-compose的安装只需要把二进制文件复制到`/usr/local/bin/`下，并赋予权限即可。

	sudo cp docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
	sudo chmod +x /usr/local/bin/docker-compose

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**
