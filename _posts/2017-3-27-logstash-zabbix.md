---
layout: post
title: "zabbix接受logstash数据并告警"
comments: true
categories: [logstash, zabbix]
description: 利用logstash+zabbix实现日志告警
---

接上一篇blog，日志是收集到了，但日志除了利用elasticsearch和kibana来统计做图外，还实时反应了系统已出现的error or warning。所以利用logstash实现日志告警是件很有意义的事情。这里由于本人也经常跟zabbix打交道，所以首先想到的便是zabbix，而恰好logstash的output模块支持输出到zabbix，让这件事变得很顺利。

logstash的output默认不自带zabbix插件，需要手动安装，在logstash的安装目录下：
```
bin/logstash-plugin install logstash-output-zabbix
```

### 编写logstash配置文件
这里用的是抓取linux的rsyslog传过来的日志，利用filter把普通级别的日志过滤掉，剩下error，warning等高级别的日志传入zabbix中，还要注意日志信息的合并，把发生日志的主机ip，日志等级，日志内容组成一条message，传进zabbix，然后配置zabbix，设置触发器，当主机发生高级别日志时，产生告警。
```
input {
　　tcp {
　　　　port => "50024"
　　}
}
filter {
　　grok {
　　　　match => { "message" => "<%{NONNEGINT:PRI}>(%{SYSLOGTIMESTAMP:syslog_timestamp}) %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
　　　　#删除多余的field
　　　　remove_field =>["syslog_timestamp","message","@version","tags","port","syslog_facility","syslog_hostname","syslog_program"]
　　　　}
　　syslog_pri{
　　　　syslog_pri_field_name => "PRI"
　　　　remove_field =>["PRI","syslog_facility_code","syslog_severity_code","syslog_facility"]
　　　　#增加field作为logstash输出到zabbix的数据zabbix_value,只能为string类型，所以需要通过引用已有字段并合并。原有的合并功能mutate-convert合并后为数组类型
　　　　add_field => [ "zabbix_message" , "%{host},%{syslog_severity},%{syslog_message}" ]
　　　　#增加field，zabbix_key为zabbix识别关键字，需和zabbix添加监控项时的key一致
　　　　add_field => [ "zabbix_key" , "%{syslog_severity}" ]
　　　　#增加field，zabbix_host需要和zabbix添加的主机名一致
　　　　add_field => [ "zabbix_host" , "logstash" ]
　　}
　　#去掉没用的低级别类型的日志
　　if [syslog_severity] == "debug" or [syslog_severity] == "informational" or [syslog_severity] == "notice" {
　　　　drop {}
　　}
　　if [tags] != [] {
　　　　drop {}
　　}
}
output{
　　zabbix {
　　　　zabbix_host => "zabbix_host"
　　　　zabbix_key => "zabbix_key"
　　　　zabbix_server_host => "localhost"
　　　　zabbix_server_port => "10051"
　　　　zabbix_value => "zabbix_message"
#　　　　multi_value => ["host","syslog_message"]
　　}
}
```
注意zabbix_host,zabbix_key,zabbix_value三个字段，启动配置文件/bin/logstash -f zabbix.conf

### 在zabbix中配置主机：

#### 添加主机
名字为logstash配置文件中设置的zabbix_host，ip默认127.0.0.1，端口默认10050
#### 添加监控项
监控项类型为Trapper，键值为logstash配置文件中的zabbix_key，信息类型为文本TXT
#### 添加触发器
名字为error : {ITEM.LASTVALUE}，表达式，目前还没找到合适的触发器，所以设置为只要获取的数据字符串长度大于零就触发


