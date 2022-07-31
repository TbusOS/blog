---
linkTitle: ""
author: "Liulf"
categories: ["Blog Environment"]
tags: ["blog","typora"] 
title: "Hugo构建"
date: 2022-07-20
weight: 3
description:
---
## Clone 博客仓库
> 将博客clone到本地
> 尽量使用ssh地址
> 并再github上配置好公钥
```shell
git clone git@github.com:TbusOS/blog.git
```


## 增加一个md文档

> 在目录 content\zh\blog\build-environment 下按照已经有的文档写一篇文档
>
> **目前博客类的文章都放在这个目录**
>
> 如果需要不同分类也可以按照blog这个目录下的文档复制一份进行修改
>
> 文章的Header按照已有的文章格式写下以下字段

``` yaml
linkTitle: "" # 链接地址
author: "Liulf" # 作者
categories: ["docs"]  # 分类
tags: ["blog","markdown","typora"] # tags 分类
title: "Hugo构建" # 标题
date: 2022-07-20 # 创建日期
weight: 3 # (这个是文章的顺序)
description: # 文章描述
```

写完文章后提交到仓库即可

## 提交文档

```shell
git add .
git pus
```

* 提交到github后会博客会自动构建大概需要50秒稍微等待一下
* 访问[Tbus (tbusos.github.io)](https://tbusos.github.io/)刷新一下查看是否更新成功
* 如果没有更新请联系Liulf


---

至此完成一个完整的博客提交流程
