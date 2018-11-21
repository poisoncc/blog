---
layout: post
title: openstack实例系统分区read-only解决方法
categories: [openstack]
description: 由于运维同事对网络的错误操作导致公司内部openstack所有实例服务挂掉。。。
---

> 起因是一个运维同事接到升级公司内网的任务后，没有通知我们，私自升级内网，openstack防止数据丢失进行读写保护，导致所有实例的系统分区read-only，所有服务无法写数据而挂掉，命令执行不了，cat /etc/mtab查看系统分区/dev/vda1变为ro。

当然这个原因也是查证了半天，也是因为公司内openstack中的instance是用来开发测试的，所以这次事故影响很小。对了，公司内部的openstack用的lvm做的存储。从此当物业停电，网络升级之前，我们都先停掉所有实例，防止该事件再次发生。当然正式环境肯定不能允许停电或者网络变化这种事情的发生。

## 定位实例所在的computer

通过instance_id在controller上运行`nova show 3fecc6e3-ca78-4ab3-9956-fd6db151b8e8`获得实例所在的计算节点以及他的instance_name。

```
| OS-EXT-SRV-ATTR:host                 | compute4                                                                         |
| OS-EXT-SRV-ATTR:hostname             | ansible-client1                                                                  |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | compute4                                                                         |
| OS-EXT-SRV-ATTR:instance_name        | instance-0000025a
```

## 定位实例的盘符

在实例所在的计算节点运行`virsh domblklist instance-0000025a`获得系统盘盘符名：

```
Target     Source
-----------------------
vda        /dev/sdd
```

## 修复分区

关闭实例后，在计算节点运行`fsck -f -y /dev/sdd1`修复分区，启动实例后系统分区可读写，服务启动正常。
