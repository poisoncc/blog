---
layout: post
title: rabbitmq集群的搭建
categories: [rabbitmq, openstack]
description: 在ubuntu16.04中搭建rabbitmq集群，供openstack使用。
---

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**

# rabbitmq集群的搭建

> 在ubuntu16.04中搭建rabbitmq集群，供openstack使用，此例三个节点。

## 安装rabbitmq

	apt-get install rabbitmq-server

## 配置rabbitmq

在其中一个节点上

	cat /var/lib/rabbitmq/.erlang.cookie

	记住这个值

再到另一个节点上
```
service rabbitmq-server stop
rm /var/lib/rabbitmq/.erlang.cookie
vi /var/lib/rabbitmq/.erlang.cookie
	把另一个节点的值写入，保存

chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
chmod 400 /var/lib/rabbitmq/.erlang.cookie

ls -lh /var/lib/rabbitmq/.erlang.cookie
	-r-------- 1 rabbitmq rabbitmq 21 Jan 16 19:39 /var/lib/rabbitmq/.erlang.cookie

service rabbitmq-server start
rabbitmqctl stop_app

rabbitmqctl join_cluster rabbit@controller1
rabbitmqctl start_app

查看集群情况
	rabbitmqctl cluster_status

```

回到刚才的节点运行

	rabbitmqctl cluster_status

	可以看到第二个节点已加入

重复操作把第三个节点也加入集群

所有节点运行

	rabbitmqctl set_policy ha-all '^(?!amq\.).*' '{"ha-mode": "all"}'

重启服务
	rabbitmqctl stop_app

	rabbitmqctl reset

	rabbitmqctl start_app

修改集群名字

	rabbitmqctl set_cluster_name rabbit@controller

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**
