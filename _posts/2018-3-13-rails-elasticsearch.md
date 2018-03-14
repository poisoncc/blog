---
layout: post
title: rails利用scroll读取elasticsearch数据
categories: [rails, elasticsearch, ajax]
description: rails读取elasticsearch的某个索引数据，并利用ajax通过scroll把数据全部遍历出来
---

> rails读取elasticsearch的某个索引数据，并利用ajax通过scroll把数据全部遍历出来。

## 用到的Gem

	gem 'elasticsearch', git: 'git://github.com/elasticsearch/elasticsearch-ruby.git'

	gem 'jquery-rails'

## 读取elasticsearch某个索引

> **整个流程的思路是：首先在`view`中做一个表格，用于呈现es的数据，`@total_count`是索引的数据条数，`@esdate`是首次搜索es返回的数据，首次返回为100条，这个数字在`controller`中的`size`中设定；首次搜索加了`:scroll = 5m`定义`scroll_id`有效期为5m，返回的`scroll_id`会被`controller`的`externe_es`方法调用，通过`scroll_id`再次去es中获取100条数据，如此反复，直到取完。`controller`获取es的数据是通过`require './lib/es/client'`实现的，里面用`init`定义es的连接ip，`search`方法便是第一次搜索，`scroll`方法便是后面循环用到的。不跳转直接在页面下面不断增量数据用到的是`ajax`技术，在`view`的`index`中添加一段`javascript`，检测我当前滚动的位置与页面底部的距离，如果小于等于`400px`，则利用ajax访问`/externe_es_ajax`,这个在`route`中定义为通过`get`方法去获取`poisons#externe_es`数据，而`poisons#externe_es`则是把`@externe_es`传给`table`,再把`table`嵌套进`index`的`<div id="taskform"></div>`。在`javascript`中判断当`scrollCount == <%= @total_count/100 %>`时不再获取数据。**

### view
vi app/views/poisons/index.html.erb

```
<h1>产品列表</h1>

<div>总共 <%= @total_count %> 条记录</div>

<div>
    <% if @total_count > 0 %>
      <div>
        <table>
          <thead>
            <tr>
              <th>品牌</th>
              <th>型号</th>
              <th>产地</th>
              <th>价格</th>
              <th>生产日期</th>
            </tr>
          </thead>
          <tbody>
            <% @esdate.each do |nocturne| %>
              <tr>
                <td><%= nocturne['_source']['name'] %></td>
                <td><%= nocturne['_source']['model'] %></td>
                <td><%= nocturne['_source']['origin'] %></td>
                <td><%= nocturne['_source']['price'] %></td>
                <td><%= nocturne['_source']['production_date'] %></td>
            <% end %>
          </tbody>
        </table>
      </div>

      <div id="taskform"></div>
</div>


<script type="text/javascript">
  var scrollCount=0;
  var ajax_result=1;
  <% if @total_count > 0 %>
      $(window).scroll(function() {
        if ($(document).height() - $(this).scrollTop() - $(this).height() <= 400 && ajax_result == 1) {
          if (scrollCount == <%= @total_count/100 %>){
            $(this).off("scroll");
            return false;
          }
          ajax_result=0;
          $(document).ready(function(){
              scrollCount=scrollCount+1;
              $.ajax({
                  url: "/externe_es_ajax",
                  success: successFnt,
              });
              function successFnt(){
                  ajax_result=1;
              }
          });
        }
      });
  <% end %>
</script>

```

vi app/views/poisons/externe_es.js.erb

```
$('#taskform').append("<%= j render(partial: 'table', locals: { externe_es: @externe_es }) %>");
```

vi app/views/poisons/_table.html.erb

```
<% if externe_es.present? %>
  <div>
    <table>
      <tbody>
        <% externe_es.each do |nocturne| %>
          <tr>
            <td><%= nocturne['_source']['name'] %></td>
            <td><%= nocturne['_source']['model'] %></td>
            <td><%= nocturne['_source']['origin'] %></td>
            <td><%= nocturne['_source']['price'] %></td>
            <td><%= nocturne['_source']['production_date'] %></td>
        <% end %>
      </tbody>
    </table>
  </div>
<% end %>
```



### lib/es
> 增加一个elasticsearch的module

vi lib/es/client.rb

```
require 'elasticsearch'
module ES
  class Client
    def self.init
      @client = Elasticsearch::Client.new hosts: ['192.168.65.100:9200'], reload_on_failure: true
      puts @client
    end

    def self.search(params, options = {})
      self.init unless @client
      options[:index] = 'nocturne'
      options[:scroll] = '5m'
      options[:body] = params if params.present?
      @client.search(options)
    end

    def self.scroll(params, options = {})
      self.init unless @client
      options[:body] = params
      @client.scroll(options)
    end

  end
end
```

### model

vi app/model/poison.rb

```
class Poison < ApplicationRecord
end
```

### controller

vi app/controllers/poisons_controller.rb

```
require './lib/es/client'

class PoisonsController < ApplicationController

  def index
    resp = ES::Client.search(es_params)
    @total_count = resp['hits']['total']
    @esdate = resp['hits']['hits']
    @@scroll_id = resp['_scroll_id']
  end

  def show

  end

  def externe_es
    resp_other = ES::Client.scroll(es_scroll_params)
    @externe_es = resp_other['hits']['hits']
    render 'externe_es.js'
  end

  private
    def es_params
      query_params = { bool: { must: [] } }

      page = params[:page].to_i
      page -= 1 if page > 0

      { query: query_params, size: 100 }
    end

    def es_scroll_params
      { scroll: "5m", scroll_id: @@scroll_id }
    end
end

```

### route

在routes.rb中加入poison和externe_es_ajax

```
resources :poisons

match '/externe_es_ajax', to: 'poisons#externe_es', via:'get'
```

### 引用jquery

> 默认rails添加了jquery的gem页面还是引用不了jquery的，需要在`application.js`中添加如下两行：

```
//= require jquery
//= require jquery_ujs
```
