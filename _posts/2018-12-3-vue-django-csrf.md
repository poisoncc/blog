---
layout: post
title: Vue和Django的csrf验证
categories: [vue, django]
description: 解决Vue和Django前后端完全分离后，表单提交遇到的csrf验证的问题
---

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**

> 最近在研究vue+django前后端分离，遇到了csrf验证的事，不想直接关闭验证了事，所以前前后后了解了一下，说一下自己的解决方案，主要是卡在刚接触vue不知道axios默认不发送cookie，需要设置一下。

## 什么是csrf
这里通俗的讲一下，具体解释请看django官网。csrf是Django提供的防止伪装提交请求的中间件，说白了就是防止别人利用Cookie跨域访问，CSRF全称`Cross-site request forgery`（跨站请求伪造），在setting的MIDDLEWARE中`django.middleware.csrf.CsrfViewMiddleware`设置打开或关闭，在template的form表单中加入{`% csrf_token %`}，django会渲染解释成`<input type="hidden", name='csrfmiddlewaretoken' value=服务器随机生成的token>`，随表单提交上去，django在处理post请求前判断该csrfmiddlewaretoken跟cookie中的csrftoken是否一致，不一致的请求被识别成csrf攻击，返回403。

而现在采用vue+django的前后端完全分离的话，就不能使用{`% csrf_token %`}了，需要自己构建验证，大致流程分为，后端生成csrftoken，前端获取cookie中的csrftoken并随表单提交。

## django处理
django后端需要先在cookie中生成csrftoken，通过下面的方法引入csrf的get_token

`from django.middleware.csrf import get_token`

该方法就是生成cookie中的csrftoken，然后在前端提交表单前传递到前端（可以不传递，让前端去cookie中获取）

```
crsftoken = get_token(request)
return JsonResponse({"crsftoken":crsftoken})
```

这一步如果操作不对，会遇到Forbidden (CSRF cookie not set.)错误。

## vue处理
**由于axios默认是不发送cookie的**，所以我们需要在引入axios后进行如下设置：

`axios.defaults.withCredentials = true`

然后前端获取csrftoken并随表单提交(也可以去cookie中获取)

`this.crsftoken = response.data.crsftoken`

```
headers: {
  'Content-Type': 'multipart/form-data',
  "X-CSRFToken": this.crsftoken
}
this.axios.post(url,formdata,config)
```

这一步如果操作不对，会遇到Forbidden (CSRF token missing or incorrect.)错误。

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**
