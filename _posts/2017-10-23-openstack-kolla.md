---
layout: post
title: kolla安装openstack
categories: [openstack, kolla]
description: 利用kolla快速安装openstack
---

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**

# 利用kolla快速安装openstack

## 配置网络环境

虚拟机双网卡，一个桥接enp0s3,一个host-only enp0s8

> 在机器上连接虚拟机，是通过host-only的IP进行访问，因为桥接网络是给openstack的实例访问外网使用的，如果你通过bridge的网络ssh，安装过程，会导致ssh中断。

vi /etc/network/interfaces

```
auto enp0s3
iface enp0s3 inet static
address 10.10.11.118
netmask 255.255.255.0
gateway 10.10.11.1
dns-nameserver 202.106.0.20

auto enp0s8
iface enp0s8 inet dhcp

```

## 安装pip：

```
apt-get update
apt-get install -y python-pip
pip install -U pip
```

## 安装相关的依赖包：

	apt-get install -y python-dev libffi-dev gcc libssl-dev python-selinux

## 安装ansible

> 版本要求Ansible >2.0 ,利用apt安装的后期安装openstack会有错误

	pip install -U ansible

## 配置docker

vi /lib/systemd/system/docker.service

	ExecStart=/usr/bin/dockerd --insecure-registry 10.10.11.118:4000

## 修改docker配置文件

	mkdir -p /etc/systemd/system/docker.service.d

vi /etc/systemd/system/docker.service.d/kolla.conf

```
[Service]
MountFlags=shared
```

	systemctl daemon-reload

	systemctl enable docker

	systemctl restart docker

## 安装ntp

	apt-get install -y ntp

	systemctl enable ntp.service

	systemctl start ntp.service

## 启动docker registry

	docker run -d -v /opt/registry:/var/lib/registry -p 4000:5000 --restart=always --name registry registry:2

去openstack[官网](http://tarballs.openstack.org/kolla/images/ "官网")下载镜像，把下载好的openstack镜像解压到registry的路径下

	tar zxvf ubuntu-source-registry-pike.tar.gz -C /opt/registry/

## 下载并配置kolla

### 下载kolla-ansible源码

	cd /home

	git clone https://github.com/openstack/kolla-ansible.git -b stable/pike

### 安装kolla-ansible

	cd kolla-ansible

	pip install .

	cp -r etc/kolla /etc/kolla/

	cp ansible/inventory/* /home/

> 如果虚拟机

mkdir -p /etc/kolla/config/nova

vi /etc/kolla/config/nova/nova-compute.conf
```
[libvirt]
virt_type=qemu
cpu_mode = none
```

### 生成密码

	kolla-genpwd

### 修改dashboard密码

vi /etc/kolla/passwords.yml

	keystone_admin_password: 123456

### 配置kolla

vi /etc/kolla/globals.yml     

> 5.0.1是镜像版本，修改ubuntu或者centos，选择桥接网络的网卡名称，vip在all-in-one下没有什么价值。

```
kolla_base_distro: "ubuntu"
kolla_internal_vip_address: "10.10.11.120"
kolla_install_type: "source"
openstack_release: "5.0.1"
docker_registry: "10.10.11.118:4000"
docker_namespace: "lokolla"
network_interface: "enp0s3"
neutron_external_interface: "enp0s3"
```

## 安装openstack

kolla-ansible deploy -i /home/all-in-one

安装完毕即可访问openstack http://10.10.11.118

> 如果安装失败，则重新运行deplay就好，但运行之前，要执行下面几步，清空已安装的文件

```
[root@all  ~]# cd /home/kolla-ansible
[root@all tools]# ./cleanup-containers
[root@all tools]# ./cleanup-host
```

## 安装多节点openstack

在部署主机上的`root`下执行`ssh-keygen -t rsa`

把部署主机的` /root/.ssh/id_rsa.pub `添加到所有目标主机的` /root/.ssh/authorized_keys `中

在ansible中ssh目标主机，测试ssh免秘钥是否成功

目标主机上需要执行以上1-6步，到安装ntp为止

在部署主机上修改multinode文件，在controller，computer等下面添加多节点主机

kolla-ansible upgrade -i /home/multinode

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**
