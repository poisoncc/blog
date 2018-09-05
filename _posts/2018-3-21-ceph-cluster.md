---
layout: post
title: ceph集群的搭建
categories: [ceph]
description: 在ubuntu16.04上搭建三个节点组成的ceph集群
---

> 在ubuntu16.04上搭建三个节点组成的ceph集群

## 集群概况

```
	192.168.99.101 ceph1
	192.168.99.102 ceph2
	192.168.99.103 ceph3
```

每个节点有两个盘，一个/dev/sda，一个/dev/sdb

## 创建ceph用户
所有节点创建cephuser用户

```
sudo apt update
sudo su
useradd -m -s /bin/bash cephuser
passwd cephuser
```

给cephuser配置无密码的sudo权限

```
echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
chmod 0440 /etc/sudoers.d/cephuser
sed -i s'/Defaults requiretty/#Defaults requiretty'/g /etc/sudoers
```

## 时间同步
所有节点要时钟同步，详情请看之前的[博文](http://blog.poison.cc/2018/03/06/openstack-ha-environment/#时间同步 "博文")

## 安装open-vm-tools
如果在虚拟机里运行所有节点，需要安装这个虚拟化工具

  apt install -y open-vm-tools

## 安装python和parted

  apt-get install -y python python-pip parted

## 在ceph1节点上配置ssh免秘钥登录其他节点
su - cephuser

ssh-keygen

vim ~/.ssh/config

```
Host ceph1
        Hostname ceph1
        User cephuser
Host ceph2
        Hostname ceph2
        User cephuser
Host ceph3
        Hostname ceph3
        User cephuser
```

chmod 644 ~/.ssh/config

ssh-keyscan ceph1 ceph2 ceph3 >> ~/.ssh/known_hosts

ssh-copy-id ceph1

ssh-copy-id ceph2

ssh-copy-id ceph3

测试：ssh osd1

## 格式化sdb分区
登录到所有节点上，格式化/dev/sdb分区

```
sudo fdisk -f
sudo parted -s /dev/sdb mklabel gpt mkpart primary xfs 0% 100%
sudo mkfs.xfs -f /dev/sdb
sudo fdisk -s /dev/sdb
sudo blkid -o value -s TYPE /dev/sdb
	xfs
```

## 创建ceph集群s(luminous)
在ceph1节点

	su - cephuser

### 配置ceph源
> 不然默认装的ceph-deploy为1.5，目前最新的是2.0，很多命令已弃用改为新的

```
wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -

echo deb https://download.ceph.com/debian-luminous/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
```

> 国内换成清华源安装会很快

```
wget -q -O- 'https://mirrors.tuna.tsinghua.edu.cn/ceph/keys/release.asc' | sudo apt-key add -

echo deb https://mirrors.tuna.tsinghua.edu.cn/ceph/debian-luminous/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
```

### 安装ceph-deploy

	sudo apt install ceph-deploy

  ceph-deploy --version

	mkdir cluster

  cd cluster

  ceph-deploy new ceph1

vim ceph.conf

```
在[global]下添加
public network = 192.168.99.0/24
osd pool default size = 2
mon_allow_pool_delete = true

单节点的话需要增加
osd pool default size = 1
osd crush chooseleaf type = 0

```

`osd pool default size`是每个pg的副本数量
`osd crush chooseleaf type`有两个值，`0`表示以`osd`建立副本，`1`表示以`node`建立副本

### 安装ceph到所有节点

	ceph-deploy install --release luminous ceph1 ceph2 ceph3

> 清华源安装

	ceph-deploy install --release luminous --repo-url https://mirrors.tuna.tsinghua.edu.cn/ceph/debian-luminous/ --gpg-url https://mirrors.tuna.tsinghua.edu.cn/ceph/keys/release.asc ceph1 ceph2 ceph3

### 初始化mon

	ceph-deploy mon create-initial

### 分发秘钥

	ceph-deploy admin ceph1 ceph2 ceph3

在所有节点上运行改变秘钥文件权限

	sudo chmod 644 /etc/ceph/ceph.client.admin.keyring

### 创建管理守护进程
> （12.x以上）

	ceph-deploy mgr create ceph1

### 查看集群状态

	sudo ceph status

### 增加osd到ceph集群
创建osd

	ceph-deploy osd create --data /dev/sdb ceph1

  ceph-deploy osd create --data /dev/sdb ceph2

  ceph-deploy osd create --data /dev/sdb ceph3


### 测试ceph集群
在任意节点运行

sudo ceph health

	HEALTH_OK

sudo ceph -s

### 失败回退

	ceph-deploy purge {ceph-node}
	
	ceph-deploy purgedata {ceph-node}

	ceph-deploy forgetkeys

	rm ceph.*
