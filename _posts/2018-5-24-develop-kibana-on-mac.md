---
layout: post
title: 在mac上搭建kibana开发环境并打包
categories: [kibana]
description: 在mac上搭建kibana的开发环境，开发完成后，打包kibana。
---

## 安装node环境

因为kibana是用nodejs开发的，所以当然首要就是安装node环境。

在mac终端运行`brew install node`,安装完成后运行`node -v`，出现node版本说明安装完成。

## 安装yarn

kibana项目的开发管理工具为yarn，所以需要安装yarn。

在终端运行`brew install yarn --without-node`，因为我们使用上面安装的node所以并不需要yarn的node。

安装完成运行`yarn -v`，出现yarn版本说明yarn安装完成。

## 安装kibana

clone kibana项目并切换到你需要的分支

`git clone https://github.com/elastic/kibana`

`git checkout -b 6.3 origin/6.3`

安装kibana所有依赖

`yarn kbn bootstrap`

运行kibana，增加oss参数启动无x-pack版本

`yarn start --oss`

## 打包kibana

`yarn build --skip-node-download`

```
options:
  --oss                   Only produce the OSS distributable of Kibana
  --no-oss                Only produce the default distributable of Kibana
  --skip-archives         Don't produce tar/zip archives
  --skip-os-packages      Don't produce rpm/deb packages
  --rpm                   Only build the rpm package
  --deb                   Only build the deb package
  --release               Produce a release-ready distributable
  --skip-node-download    Reuse existing downloads of node.js
  --verbose,-v            Turn on verbose logging
  --debug,-d              Turn on debug logging
```

打包过程中，如果在国内会遇到错误，打包失败，具体是在`Running script [build] in [x-pack]:`遇到如下错误：

```
[16:08:30] 'prepare' errored after 1.05 min
[16:08:30] Error: connect ETIMEDOUT 52.216.16.131:443
    at Object._errnoException (util.js:992:11)
    at _exceptionWithHostPort (util.js:1014:20)
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1186:14)
error Command failed with exit code 1.
```

或者

`Error: tunneling socket could not be established, statusCode=503`

这是因为有些包需要翻墙下载。。。而且传统的ssr并不能使终端也翻墙，所以需要设置终端也翻墙。

**不论我是否打包x-pack版本，都会遇到，原因未知。**
