---
title: "Docker进阶-资源管理Swarm+Portainer"
date: 2018-09-07 12:13:29
draft: false
categories: ["Docker"]
---

## Docker Swarm资源管理

Docker Swarm是Docker官方三剑客项目之一，提供Docker容器集群服务，是Docker官方对容器云生态进行支持的核心方案。

使用它，用户可以将多个Docker主机封装为单个大型的虚拟Docker主机，快速打造一套容器云平台。

注意：Docker1.12.0之后版本，Swarm模块已经内嵌入Docker引擎，成为Docker子命令docker swarm,绝大多用户已经开始使用Swarm模块，Docker引擎API已经删除Docker Swarm。

### 基本概念

Swarm是使用SwarmKit构建的Docker引擎内置(原生)的集群管理和编排工具。使用Swarm集群之前需要了解以下几个概念。

#### 节点

运行Docker的主机可以主动初始化一个Swarm集群或者加入一个已存在的Swarm集群，这样运行Docker的主机就成为一个Swarm集群的节点(node)。

节点分为管理(manager)节点和工作(worker)节点。

* 管理节点用于Swarm集群的管理，docker swarm集合基本只能在管理节点执行。

* 工作节点是任务执行节点，管理节点将服务(service)下发至工作节点执行。

集群中管理节点与工作节点的关系

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-resource-manage-01.png)

#### 服务和任务

任务(Task)是Swarm中的最小的调度单位，目前来说就是一个单一的容器。

服务(Services)是指一组任务的集合，服务定义了任务的属性。

服务有两种模式：

* replicated services 按照一定规则在各个工作节点上运行指定个数的任务。 

* global services每个工作节点运行一个任务

两个模式通过docker service create的--mode参数指定

容器、任务、服务的关系

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-resource-manage-02.png)

### 创建Swarm集群

了解Swarm集群由管理节点和工作节点组成后，我们创建一个包含一个管理节点和两个工作节点的最小Swarm集群。

#### 初始化集群

使用docker swarm init在本地初始化一个Swarm集群。

``` bash
docker swarm init --advertise-addr 192.168.1.1
```

如果你的Docker主机有多个网段，拥有多个IP，必须使用--advertise-addr指定IP。执行docker swarm init命令的节点自动成为管理节点。

注意：使用docker swarm init

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-resource-manage-03.png)

#### 增加工作节点

在另外两台服务器上执行上一步创建管理节点时候的输出的加入swarm集群的全集

``` bash
docker swarm join \
--token SWMTKN-1-3pu6hszjas19xyp7ghgosyx9k8atbfcr8p2is99znpy26u2lkl-1awxwuwd3z9j1z3puu7rcgdbx \ 
192.168.1.1:2377
```

#### 查看集群

在管理节点使用docker node ls查看集群。

```bash
docker node ls
```

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-resource-manage-04.png)

### 部署服务

使用docker service命令来管理Swarm集群中的服务，该命令只能在管理节点运行。

#### 新建服务

在创建好的Swarm集群中运行nginx服务

``` bash
docker service create --replicas 3 -p 80:80 --name nginx nginx:latest
```

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-resource-manage-05.png)

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-resource-manage-06.png)

现在我们使用浏览器，输入任意节点IP，即可看到nginx默认页面。

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-resource-manage-07.png)

#### 查看服务

查看当前Swarm集群运行的服务

``` bash
docker service ls
```

查看某个服务的详情

``` bash
docker service ps nginx
```

查看某个服务的日志

``` bash
docker service logs nginx
```

#### 删除服务

从Swarm集群中移除某个服务

``` bash
docker service rm nginx
```

### 资源管理

前面利用Docker Swarm快速搭建一个最小集群，也可以在集群上部署服务，但是会发现swarm中并没有提供统一入口查看节点的资源使用情况。这时我们可以用图形化管理工具Portainer帮我们管理swarm集群。

Portainer是Docker的图形化管理工具，提供状态显示面板、应用模板快速部署、容器镜像网络数据卷的基本操作(包括上传下载镜像、创建容器等操作)、事件日志显示、容器控制台操作、Swarm集群和服务等集中管理和操作、登陆用户管理和控制等功能。功能十分全面，基本能满足小型单位对容器管理的全部需求。

#### Portainer集群运行

下载Portainer镜像

``` bash

# 查询当前有哪些Portainer镜像
docker search portainer
docker pull portainer/portainer
```

安装Portainer(管理节点)

``` bash
docker run  -d -p 9000:9000 \
--name portainer --restart=always \
-v /var/run/docker.sock:/var/run/docker.sock \
portainer/portainer
```

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-resource-manage-08.png)

#### Portainer配置

设置管理员帐号密码

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-resource-manage-09.png)

Portainer界面内容

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-resource-manage-10.png)

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-resource-manage-11.png)

*关注【小码农薛尧】，了解更多信息*