---
layout: post
title: elasticsearch的swarm集群
categories: [docker, swarm, elasticsearch]
description: 利用docker swarm搭建elasticsearch集群
---

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**

# 利用docker swarm搭建elasticsearch集群

废话不多说直接上Dockerfile

```
FROM alpine:latest
RUN apk update \
    && apk upgrade \
    && apk add curl wget bash openssl openjdk8 \
    && rm -rf /var/cache/apk/*
WORKDIR /root/
RUN wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.0.tar.gz -O elasticsearch-5.6.0.tar.gz
RUN tar -xf  elasticsearch-5.6*.tar.gz -C /usr/local/ \
    && mv /usr/local/elasticsearch-5.6* /usr/local/elasticsearch \
    && mkdir /usr/local/elasticsearch/logs \
    && mkdir /usr/local/elasticsearch/data \
    && echo '-Xms512m' > /usr/local/elasticsearch/config/jvm.options \
    && echo '-Xmx512m' >> /usr/local/elasticsearch/config/jvm.options \
    && adduser -D -u 1000 -h /usr/local/elasticsearch elasticsearch \
    && chown -R elasticsearch /usr/local/elasticsearch
USER elasticsearch
CMD ["/usr/local/elasticsearch/bin/elasticsearch", "-Ecluster.name=es-cluster", "-Enode.name=${HOSTNAME}", "-Epath.data=/usr/local/elasticsearch/data", "-Epath.logs=/usr/local/elasticsearch/logs", "-Enetwork.host=0.0.0.0", "-Ediscovery.zen.ping.unicast.hosts=es-master"]
EXPOSE 9200 9300
```

搭建镜像

	docker build -t 'es:5.6' .

创建swarm网络

	docker network create --driver=overlay appnet

创建elasticsearch服务

```
docker service create --detach=true --name es-master -p 9200:9200 --network appnet --replicas 1 --with-registry-auth es:5.6
docker service create --detach=true --name es-data-1  --network appnet --replicas 1 --with-registry-auth es:5.6
docker service create --detach=true --name es-data-2  --network appnet --replicas 1 --with-registry-auth es:5.6
docker service create --detach=true --name es-data-3  --network appnet --replicas 1 --with-registry-auth es:5.6
```

查看es服务运行状态

```
docker service ls -f name=es
curl -XGET http://127.0.0.1:9200/_cluster/health?pretty
curl -XGET http://127.0.0.1:9200/_cat/nodes?v
```

一个简单的es集群便搭建好了，当然具体的副本数量，es的jvm大小，es的数据持久化等细节都没做，以后慢慢补上。

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**
