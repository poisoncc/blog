---
layout: post
title: "docker static ip"
categories: [docker]
description: docker静态ip的利用
---

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**

> docker容器的网络在默认情况下会使用bridge模式，其ip地址是dhcp获取的，而在某些场景下，需要固定docker容器的ip。

> 这里使用docker-compose工具编排容器。

直接奉上docker-compose.yml

```
version: '2'
services:
  mysql:
    image: daocloud.io/mysql:latest
    environment:
        - MYSQL_ROOT_PASSWORD=root
    container_name: mysql
    networks:
      nocturne:
        ipv4_address: 172.25.0.15
    restart: always
networks:
  nocturne:
    driver: bridge
    ipam:
      config:
      - subnet: 172.25.0.0/24
```

首先在network中设置自己定义的网络nocturne，设置subnet，注意driver使用bridge；

然后在具体要固定ip的service下的networks设置为上一步定义的nocturne，然后设置subnet中某个具体的ip。

这样当docker-compose起容器的时候会先创建定义的bridge网络，然后再起容器。

使用`docker inspect mysql`查看容器ip是否为`172.25.0.15`。

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**
