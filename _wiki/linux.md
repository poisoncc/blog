---
layout: wiki
title: Linux
categories: linux
description: linux 常用操作记录。
---

### 查看linux内核版本信息

	uname -a

### 看系统信息

	cat /etc/redhat-release

### 给普通用户root权限

	修改 /etc/passwd 文件，找到如下行，把用户ID修改为 0 ，如下所示：
	
	tommy:x:0:33:tommy:/data/webroot:/bin/bash

### 压缩

	tar -jcv -f xxxx.tar.bz2   xxxx      压缩

	tar -jxv -f xxxx.tar.bz2    解压

	-j对应.bz文件 -z对应.gz文件
	
	zip用unzip解压
	
	压缩拆分

		tar -jcf - logstash-5.0.1.tar.gz |split -b 30m - logstash.tar.bz2

	解压聚合

		cat logstash.tar.bz2a* |tar -jx

### 安装ssh

	apt-get install openssh-server

### 筛选没有`#`的语句

	grep -v '#'    

### 查找文件

	find / -name python

### 筛选文件字段并显示行号

	cat zabbix_server.conf |grep -n 'SSHKey'

### 制作linux启动U盘

	dd if=xxx.iso of=/dev/sdb bs=1M

### 查看文件信息

	stat xxxx.xx

### 查看端口占用情况

	netstat -anp | grep 8080

### 查看进程

	ps -ef |grep logstash

### 查看服务开机自启动

	systemctl list-unit-files |grep rsyslog

	systemctl is-enabled iptables.service

	systemctl is-enabled servicename.service #查询服务是否开机启动

	systemctl enable *.service #开机运行服务

	systemctl disable *.service #取消开机运行

	systemctl start *.service #启动服务

	systemctl stop *.service #停止服务

	systemctl restart *.service #重启服务

	systemctl reload *.service #重新加载服务配置文件

	systemctl status *.service #查询服务运行状态

	systemctl --failed #显示启动失败的服务

### 添加用户并赋予管理员权限

	sudo useradd -m hadoop -s /bin/bash

	sudo passewd hadoop

	sudo adduser hadoop sudo

### 查看文件大小

	查看当前路径下的总文件大小

		du -sh 

		du -sh *** 某文件大小

	查看当前目录下各目录的大小

		du -h --max-depth=1

### 链接curl

	curl -u name:password

### 搜索文件目录里是否包含某个词

	find .|xargs grep -ri "IBM" -l

### 如果make安装遇到没有makefile

	需要先     ./configure

### 手动制造日志

	logger   -p kern.error jj
	
### ubuntu安装桌面

	sudo apt-get install ubuntu-desktop

### 修改LVM 中Volume group Name

	vgdisplay

	vgrename localhost-vg cinder-volumes

### 查看cpu和分区

	lscpu查看cpu

	lsblk查看分区

### ubuntu挂载新数据卷

	查看所有卷

		fdisk -l

	格式化新卷

		mkfs.ext4 /dev/vdb

	挂载新卷到空目录

		mount /dev/vdb esdata

	卸载卷

		umount /dev/vdb

	开机自动挂载

		vi /etc/fstab 

		/dev/vdb esdata ext4 defaults 0 0
		
### linux修改hosts后无法ping通域名，但能ping通ip

	vim /etc/resolv.conf

		8.8.8.8

### linux关闭swap

	vi /etc/sysconfig/selinux

		SELINUX=disabled

	swapoff -a

	vim /etc/fstab

		*remove the swap line*
		
### linux 设置服务开机自启

	systemctl daemon-reload

	systemctl enable elasticsearch.service

	systemctl start elasticsearch.service
	
	systemctl status elasticsearch

### ubuntu卸载软件

	卸载软件

		sudo apt-get remove --purge 软件名称  

		sudo apt-get autoremove --purge 软件名称 

	模糊搜索软件名称

		dpkg --get-selections | grep 软件

	清除卸载残留

		dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P 

	卸载如遇到未知问题

		ps -A | grep apt

		kill -9 xxxxx

### ubuntu安装过的软件，其deb包保存在

	cd /var/cache/apt/archives
	
### Vim 替换
	
	:%s/查找字符/替换字符/g
	
	:%s/“/"/g
