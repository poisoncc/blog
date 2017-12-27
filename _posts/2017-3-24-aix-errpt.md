---
layout: post
title: "用ELK收集aix的errpt日志"
comments: true
categories: [logstash, kibana, elasticsearch, aix]
---

领导下达任务，一周内把aix的errpt日志收集起来，本来以为很轻松简单，可实际操作后发现，处处是坑。。。

首先aix的errpt会输出aix的硬件错误日志，这和平时的系统日志不一样，后者只是软件层面的日志。也因为和系统日志不一样，所以不能用平常的syslog的方式收集，后来终于找到一个思路，并验证成功。

思路：在aix上放一个shell脚本，添加定时任务，每五分钟运行一次，shell脚本会运行errpt指令，并抓取前五分钟的errpt日志，通过解析errpt的identifier，然后再运行errpt -aj identifier |more，最后把每一条errpt以及其more信息，组合成一个日志，然后通过logger伪装成系统日志，由syslog发送到远程的ELK中。

## 实际操作：

### shell脚本aix_errpt.sh
```
#!/usr/bin/ksh
#!/bin/sh

rm -rf /usr/local/sbin/errpt
errpt>/usr/local/sbin/errpt

time=`date +%m%d%H%M`
time_y=`date +%Y`
time_late=`expr $time \* 100+$time_y – 2500`

cat /usr/local/sbin/errpt |awk ‘NR>1’ | while read line
do
　　time_even=`echo $line | awk ‘BEGIN{FS=” “}{print $2}’`
　　identifier=`echo $line | awk ‘BEGIN{FS=” “}{print $1}’`
　　if [ $time_even -gt $time_late ]
　　then
　　　　rm -rf /usr/local/sbin/more
　　　　errpt -aj $identifier |more>/usr/local/sbin/more
　　　　more=`cat /usr/local/sbin/more |awk ‘NR>1 && NR<29’`
　　　　message=”$line”” <88> “”$more”
　　　　# echo $message
　　　　logger -p local6.error $message
　　fi
done
```
因为errpt的时间戳格式是月天时分年，所以先是得到系统时间，并把其转换成和errpt的时间戳一样的格式，再减去五分钟，得到time_late。运行errp并保存为errpt文件，cat抓取errpt文件，从第二行开始，因为第一行是抬头属性，然后循环每一行。解析每一条日志的时间戳和identifier，下面到达if代码块，利用-gt把所有在前五分钟发生的日志过滤出来，并利用identifier把该日志的more信息的前29行也添加到该条日志后面。拼接的时候，中间加了<88>，是为了logstash的filter能分别解析出message和more。最后通过logger伪装成local6用户的error发送到syslog中。

### 在aix的syslog配置文件中添加对local6.err的收集。
编辑/etc/syslog.conf，添加
```
local6.err	@ip
```
一定要注意中间是tab键，不是空格，坑了我好久。

编辑/etc/servces，找到syslog，修改其发送的端口，最好别用默认的514，以免和别的冲突。
最后重启syslog进程
```
stopsrc -s syslogd
startsrc -s syslogd
```
### 添加定时任务，每五分钟执行一次aix_errpt.sh
```
crontab -e
0,5,10,15,20,25,30,35,40,45,50,55 * * * * /bin/sh /usr/local/sbin/aix_errpt.sh
```
### 在远程logstash中添加对aix收集的conf。aix_errpt.conf如下：
```
input {
　　udp {
　　　　port => “50016”
　　　　type => errpt
　　}
}

filter{
　　grok {
　　　　match => { “message” => “<%{NONNEGINT:PRI}>%{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{DATA:root} %{DATA:IDENTIFIER} %{DATA:event_time} %{DATA:event_type} %{DATA:event_class} %{DATA:resource_name} %{GREEDYDATA:syslog_message} <%{NONNEGINT:FENGEFU}> %{GREEDYDATA:event_more}” }
　　　　remove_field =>[“message”,”FENGEFU”,”PRI”,”syslog_program”,”syslog_pid”,”root”]
　　}

}

output{
　　# stdout {codec => “rubydebug”}
　　elasticsearch{
　　　　hosts => “localhost:9200”
　　　　index => “aix_errpt-%{+YYYY.MMdd}”
　　}
}
```
运行
```
./logstash_dir/bin/logstash -f aix_errpt.conf &
```
大功告成！在elasticsearch中查看效果吧。

## 遇到的问题：
1.注意定时任务中要运行文件的权限
2.定时任务要运行的文件中，所有路径要写绝对路径
3.aix的crontab 中不支持/符号，所以只能遍历每5分钟执行一次
4.aix的syslog配置文件，@前是tab

关于cron定时任务，如果定时任务没有执行，可查看cron日志，/var/spool/mail/root
