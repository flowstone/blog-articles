---
title: "Docker入门-数据挂载"
date: 2019-08-18 12:31:57
draft: false
categories: ["Docker"]
---
## Docker数据管理

在容器中管理数据主要有两种方式：
* 数据卷(Volumes)
* 挂载主机目录(Bind mounts)

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-data-mount-01.png)

### 数据卷

数据卷是一个可供一个或多个容器使用的特殊目录，它绕过UFS,可以提供很多有用的特性：
* 数据卷可以在容器之间共享和重用
* 对数据卷的修改会立马生效
* 对数据卷的更新，不会影响镜像
* 数据卷默认会一直存在，即使容器被删除

注意： 数据卷的使用，类似于Linux下对目录或文件进行mount，镜像中的被指定为挂载点的目录中的文件会隐藏掉，能显示看的是挂载的数据卷。

Docker中提供了两种挂载方式，-v和-mount

Docker新用户应该选择 --mount参数

经验丰富的Docker使用者对-v或者--volume已经很熟悉了，但是推荐使用-mount参数。

创建一个数据卷
``` bash
docker volume create my-volume
```

查看指定数据卷的信息
``` bash
docker volume inspect my-volume
```
![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-data-mount-02.png)

启动一个挂载数据卷的容器：

在用docker run命令的时候，使用--mount标记来将数据卷挂载到容器里。

创建一个名为session-web的容器，并加载一个数据卷到容器中的/webapp目录。
``` bash
# 方法一
docker run --name session-web -d -p 8888:8080 --mount source=my-volume,target=/webapp  session-web:latest
# 方法二
docker run --name session-web -d -p 8888:8080 -v my-volume:/webapp     session-web:latest
```

删除数据卷
``` bash
docker volume rm my-volume
```

数据卷是被设计用来持久化数据的，它的生命周期独立于容器，Docker不会在容器被删除后自动删除数据卷，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的数据卷。
如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用docker rm -v这个命令。

无主的数据卷可能会占据很多空间，要清理请使用以下命令
``` bash
docker volume prune
```

### 挂载主机目录

使用--mount标记可以指定挂载一个本地主机的目录到容器中去
``` bash
# 方法一
docker run --name session-web -d -p 8888:8080 \
-v my-volume:/webapp \
session-web:latest
# 方法二
docker run --name session-web -d -p 8888:8080 \
--mount type=bind,source=/src/webapp,target=/opt/webapp session-web:latest
```

上面的命令加载主机的/src/webapp目录到容器的/opt/webapp目录。这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。

本地目录的路径必须是绝对路径

以前,使用-v参数时如果本地目录不存在Docker会自动为你创建一个文件夹。

现在,使用--mount参数时如果本地目录不存在，Docker会报错。Docker挂载主机目录的默认权限是读写，用户也可以通过增加readonly指定为只读。

#### 挂载一个本地主机文件作为数据卷

--mount标记也可以从主机挂载单个文件到容器中
``` bash
# 方法一
docker run --rm -it \
--mount type=bind,source=#HOME/.bash_history,target=/root/.bash_history \ 
ubuntu:17.10 bash

# 方法二
docker run --rm -it \
-v $HOME/.bash_history:/root/.bash_history \
ubuntu:17.10 bash
```
