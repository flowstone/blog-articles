---
title: "Docker进阶-快速扩容"
date: 2019-08-21 12:35:15
draft: false
categories: ["Docker"]
---

## 1、命令方式

在创建好的Swarm集群中运行nginx服务，并使用--replicas参数指定启动的副本数。

``` bash
docker service create --replicas 3 -p 80:80 --name nginx nginx:latest
```

或者

``` ba
docker service create -p 80:80 --name nginx nginx:latest
docker service scale nginx=3
docker service ls #查看副本情况
```

## 2、portainer方式

可以使用portainer的方式在web界面上创建服务并指定副本数，同时可以随时动态增减副本数。

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-rapid-expansion-01.png)