---
layout: post
title: "openstack高可用集群手动部署之环境准备（一）"
categories: [openstack]
description: openstack云平台搭建最主要的便是基础环境的搭建了，今天开始把自己搭建的过程分享出来。
---

> openstack云平台搭建最主要的便是基础环境的搭建了，今天开始把自己搭建的过程分享出来。如有不对之处，还请留言纠正。

# openstack高可用集群手动部署之环境准备（一）
## 网络环境
网络，在IT领域中最为重要，openstack也不例外，我这里推荐新手在虚拟机的练习。

> 首先介绍一下我的环境，macbook pro 16G的电脑，安装了virtualbox，五个虚拟机：三个controller（各1.5G内存，40G空间），一个computer（2G内存，20G空间），一个storage（500M内存40G空间，这里用的lvm，没上ceph）。**虚拟机使用的是ubuntu 16.04，openstack安装的是pike版本**。

所有节点三个interface：

	enp0s3是桥接模式，以后用于instance连接外网使用；

	enp0s8是NAT模式，用于部署过程中下载软件使用

	enp0s9是host-only模式，用于各节点之间的交互；

### 更改各节点hosts
vi /etc/hosts

```
127.0.0.1	localhost
127.0.1.1	controller1

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

192.168.99.112 controller1
192.168.99.113 controller2
192.168.99.114 controller3
192.168.99.115 computer1
192.168.99.116 storage1

192.168.99.150 controller
```

**注意`127.0.1.1`后面各节点的不同，`controller`的ip用于ha的vip，所有ip均为enp0s9的网段内**。

### 更改hostname
更改所有节点的`/etc/hostname`值。

### 更改interfaces
vi /etc/network/interfaces
```
source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto enp0s3
iface enp0s3 inet manual
up ip link set dev $IFACE up
down ip link set dev $IFACE down

auto enp0s8
iface enp0s8 inet dhcp
up route add default gw 10.0.3.2 dev enp0s8

auto enp0s9
iface enp0s9 inet static
address 192.168.99.112
netmask 255.255.255.0
gateway 192.168.99.1
up route del default dev enp0s9
```

**up route是因为`route -e`后查看节点默认的网卡是enp0s9，导致虚拟机连不上外网，所以默认网卡改为enp0s8**。

> storage不需要enp0s3，在实际生产环境中可改为特殊的用于存储的万兆网卡网络。

## 虚拟ip
**vip在openstack-ha中至关重要，所有寻找controller的组件/节点，都连接的是vip**。

> 这里使用的是pacemaker，至于keepalived，由于本人没有找到合适的防止脑裂的方法，所以弃用。

### 安装pacemaker

在所有controller节点上安装pacemaker及相关组件：

	apt install pacemaker crmsh corosync cluster-glue resource-agents libqb0

vi /etc/corosync/corosync.conf

```
# Please read the corosync.conf.5 manual page
totem {
	version: 2
	cluster_name: haproxy-prod
	token: 3000
	token_retransmits_before_loss_const: 10

	clear_node_high_bit: yes
	crypto_cipher: none
	crypto_hash: none

	interface {
		ringnumber: 0
		bindnetaddr: 192.168.99.0
		mcastport: 5405
		ttl: 1
	}
}

nodelist {
  node {
    ring0_addr: 192.168.99.112
  }
  node {
    ring0_addr: 192.168.99.113
  }
  node {
    ring0_addr: 192.168.99.114
  }
}

logging {
        fileline: off
        to_stderr: yes
        to_logfile: no
        to_syslog: yes
        syslog_facility: daemon
        debug: off
        timestamp: on
	logger_subsys {
		subsys: QUORUM
		debug: off
	}
}

quorum {
	provider: corosync_votequorum
	expected_votes: 2
}

service {
  name: pacemaker
  ver: 1
}
```

### 设置并查看集群

重启corosync

	/etc/init.d/corosync restart

查看当前节点

	corosync-cfgtool -s

查看集群节点join情况

	corosync-cmapctl runtime.totem.pg.mrp.srp.members

重启pacemaker

	/etc/init.d/pacemaker restart

查看集群情况

	crm_mon -1

设置集群基本属性

	crm configure property pe-warn-series-max="1000" pe-input-series-max="1000" pe-error-series-max="1000" cluster-recheck-interval="5min"

关闭STONITH，如果生产环境设置合适的STONITH

	crm configure property stonith-enabled=false

### 开启vip

开启vip（只在一个节点上运行）

```
crm configure primitive vip ocf:heartbeat:IPaddr2   params ip="192.168.99.150" cidr_netmask="24" op monitor interval="30s"
```

查看vip情况

        crm resource status vip

## 时间同步

所有节点安装ntp服务

	apt install chrony

### controller节点

vi /etc/chrony/chrony.conf

```
server 223.65.211.42 iburst
allow 192.168.99.0/24
```

### other节点

vi /etc/chrony/chrony.conf

```
server controller iburst
allow 192.168.99.0/24
```

查看ntp服务`chronyc sources`

## 配置openstack源

在所有节点上运行

```
apt install software-properties-common

add-apt-repository cloud-archive:pike

apt update && apt dist-upgrade

#如果内核更新的需要重启节点
reboot

apt install python-openstackclient
```

## 安装数据库
请参考之前的[文章](http://blog.nocturne.cc/2018/02/27/mariadb-galera/ "文章")，在三个controller上安装mariadb-galera集群。

## 消息队列
请参考之前的[文章](http://blog.nocturne.cc/2018/02/13/rabbitmq/ "文章")，在三个controller上安装rabbitmq集群。

## Memcached

在所有controller节点上安装memcached

	apt install memcached python-memcache

配置memcached

vi /etc/memcached.conf

	-l 192.168.99.112

添加该节点的具体ip

重启服务

	service memcached restart

