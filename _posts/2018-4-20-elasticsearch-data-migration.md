---
layout: post
title: 泛谈elasticsearch集群的数据迁移方法
categories: [elasticsearch]
description: 介绍一下自己在工作中用到过得几种es数据迁移方法
---

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**

> 参加工作两年，基本都在跟elk打交道，负责公司内部es集群的部署和运维工作，所以免不了对es集群的数据进行迁移处理。这里介绍三种自己实践过的方案。

## 传统拷贝（不推荐）

> 相继去掉集群中的节点至只有一个节点，然后再去掉副本，最后拷贝的形式迁移到新的集群，再恢复节点副本等到集群健康状态。

这种方案有很多局限性，适用于迁移目的地跟被迁移集群之间无网络的情况。

## reindex迁移（不推荐）

> 使用elasticsearch自带的reindex功能进行迁移。

这种方案其实并不是reindex的本意，它是用来“重构”索引的，速度也超慢；而且当集群里索引太多的话，挨个迁移很是麻烦，还需要在新集群的elasticsearch.yml中添加reindex白名单设置：

`reindex.remote.whitelist: ["123.456.78.9:9200"]`

所以reindex还是用来重构索引吧，合并索引等。

## 过河拆桥（推荐）

> 把目标主机加入集群里，把之前的节点相继node.data: false，最后把源主机退出集群。

这个方法名是我内部叫的，适用于两个集群之间有网络的情况，而且速度很快，实际上跟第一种方案是互补的存在，依据网络情况选择使用第一种还是第二种。

该方法其实来源于elasticsearch的滚动升级，挨个升级各节点的es版本，但当es版本跨度很大时，请慎用，参考es官方升级注意事项：

1. es在6.x之后取消了单索引多type，所以5.x——>6.x如果之前是多type需要reindex之后再升级。

2. 5.0-5.5升级到6.x只能Full cluster restart upgrade，5.6升级到6.x才能Rolling upgrade。

**生产环境最好还是尽量避免数据迁移，特别是数据很多的情况**

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**
