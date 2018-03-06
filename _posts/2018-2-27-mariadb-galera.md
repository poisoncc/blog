---
layout: post
title: "利用galera做mariaDB集群"
comments: true
categories: [mariadb, galera, openstack]
description: ubuntu 16.04利用galera做高可用备份的mariaDB 10.1集群
---

# ubuntu 16.04利用galera做高可用备份的mariaDB 10.1集群

## 安装mariadb
查看系统信息：lsb_release -a
> mariadb在10.1之后集成了galera cluster，之前的版本需要单独安装，节点数量最好三个及以上，因为当两个节点的时候，一个节点down，集群为防止脑裂，会不可用。

	sudo apt-get install software-properties-common

	sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8

	sudo add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://ftp.yz.yamagata-u.ac.jp/pub/dbms/mariadb/repo/10.1/ubuntu xenial main'

	sudo apt-get update

	sudo apt-get install rsync mariadb-server

设置root密码
	
	mysql_secure_installation

	mysql -uroot -padmin

> 如遇到ERROR 1524 (HY000): Plugin 'unix_socket' is not loaded错误（目前是在默认装的mariadb10.0后，升级为10.1后遇到）

	sudo systemctl stop mysqld 
	sudo mysqld_safe --skip-grant-tables &
	mysql -u root
		select Host,User,plugin from mysql.user where User='root';
		update mysql.user set plugin='mysql_native_password';
		update mysql.user set password=PASSWORD("newpassword") where User='root';
		flush privileges;
		exit
	sudo kill -9 $(pgrep mysql)
	sudo service mariadb start
	mysql -u root -p
		install plugin unix_socket soname 'auth_socket';
		exit
		
## 配置galera
### node1节点
```
vi /etc/mysql/conf.d/galera.cnf
```

    [mysqld]
    binlog_format=ROW
    default-storage-engine=innodb
    innodb_autoinc_lock_mode=2
    bind-address=0.0.0.0
    
    # Galera Provider Configuration
    wsrep_on=ON
    wsrep_provider=/usr/lib/galera/libgalera_smm.so
    
    # Galera Cluster Configuration
    wsrep_cluster_name="test_cluster"
    wsrep_cluster_address="gcomm://192.168.99.102,192.168.99.103,192.168.99.104"
    
    # Galera Synchronization Configuration
    wsrep_sst_method=rsync
    
    # Galera Node Configuration
    wsrep_node_address="192.168.99.102"
    wsrep_node_name="node1"
	

```
sudo systemctl stop mysql
sudo systemctl status mysql
galera_new_cluster
```


### node2节点
```
vi /etc/mysql/conf.d/galera.cnf
```
    [mysqld]
    binlog_format=ROW
    default-storage-engine=innodb
    innodb_autoinc_lock_mode=2
    bind-address=0.0.0.0
    
    # Galera Provider Configuration
    wsrep_on=ON
    wsrep_provider=/usr/lib/galera/libgalera_smm.so
    
    # Galera Cluster Configuration
    wsrep_cluster_name="test_cluster"
    wsrep_cluster_address="gcomm://192.168.99.102,192.168.99.103,192.168.99.104"
    
    # Galera Synchronization Configuration
    wsrep_sst_method=rsync
    
    # Galera Node Configuration
    wsrep_node_address="192.168.99.103"
    wsrep_node_name="node2"
	
```
sudo systemctl stop mysql
sudo systemctl status mysql
sudo systemctl start mysql
```
	
### node3节点
```
vi /etc/mysql/conf.d/galera.cnf
```
    [mysqld]
    binlog_format=ROW
    default-storage-engine=innodb
    innodb_autoinc_lock_mode=2
    bind-address=0.0.0.0
    
    # Galera Provider Configuration
    wsrep_on=ON
    wsrep_provider=/usr/lib/galera/libgalera_smm.so
    
    # Galera Cluster Configuration
    wsrep_cluster_name="test_cluster"
    wsrep_cluster_address="gcomm://192.168.99.102,192.168.99.103,192.168.99.104"
    
    # Galera Synchronization Configuration
    wsrep_sst_method=rsync
    
    # Galera Node Configuration
    wsrep_node_address="192.168.99.104"
    wsrep_node_name="node3"
	
```
sudo systemctl stop mysql
sudo systemctl status mysql
sudo systemctl start mysql
```
## 验证集群

    mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"

> 如果这个值跟节点数量一致，则说明集群启动正常，详细的集群信息可查看[官方说明](https://mariadb.com/kb/en/library/galera-cluster-status-variables/ "官方说明")

## 注意事项
如果想myuser使用mypassword从任何主机连接到mysql服务器的话。

    GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;

如果想允许用户myuser从ip为192.168.1.3的主机连接到mysql服务器，并使用mypassword作为密码

    GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.1.3' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;


集群若因重启后未恢复正常，需要

    rm /var/lib/mysql/grastate.dat /var/lib/mysql/galera.cache

然后`galera_new_cluster`其他节点直接`sudo systemctl start mysql`

如果还是开启不了，则`ps -ef | grep mysql` ，然后`kill -9 xxx` 删除mysql进程。

mysql当一个ip太多连接数据库错误时，数据库会把该ip拉黑，连接报错

	ERROR 1129 (HY000): Host '192.168.99.105' is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'

解决方法就是

	mysqladmin -uroot -padmin flush-hosts


