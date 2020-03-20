---
layout: post
title:  "docker 常用命令"
categories: blog
tags: ['docker']
published: true
comments: true
script: [post.js]
---

* Do not remove this line (it will not be displayed)
{: toc}

## docker

### 创建镜像

```
docker build -t image_name .

```

### 发布镜像

```
docker publish image_name
```

## docker swarm

### 创建服务

```
docker service create --name appname --replicas 1 --limit-cpu 8 --limit-memory 4294967296 --with-registry-auth --network snet -p 8080:8080 image_name
```

### 更新服务

```
docker service update --with-registry-auth --image image_name
```


**参数说明:**

* `--replicas` 副本
* `--limit-cpu` cpu限制
* `--limit-memory` 内存限制
* `--network` 网络模式
* `--with-registry-auth` 多节点下载镜像
* `-p` 端口映射

{% endpost #9D9D9D %}

