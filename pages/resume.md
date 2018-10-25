---
layout: resume
title: 我的个人简历
description: 隐藏的秘籍
comments: false
keywords: resume
permalink: /resume/
---

# **<center>毛广正的个人简历</center>**

# 个人信息

 - 毛广正 / 男 / 1992
 - 本科 / 天津工业大学 / 软件工程专业
 - 工作年限：2年
 - 联系方式：15620633942
 - E-mail：553780882@qq.com
 - 技术博客：http://blog.poison.cc
 - 期望职业：运维开发工程师 - devops工程师
 - 期望薪资：税前月薪18k~25k
 - 期望城市：北京

---

# 工作经历

## 北京万维星辰科技有限公司 （ 2017年5月 ~ 今 ）

### 万维云平台系统
负责openstack云平台controller高可用的研究部署，ceph存储集群的部署以及openstack与ceph的融合的研究部署工作。在公司中经云机房搭建3constroller+6computer+6ceph集群，公司内部机房搭建1constroller+6computer+6lvm集群；以及日常云平台的运维工作。

### 万维devops系统
独立负责公司devops系统的研究部署工作。公司业务是基于elk+rails的日志分析工作，初期负责elk+rails+
elastalert的docker化以及利用docker-compose和heat快速部署公司运营部所需的所有日志分析系统；后期基于公司需求，使用harbor管理所有docker镜像，使用jenkins管理开发组前端实时push代码后编译docker镜像，并自动更新kubernetes中的前端服务；elasticsearch集群使用k8s管理，根据运营组需求部署master+data+ingest集群，k8s存储也使用ceph集群。

### 万维日志分析系统
负责某些日志的获取和解析工作、系统的搭建维护优化工作以及某些功能开发的工作。探索网络事件、数据库事件、Linux文件变动事件的日志收集和解析；负责elasticsearch集群的搭建、升级、迁移、优化、维护等；使用ruby on rails开发日志详情功能，使用elasticsearch的gem去获取es数据，利用ajax和es的scroll_id特性不断循环遍历出es某些索引里的全部数据，并以瀑布流的形式显示出来。

### 万维日志存储系统
负责某些前端页面的开发工作。使用vue+bootstrap+echarts开发日志上传下载页面，以及日志存储空间统计页面。在初期后端api没有提供的情况下使用python的flask做了简单的api。



## 北京中电汇通科技有限公司 （ 2016年7月 ~ 2017年5月 ）

### ZDOS日志分析告警系统
独立负责日志分析告警工作。把不同类型的主机（包括linux，windows，aix，hp-unix等）的日志集中收集到ELK中，通过解析并抓取部分关键字，结合公司已有的zabbix，提供告警功能；并且在kibana中进行统计展示。期间还做了kibana的汉化工作。linux利用syslog或者rsyslog，windows利用nxlog，把系统日志发送到远程服务器的logstash中进行解析，然后分别发送到elasticsearch和zabbix中，es负责存储分词等让kibana调用进行前台统计展示；zabbix负责监控日志的情况，设置监控项触发器从而起到告警作用。某些日志（比如aix的errpt）需要sh脚本模拟syslog发送到logstash中；某些监控（比如日志重复告警）需要利用python调用es的api发送到zabbix实行监控。

---

# 教育经历
## 天津工业大学 （ 2012年9月 ~ 2016年6月 ）
专业：软件工程

学历：本科

在校本科学习的是JAVA方向，主要学习利用ssh和spring mvc框架的javaweb开发，参加过网站的开发工作。

---

# 致谢
感谢您花时间阅读我的简历，期待能有机会和您共事。
