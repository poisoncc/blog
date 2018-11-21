---
layout: post
title: 泛谈docker的几种网络模式
categories: [docker]
description: 简单的说一下自己工作实践中用到过的几种docker网络模式
---

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**

> docker有四种网络模式，分别是Bridge、None、Host、自定义网络，指定网络模式采用--network=xxxx方法。

## 常用的docker网络命令

查看docker网络列表 `docker network list`

查看某个网络的详情 `docker network inspect network_id`

删除某个网络 `docker network rm network_id`

## Bridge模式

> 这是docker默认的网络模式，也是最常用的模式。在这种模式下，docker的容器默认使用安装docker时创建的网络docker0，跟宿主机不在一个网段，不能跟宿主机网段通信，如果要通信只能采取端口映射。

在linux下安装完docker后，`ifconfig`后发现多了一个docker0虚拟网卡，这就是docker默认的网络模式bridge使用的网卡，其网段就是默认容器的网段。

```
docker0   Link encap:Ethernet  HWaddr 02:42:fc:25:ad:61  
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

## None模式

> 在这种模式下，容器无法联网，仅有一个回环网卡lo

这种模式一般不会用，仅在测试或者需要容器绝对隔离的情况下使用，相对于其他网络更加安全，跟linux的lo一样。

## Host模式

> 在这种模式下，容器使用宿主机网络，端口占用宿主机端口。

这种模式一般在测试环境使用，网络隔离性最差，但是网络性能最好，直接用宿主机ip通信。

## 自定义网络

> docker允许自定义三种网络模式，Bridge、Overlay、Macvlan，下面用docker-compose演示

### Bridge模式

> 跟默认的Bridge模式一样，只不过可以自定义网段

在docker-compose中定义网络模式：

` vi docker-compose.yml`

```
version: '2'
services:
  poison:
    image: poison:1.0
    networks:
      poison:
        ipv4_address: 172.25.0.16
    container_name: poison_container
    restart: always

networks:
  poison:
    driver: bridge
    ipam:
      config:
      - subnet: 172.25.0.0/24
```

**不能定义为宿主机网段，会发生冲突造成宿主机网络挂掉**

### Overlay模式

这种模式一般用于docker swarm中，是docker自带的跨主机网络模式。但并不是非要用docker swarm才能使用overlay网络，只是单独使用的话，如果要跨主机通信，需要有一个key-value存储服务，例如etcd等。

### Macvlan模式

> macvlan模式可虚拟出与宿主机同网段的网络，也可用于容器跨主机通信。

**macvlan需要绑定的网卡开启网络混杂模式，如果是virtualbox，需要同时在vb客户端中设置虚拟机网卡混杂，也需要ifconfig设置一下**

查看宿主机是否支持macvlan：

`modprobe macvlan`

`lsmod | grep macvlan`

如果出现类似`macvlan                24576  0`则说明支持。

```
version: '2'
services:
  poison:
    image: poison:1.0
    networks:
      poison:
        ipv4_address: 192.168.100.16
    container_name: poison_container
    restart: always

macvlan
  networks:
    poison:
      driver: macvlan
      driver_opts:
        parent: enp0s3 # 宿主机网卡名称
      ipam:
        config:
        - subnet: 192.168.100.0/24
          gateway: 192.168.100.1
```

但经过测试发现：

1. 宿主机ping不通本宿主机上的容器
2. 宿主机可以ping通其他宿主机上的容器
3. 容器之间可以ping通

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**
