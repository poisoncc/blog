---
layout: post
title: "openstack高可用集群手动部署之keystone"
categories: [openstack]
description: openstack的身份认证组件--keystone的高可用安装
---

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**

> openstack的身份认证组件--keystone的高可用安装,如有不对之处，还请留言纠正。

# openstack高可用集群手动部署之keystone

> keystone组件的安装全部在controller节点上执行。

## 创建数据库
进入mariadb中创建keystone数据库并授权用户keystone操作keystone数据库的权限

```
CREATE DATABASE keystone;

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'openstack_passwd';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'openstack_passwd';

```

**为了方便所有的密码均设置为`openstack_passwd`，当然生产环境还是不要这样了。**

## 安装必要软件
在所有controller节点

	apt install keystone  apache2 libapache2-mod-wsgi

## 配置keystone
修改所有controller的keystone配置文件

vi /etc/keystone/keystone.conf

```
[database]
connection = mysql+pymysql://keystone:openstack_passwd@controller/keystone

[token]
provider = fernet

[catalog]
driver = keystone.catalog.backends.sql.Catalog

[identity]
driver = keystone.identity.backends.sql.Identity
```

## 填充数据库
在其中一个controller上运行

	su -s /bin/sh -c "keystone-manage db_sync" keystone

## 生成秘钥文件
在controller1上运行

```
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone

keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

这两步会在`/etc/keystone/`下生成两个秘钥文件`credential-keys`和`fernet-keys`

把秘钥文件分发到其他controller节点上

```
scp -r /etc/keystone/*-keys xdstar@controller2:~
mv *-keys /etc/keystone/
chown -R keystone:keystone /etc/keystone/credential-keys
chown -R keystone:keystone /etc/keystone/fernet-keys
```

## 引导认证服务

在其中一个controller上运行

```
keystone-manage bootstrap --bootstrap-password openstack_passwd \
  --bootstrap-admin-url http://controller:35357/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```

## 配置apache

每个controller分别设置为该节点的hosname

vi /etc/apache2/apache2.conf

	ServerName controller1

service apache2 restart

## 制作初始化用户环境脚本

vi admin-openrc

```
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=openstack_passwd
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2

```

## 创建demo项目和demo用户
在其中一个controller上运行即可

```
. admin-openrc

openstack project create --domain default --description "Service Project" service

openstack project create --domain default --description "Demo Project" demo

```
设置demo用户密码

	openstack user create --domain default --password-prompt demo

创建demo角色并把demo用户添加进去

	openstack role create user

	openstack role add --project demo --user demo user

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**
