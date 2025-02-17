---
title: "Docker入门-常用命令"
date: 2019-08-14 12:29:00
draft: false
categories: ["Docker"]
---

## Docker镜像操作

Docker运行容器前需要本地存在对应的镜像，如果本地不存在该镜像，Docker会从镜像仓库下载该镜像。

### 获取镜像

从Docker镜像仓库获取镜像的命令是docker pull。其命令格式为：

``` bash
docker pull [选项][Docker Registry地址[:端口号]/]仓库名[:标签]
```

具体的选项可以通过docker pull --help命令看到，这里我们说一下镜像名称的格式。Docker镜像仓库地址：地址的格式一般是<域名/IP>[:端口号]。默认地址是Docker Hub。仓库名：如之前所说，这里的仓库名是两段式名称，即<用户名>/<软件名>。对于Docker Hub,如果不给出用户名，则默认为library,也就是官方镜像。

``` bash
docker pull ubuntu:16.04
```

上面的命令中没有给出Docker镜像仓库地址，因此将会从Docker Hub获取镜像。而镜像名称是ubuntu:16.04,因此将会获取官方镜像library/ubuntu仓库中标签为16.04的镜像。

### 运行镜像

有了镜像后，我们就能够以这个镜像为基础启动并运行一个容器。以上面的ubuntu:16.04为例，如果我们打算启动里面的bash并且进行交互式操作的话，可以执行下面的命令。

``` bash
docker run -it --rm ubuntu:16.04 bash
```

-it:这是两个参数，一个是-i:交互式操作，一个是-t终端。

--rm:这个参数是说容器退出后随之将其删除

ubuntu:16.04:这是指用ubuntu:16.04镜像为基础来启动容器。

bash:放在镜像名后的是命令，这里我们希望有个交互式shell,因此用的是bash。

最后我们通过exit退出了这个容器。

### 列出镜像

要想列出已经下载下来的镜像，可以使用docker image ls命令。列表包含了仓库名、标签、镜像ID、创建时间以及所占用的空间。

``` bash
docker image ls
```

查看镜像、容器、数据卷所占用的空间。

``` bash
docker system df
```

仓库名、标签均为<none>的镜像称为虚悬镜像(dangling image)，显示这类镜像

``` bash
docker image ls -f dangling=true
```

一般来说，虚悬镜像已经失去了存在的价值，是可以随意删除的，可以用下面的命令删除

``` bash
docker image prune
```

### 删除本地镜像

如果要删除本地的镜像，可以使用docker image rm命令，其格式为：

``` bash
docker image rm [选项] <镜像1>[<镜像2>...]
```

其中，<镜像>可以是镜像短ID、镜像长ID、镜像名或者镜像摘要。

使用docker image ls -q来配置docker image rm，这样可以批量删除希望删除的镜像。

``` bash
docker image rm $(docker image ls -q ubuntu) #删除所有仓库名为redis的镜像
```

或者删除所有在ubuntu:16.04之前的镜像：

``` bash
docker image rm $(docker image ls -q -f before=ubuntu:16.04)
```

## Docker容器操作

容器是独立运行的一个或一组应该，以及它们运行态环境。对应的，虚拟机可以理解为模拟运行的一整套操作系统(提供了运行态环境和其他系统环境)和跑在上面的应用。

### 启动容器

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态(stopped)的容器重新启动。

因为Docker的容器实是轻量级的，用户可以随时删除和新创建容器。

#### 新建并启动

``` bash
docker run
```

输出一个“Hello World”，之后终止容器。

``` bash
docker run ubuntu:16.04 /bin/echo "Hello world"
```

#### 启动已终止容器

``` bash
docker container start 或者 docker start
```

启动一个bash终端，允许用户进行交互。

``` bash
docker run -t -i ubuntu:16.04 /bin/bash
```

-t 让Docker分配一个伪终端并绑定到容器的标准输入上，-i则让容器的标准输入保持打开。当利用docker run来创建容器时，Docker在后台运行的标准操作包括：

* 检查本地是否存在指定的镜像，不存在就从公有仓库下载
* 利用镜像创建并启动一个容器
* 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
* 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
* 从地址池配置一个ip地址给容器
* 执行用户指定的应用程序
* 执行完毕后容器被终止

### 后台运行

很多时间，需要让Docker在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加-d参数来实现。

如果不使用-d参数运行容器，比如docker run hello-world会把日志打印在控制台。
如果使用-d参数运行容器，比如docker run -d hello-world不会输出日志，只会打印容器id(输出结果可以用docker logs查看)；

注：容器是否会长久运行，是和docker run指定的命令有关，和-d参数无关。

### 停止运行的容器

可以使用docker container stop来终止一个运行中的容器。终止状态的容器可以用docker container ls -a 命令看到。处于终止状态的容器，可以通过docker container start命令来重新启动。此处，docker container restart命令会将一个运行态的容器终止，处于再重新启动它。

### 进入容器

在使用-d参数时，容器启动后进入后台，某些时候需要进入容器进行操作，使用docker exec命令可以进入到运行中。

exec命令 -i -t参数

docker exec后边可以跟多个参数，这是主要说明 -i -t参数。
只用-i参数时，由于没有分配伪终端，界面没有我们熟悉的Linux命令提示符，但命令执行结果仍然可以返回。当-i -t参数一起使用时，则可以看到我们熟悉的Linux命令提示符。

``` bash
docker exec -it 容器id /bin/bash
```

### 导出和导入容器

#### 导出容器

如果要导出本地某个容器，可以使用docker export命令。

``` bash
docker export 容器ID>导出文件名.tar
```

#### 导入容器

可以使用docker import从容器快照文件中再导入为镜像

``` bash
cat 导出文件名.tar|docker import - 镜像用户/镜像名：镜像版本
```

此外，也可以通过指定URL或者某个目录来导入

``` bash
docker import http://study.163.com/image.tgz example/imagerepo
```

### 删除容器

#### 删除容器

可以使用docker container rm来删除一个处于终止状态的容器

``` bash
docker container rm ubuntu:16:04
```

如果要删除一个运行中的容器，可以添加-f参数。Docker会发送SIGKILL信号给容器。

#### 清楚所有处于终止状态的容器

用docker container ls -a 命令可以查看所有已经创建的包括终止状态的容器，如果数量太多要一个个删除可以会很麻烦，用下面的命令可以清理掉所有处于终止状态的容器。

```
docker container prune
```