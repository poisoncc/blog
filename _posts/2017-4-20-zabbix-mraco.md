---
layout: post
title: "zabbix触发器名称宏变量被省略"
comments: true
categories: [zabbix]
description: 关于zabbix首页触发器名称包含宏变量被省略的问题
---

# 关于zabbix触发器名称包含宏变量被省略的问题

## 场景
这个问题出现的场景是，领导让我监控服务器被链接USB后，zabbix上显示提示告警。环境仍然是我用logstash收集服务器日志，分别发送到elasticsearch和zabbix中，监控采用的是logstash-output-zabbix方式，然后在zabbix上设置监控项和触发器。但由于logstash收集的日志经过过滤筛选后，value可能依然有点长，毕竟需要把日志情况表达清楚。

## 出现问题
然后到zabbix后，首页需要显示出来，但问题这时就出现了，我在触发器的详细描述里直接添加了${LASTVALUE}这个宏变量，但在前端给我只显示了部分字符，后面用三个点代替了。我的天，这怎么行，我需要客户看清楚具体是什么USB设备链接了主机啊，而且后面很多的监控套路都需要完全显示宏变量。

## 思考问题
后面开始解决这个问题，首先我想到的是部门里的前端同事，他给我一个提示是：text-overflow，因为他在巡检数据的页面通过chrome的F12找页面属性找到了这个属性，通过chrome修改确实在巡检数据页面可以完全显示，可我要的是首页仪表盘的完全显示，我在前端代码中找了半天无果，因为zabbix的css好像是自动生成的，因为对zabbix前端无所了解，所以无从下手。只能在有关触发器的那些文件中碰，搜索可能会出现的关键字。

后来思考了以下，这样的思路是不正确的，因为zabbix不仅仅是首页调用宏变量给省略了，其他地方的调用也省略了，可能是全局设置宏变量属性了，于是开始搜索查看关于宏变量mraco的文件，半天后，依然无果。

## 解决问题

就在快要放弃的时候，突然意识到，他会不会省略的字符长度是一样的？确实通过验证zabbix所有省略的宏变量都是一样的长度：20个字符。这下思路一下来了，搜索类似length：20的字符串。

终于在./include/items.inc.php文件中找到如下代码：
```
switch ($item['value_type']) {
　　　　case ITEM_VALUE_TYPE_STR:
　　　　　　　　$mapping = getMappedValue($value, $item['valuemapid']);
　　　　// break; is not missing here
　　　　case ITEM_VALUE_TYPE_TEXT:
　　　　case ITEM_VALUE_TYPE_LOG:
　　　　　　　　if ($trim && mb_strlen($value) > 20) {
　　　　　　　　　　　　$value = mb_substr($value, 0, 20).'…';
　　　　　　　　}

　　　　　　　　if ($mapping !== false) {
　　　　　　　　　　　　$value = $mapping.' ('.$value.')';
　　　　　　　　}
　　　　　　　　break;
　　　　default:
　　　　　　　　$value = applyValueMap($value, $item['valuemapid']);
}
```
注意那个20的逻辑，o(∩_∩)o…哈哈，终于找到你，还好我没放弃。通过修改，前端宏变量果然全部显示。

通过这个问题，感觉这个思考过程很特殊，所以记录下来，并把此问题的解决方法告诉大家。
