---
layout: post
title: 使用heat编排elasticsearch集群
categories: [openstack, heat, elasticsearch]
description: 使用openstack的编排工具heat编排不同需求的elasticsearch集群。
---

> 由于公司开发需求，需要时常搭建不同需求的es集群，每次手动搭建既费时又费力，而公司内部开发环境在openstack上，es使用docker部署，所以便使用heat去编排不同es集群，很是方便。

**本文前提环境：openstack已安装好heat功能，有自己的es-docker的gitlab库，有自己的docker images库，openstack的基础镜像有docker环境、docker images库的秘钥、gitlab秘钥**

## openstack基础指令

  > 在配置heat的yml文件时需要填写你需要的openstack参数

  1. 获取openstack images的列表，找到自己需要的镜像。

    `openstack image list`

    ```
    +--------------------------------------+----------------------------+--------+
    | ID                                   | Name                       | Status |
    +--------------------------------------+----------------------------+--------+
    | d5c7a4a9-c069-4ff2-9d5a-71eed6147969 | ubuntu-16.04-docker        | active |
    +--------------------------------------+----------------------------+--------+
    ```

  2. 获取openstack flavor的列表，找到自己需要的模板。

    `openstack flavor list`

    ```
    +--------------------------------------+-------------------------+-------+------+-----------+-------+-----------+
    | ID                                   | Name                    |   RAM | Disk | Ephemeral | VCPUs | Is Public |
    +--------------------------------------+-------------------------+-------+------+-----------+-------+-----------+
    | 04004d05-08d9-42d6-85fc-c25894074d34 | 8cpu-16g                | 16384 |   30 |         0 |     8 | True      |
    +--------------------------------------+-------------------------+-------+------+-----------+-------+-----------+
    ```

  3. 获取openstack keypair的列表，找到自己需要的秘钥。

    `	openstack keypair list`

    ```
    +----------------+-------------------------------------------------+
    | Name           | Fingerprint                                     |
    +----------------+-------------------------------------------------+
    | poison         | 3b:65:7a:88:e2:2c:d9:88:b4:88:d3:a2:dc:90:f5:0f |
    +----------------+-------------------------------------------------+
    ```

  4. 获取openstack network的列表，找到自己需要的网络。

    `openstack network list`

    ```
    +--------------------------------------+-------------+--------------------------------------+
    | ID                                   | Name        | Subnets                              |
    +--------------------------------------+-------------+--------------------------------------+
    | 12345678-f682-4eaf-9c47-f7a6af875d15 | outer       | 12345678-3475-472b-aef6-66b5bb6f67c7 |
    | 12345678-8564-4ef7-ac53-e237443c4cec | inner       | 12345678-da1b-43e7-b7b3-97bc4a3e8cf8 |
    +--------------------------------------+-------------+--------------------------------------+
    ```

## heat的配置文件的介绍

  > heat的编排全靠一个yml文件，在里面需要配置好所需的实例，网络等

  ```
  # openstack版本号
  heat_template_version: pike
  # 该yml的介绍，方便管理和查看
  description: Launch a basic instance with CirrOS image using the
               ``m1.tiny`` flavor, ``mykey`` key,  and one network.

  # 全局参数，把上一步获取的你需要的openstack参数声明在这里，只需要修改default就好，description也可以描述一下
  # 如果你不需要全局参数也可以不写，在每个模块直接定义就行
  parameters:
    network:
      type: string
      description: Network ID to use for the instance.
      default: selfservice
    key_name:
      type: string
      label: Key Name
      description: Name of key-pair to be used for compute instance
      default: maogz
    image:
      type: string
      label: Image ID
      description: Image to be used for compute instance
      default: cirros
    flavor:
      type: string
      label: Instance Type
      description: Type of instance (flavor) to be used
      default: m1.nano

  # 这里便是定义资源的地方，浮动ip、volume卷、实例，需要多少定义多少，注意name要不一样，比如instance1、instance2等
  resources:
    floating_ip1:
      type: OS::Nova::FloatingIP
      properties:
        pool: outer # 这里填写网络池，表明浮动ip从哪个网络池获取

    volume1:
      type: OS::Cinder::Volume
      properties:
        size: 10 # 这里填写volume的size，单位是GB

    instance1:
      type: OS::Nova::Server
      properties: # 下面填写实例的配置，可以用get_param获取上面的全局参数，也可以直接定义
        image: { get_param: image }
        flavor: { get_param: flavor }
        key_name: { get_param: key_name }
        security_groups:
          - default
        networks:
          - network: { get_param: network }

    association: # 这里绑定实例和浮动ip
      type: OS::Nova::FloatingIPAssociation
      properties:
        floating_ip: { get_resource: floating_ip }
        server_id: { get_resource: instance }

    volume_attachment: # 这里设置instance连接volume
      type: OS::Cinder::VolumeAttachment
      properties:
        volume_id: { get_resource: volume }
        instance_uuid: { get_resource: instance }

  # 这里设置启动heat编排后的输出，一般看结果就好，可忽略
  outputs:
    instance_name:
      description: Name of the instance.
      value: { get_attr: [ instance, name ] }
    instance_ip:
      description: IP address of the instance.
      value: { get_attr: [ instance, first_address ] }
  ```

## 编排elasticsearch集群

  > 根据上面heat的yml介绍，下面便可以根据需求来直接编排一个elasticsearch的集群了。

  1. 固定内网ip

    > 模板是随机分配ip的，但是搭建es集群需要提前知道实例的ip，便于集群内部通信

    ```
    instance_1:
      type: OS::Nova::Server
      properties:
        name: test_heat_1
        image: { get_param: image }
        flavor: { get_param: flavor }
        key_name: { get_param: key_name }
        security_groups:
          - default
        networks:
          - network: { get_param: network }
            subnet: { get_param: subnet }
            fixed_ip: 172.16.1.11
    ```

  2. 固定浮动ip

    > 如果需要连浮动ip也固定，不用手动绑定的话，先创建出浮动ip，然后直接在实例的networks下填写浮动ip的id即可

    ```
    instance_1:
      type: OS::Nova::Server
      properties:
        name: test_heat_1
        image: { get_param: image }
        flavor: { get_param: flavor }
        key_name: { get_param: key_name }
        security_groups:
          - default
        networks:
          - network: { get_param: network }
            subnet: { get_param: subnet }
            fixed_ip: 172.16.1.11
            floating_ip: 12345678-6b1e-44be-98a2-bf149a4ba860
    ```

  3. 使用cinder存储

    > 默认创建实例不使用cinder持久化boot卷，卷随实例删除而删除，可以设置使用持久卷

    ```
    resources:

  	  volume_1:
  	    type: OS::Cinder::Volume
  	    properties:
  	      name: test_heat
  	      image:  { get_param: image }
  	      size: 10

  	  instance_1:
  	    type: OS::Nova::Server
  	    properties:
  	      name: test_1
  	      block_device_mapping: [{ device_name: "vda", volume_id : { get_resource : volume_1 }, delete_on_termination : "false" }]
  	      flavor: { get_param: flavor }
  	      key_name: { get_param: key_name }
  	      security_groups:
  	        - default
  	      networks:
  	        - network: { get_param: network }
  	          subnet: { get_param: subnet }
            fixed_ip: 172.16.1.11
    ```

  4. 添加外部脚本

    > 这才是本文的重点，添加外部脚本，设置实例开机后运行该脚本，达到编排elasticsearch集群的目的

    ```
    instance_1:
      type: OS::Nova::Server
      properties:
        name: test_heat_1
        image: { get_param: image }
        flavor: { get_param: flavor }
        key_name: { get_param: key_name }
        security_groups:
          - default
        networks:
          - network: { get_param: network }
            subnet: { get_param: subnet }
            fixed_ip: 172.16.1.11
            floating_ip: 12345678-6b1e-44be-98a2-bf149a4ba860
        user_data:
          str_replace:
            template: { get_file: /home/poison/Templates/test.sh }
            params:
              $Nodename: elasticsearch_node_1
    ```

    脚本路径填写openstack主机上的绝对路径，也就是脚本放在该heat.yml同主机中，会自动复制到所有实例中。params可以设置实例的环境变量，达到不同es节点的目的。

    脚本很重要，需要编写好elasticsearch的启动命令，**因为这一步涉及到我docker image库（harbor）和gitlab的隐私，所以简写，理解意思即可，请根据自己实际情况编写脚本**。

    ```
    #!/bin/bash

    # 拉取代码
    cd /home/ubuntu/
    git clone -b xxxx git@xxxx.git --depth=1

    # 配置elasticsearch启动需要的系统环境
    echo 'vm.max_map_count=262144' >> /etc/sysctl.conf
    sysctl -w vm.max_map_count=262144
    cd /home/ubuntu/xxxx

    # 配置elasticsearch的需求
    sed -i 's/-Xms2g/-Xms4g/g' elasticsearch/container-files/jvm.options
    sed -i 's/-Xmx2g/-Xmx4g/g' elasticsearch/container-files/jvm.options
    sed -i 's/node-1/$Nodename/g' elasticsearch/container-files/elasticsearch.yml
    echo 'discovery.zen.ping.unicast.hosts: ["172.16.1.11", "172.16.1.12", "172.16.1.13", "172.16.1.14"]' >> elasticsearch/container-files/elasticsearch.yml
    sed -i 's/#discovery.zen.minimum_master_nodes: 3/discovery.zen.minimum_master_nodes: 3/g' elasticsearch/container-files/elasticsearch.yml

    # mount volume，持久化es数据
    mkfs.ext4 /dev/vdb
    mount /dev/vdb /home/ubuntu/xxxx/elasticsearch/data
    echo '/dev/vdb /home/ubuntu/xxxx/elasticsearch/data ext4 defaults 0 0' >> /etc/fstab

    # 登录docker image库，启动es集群
    docker login -u admin -p xxxx 172.16.1.88:443
    chmod 777 -R elasticsearch/esdata
    docker-compose up -d elasticsearch
    docker logout 172.16.1.88:443

    # delete cfn-userdata，删除脚本记录，防止隐私泄露，不过不知道是否有效。
    rm -rf /var/lib/heat-cfntools/cfn-userdata
    ```

## openstack heat基础命令

  > 编写好heat.yml直接运行openstack命令执行heat编排。

  1. 启动heat编排

    `openstack stack create -t demo-template.yml test_heat`

  2. 删除heat编排

    `openstack stack delete --yes test_heat`

  3. 查看编排列表

    `openstack stack list`
