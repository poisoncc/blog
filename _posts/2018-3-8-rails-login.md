---
layout: post
title: rails环境搭建以及简单登录
categories: [rails]
description: "在ubuntu16.04下搭建ruby on rails环境，做一个简单登录的权限控制"
---

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**

> 在ubuntu16.04下搭建ruby on rails环境，做一个简单登录的权限控制。

## rails环境搭建
### 下载rvm用于管理ruby

	curl -L https://get.rvm.io | bash

	需要重新登录以及source /etc/profile.d/rvm.sh

	rvm --version

### 下载ruby

```
rvm install 2.3.1

rvm use 2.3.1 --default # 把2.3.1设为默认版本

ruby --version

gem sources --remove https://rubygems.org/ # 删除官方源

gem sources -a https://ruby.taobao.org/ # 添加淘宝源

gem sources -l # 查看gem源

gem sources -u # 清除缓存
```

### 下载rails

	gem install rails

	rails -v

### 错误问题说明

> 遇到这个错误时说明没有安装nodejs

	Bundler::GemRequireError: There was an error while trying to load the gem 'uglifier'.

	apt-get install nodejs

> 遇到这个错误说明需要production的secret_key_base

	#<RuntimeError: Missing `secret_key_base` for 'production' environment, set this value in `config/secrets.yml`>

	rake secret RAILS_ENV=production

	把生成的值填入./config/secrets.yml

## 搭建简单的项目

### 创建项目

	rails new nocturne

	cd nocturne

### 添加需要的gem
vi Gemfile

```
gem 'mysql2' # 连接mysql
gem 'devise', '~> 4.2' # 做用户权限
```

bundle install

### 配置mysql
vi config/database.yml

```
default: &default
  adapter: mysql2
  encoding: utf8
  reconnect: false
  database: ****
  pool: 10
  username: root
  password: ****
  host: ***.***.***.***

development:
  <<: *default

test:
  <<: *default

production:
  <<: *default
  pool: 25
```

### 配置devise

rails g devise:install

### 设置路由配置root路径
vi config/routes.rb

	get 'home/index' # 删掉这一行

	root to: 'home#index'

### 配置提示告警功能
vi app/views/layouts/application.html.erb

```
<p class="notice"><%= notice %></p>
<p class="alert"><%= alert %></p>
```

### 自动产生登录注册页面

rails g devise:views

### 产生home的controller

rails g controller home index

### 产生user
rails g devise user

rails db:migrate

### 增加nocturne页面
rails g scaffold nocturne column1 column2 column3 column4 column5

rails db:migrate

### 增加页面的登录验证
vi app/controllers/nocturnes_controller.rb

	before_action :authenticate_user! # 这个是Devise的提供的方法，验证登录后才能打开这个页面

### 增加登录注册导航
vi app/views/layouts/application.html.erb

```
  <% if current_user %>
      <%= link_to('注销', destroy_user_session_path, :method => :delete) %> |
      <%= link_to('修改密码', edit_registration_path(:user)) %>
    <% else %>
      <%= link_to('注册', new_registration_path(:user)) %> |
      <%= link_to('登录', new_session_path(:user)) %>
  <% end %>
</html>
```

### 配置页面role访问权限

rails g migration add_role_to_users

vi db/migrate/*_add_role_to_users.rb

		add_column :users, :role, :string

rails db:migrate

vi app/models/user.rb

```
def admin?
    self.role == "admin"
end
```

vi app/controllers/application_controller.rb

```
protected

def authenticate_admin
  unless current_user.admin?
    # flash[:alert] = "Not allow!"
    redirect_to root_path
  end
end
```

vi app/controllers/nocturnes_controller.rb

	before_action :authenticate_admin  # 检查是否有权限

> 注册完用户后，去mysql的user表的role列上增加role为admin

### 增加首页到nocturne的连接
vi app/views/home/index.html.erb

	<%= link_to "nocturnes", nocturnes_path %>

## 验证
rails s

http://localhost:3000


**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**
