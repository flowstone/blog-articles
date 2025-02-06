---
title: "Docker入门-搭建docker私有仓库"
date: 2019-08-16 12:30:55
draft: false
categories: "Docker"
---


## Docker Hub

目前Docker官方维护了一个公共仓库Docker Hub，其中已经包括了数量超过15000个镜像。大部分需求都可以通过在Docker Hub中直接下载镜像来使用。

### 注册登录

可以在https://hub.docker.com 免费注册一个Docker账号。在命令行执行docker login输入用户名及密码来完成在命令行界面登记Docker Hub。你可以通过docker logout退出登录。

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-repository-01.png)

### 拉取镜像

可以通过docker search命令来查找官方仓库中的镜像，并利用docker pull命令来将它下载到本地。

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-repository-02.png)

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-repository-03.png)

### 推送镜像

用户也可以在登录后通过docker push命令来将自己的镜像推送到Docker Hub。

修改本地镜像的名字为账号名/镜像名

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-repository-04.png)

上传镜像到公共仓库

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-repository-05.png)

上传过后，查看远程公共仓库

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-repository-06.png)

## 私有仓库

有时候使用Docker Hub这样的公共仓库可能不方便，用户可以创建一个本地仓库供私人使用。比如，基于公司内部项目构建的镜像。

docker-registry是官方提供的工具，可以用于构建私有的镜像仓库。

安装运行docker-registry

可以通过获取官方registry镜像来运行。默认情况下，仓库会被创建在容器的/var/lib/registry目录下。可以通过-v参数来将镜像文件存放在本地的指定路径。

``` bash
docker run --name registry -d  -p 5000:5000 --restart=always  -v /opt/data/registry:/var/lib/registry registry
```

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-repository-07.png)

在私有仓库上传、搜索、下载镜像

创建好私有仓库之后，就可以使用docker tag来标记一个镜像，然后推送它到仓库。先在本机查看已有的镜像。

``` bash
docker image ls
```

使用docker tag将session-web:latest这个镜像标记为127.0.0.1:5000/session-web:latest格式为docker tag IMAGE[:TAG][REGISTRY_HOST[:REGISTRY_PORT]/]REPOSITORY[:TAG]

``` bash
docker tag session-web:latest 127.0.0.1:5000/session-web:latest
```

使用docker push上传标记的镜像

``` bash
docker push 127.0.0.1:5000/session-web:latest
```

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-repository-08.png)

用curl查看仓库中的镜像

``` bash
curl 127.0.0.1:5000/v2/_catlog
```

如果可以看到{"repositories":["session-web"]},表明镜像已经被成功上传了。

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-repository-09.png)

先删除已有镜像，再尝试从私有仓库中下载这个镜像。

``` bash
docker image rm 127.0.0.1:5000/session-web:latest
docker pull 127.0.0.1:5000/session-web:latest
```

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-repository-10.png)

### 注意事项

如果不想使用127.0.0.1:5000作为仓库地址，比如想让本网段的其他主机也能把镜像推送到私有仓库。你就得把例如192.168.1.1:5000这样的内网地址作为私有仓库地址，这时你会发现无法成功推送镜像。

#### 可以用下面方式解决

对于使用systemd的系统，请在/etc/docker/daemon.json中写入如下内容(如果文件不存在请新建该文件)

``` json
{
​    "registry-mirror":[
​        "http://hub-mirror.c.163.com"
​    ],

​    "insecure-registries":[
​        "192.168.1.1:5000"
​    ]
}
```

