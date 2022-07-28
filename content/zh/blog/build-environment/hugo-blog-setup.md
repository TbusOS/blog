---
linkTitle: ""
author: "Liulf"
categories: ["docs"]
tags: ["blog","markdown","typora"] 
title: "Hugo构建"
date: 2022-07-20
weight: 3
description:
---
## 准备工作

### 安装go
> 下载go https://studygolang.com/dl
> 下载最新的golang版本后安装
### 安装npm
> http://nodejs.cn/download/
> 下载nodejs安装后会有npm命令
### 安装hugo

>https://github.com/gohugoio/hugo/releases/tag/v0.101.0
https://github.com/gohugoio/hugo/releases/download/v0.101.0/hugo_extended_0.101.0_Windows-64bit.zip
> 下载带extended的最新版本并安装
## Clone 博客仓库
> 将博客clone到本地
> 尽量使用ssh地址
> 并再github上配置好公钥
```shell
git clone git@github.com:TbusOS/tbusos.github.io.git
```
## 构建
```shell
npm install 	#下载 主题中的依赖
hugo mod graph 	# 下载hugo需要go依赖
#第一次构建以上两步可能时间比较久，后续不用执行这两个步骤
hugo server 
```
执行可以在本地 [http://127.0.0.1:1313]( http://127.0.0.1:1313) 预览博客

## 试着提交

> 在目录 content/alpha/blog/environment-build 下按照已经有的文档写一篇文档

> 重新执行hugo server 可以完成构建

> 再次访问[http://127.0.0.1:1313]( http://127.0.0.1:1313) 预览自己的文章

``` shell
hugo #执行hugo 可以完成构建，在docs目录下是要提交的文件，content目录是文章的目录需要一并提交
git add . #添加到提交目录
git push **** #push到github
```



---

至此完成一个完整的博客提交流程

## 注意事项
{{< alert color="warning" title="注意事项!" >}}构建的时候发现有时候docs目录没有及时更新，提交之前建议手动删除docs再重新构建{{< /alert >}}


