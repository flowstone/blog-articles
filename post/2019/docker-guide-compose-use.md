---
title: "Docker入门-docker compose的使用"
date: 2019-08-19 12:33:56
draft: false
categories: "Docker"
---
## Compose简介

Compose项目是Docker官方的开源项目，负责实现对Docker容器集群的快速编排。其代码目前在https://github.com/docker/compose 上开源。

Compose定位是定义和运行多个Docker容器的应用，其前身是开源项目Fig。

通过前面内容的介绍，我们知道使用一个Dockerfile模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某任务的情况。例如要实现一个Web项目,除了Web服务容器本身，往往还需要加上后端的数据库服务容器，甚至还包括负载均衡容器等。

Compose恰好满足了这样的需求。它允许用户通过一个单独的docker-compose.yml模板文件来定义一组相关联的应用容器为一个项目(project)。

Compose中有两个重要的概念：
* 服务(service):一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
* 项目(project):由一组关联的应用容器组成的一个完整业务单元。

Compose的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

Compose项目由Python编写，实现上调用了Docker服务提供的API来对容器进行管理。

## 安装和卸载

Compose支持Linux、macOS、Windows10三大平台。

Compose可以通过Python的包管理工具pip进行安装，也可以直接下载编译好的二进制文件使用，甚至能够直接在Docker容器中运行。

Docker for Mac、Docker for Windows自带docker-compose二进制文件，安装Docker之后可以直接使用。
``` bash
docker-compose --version
```

Linux系统需要单独使用二进制或者pip方式进行安装。

### Linux安装docker-compose
#### 二进制包
在Linux上的安装十分简单，从官方GitHub Release处直接下载编译好的二进制文件即可。例如，在Linux64位系统上直接下载对应的二进制包。
``` bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose  #赋予可执行权限
```
![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-compose-01.png)

#### PIP安装
如果您计算机的架构是ARM(例如，树莓派),建议使用pip安装。
``` bash
sudo pip install -U docker-compose
```

## 使用

场景：最常见的项目是web网站，一般的web网站都会依赖第三方提供的服务(比如：DB和cache),我们拿dubbo-admin进行讲解(dubbo-admin依赖zookeeper)。

### Compose构建dubbo-admin服务

从github上获取dubbo-admin的master分支源码
``` bash
git clone -b master https://github.com/apache/incubator-dubbo-ops.git
```

修改admin中的application配置，把zookeeper地址改为zookeeper://zookeeper:2181

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-compose-02.png)

使用maven进行编译打包
``` bash
mvn clean package -Dmaven.test.skip=true
```
![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-compose-03.png)

在dubbo-admin目录下编写Dockerfile文件，内容为
``` bash
# FROM,表示使用JDK8环境为基础镜像，如果镜像不是本地会从DockerHub进行下载
FROM openjdk:8-jdk-alpine
# 作者
MAINTAINER Simon<xueyao.me@gmail.com>
VOLUME /tmp
# ADD,拷贝文件并且重命名
ADD ./target/dubbo-admin-0.0.1-SNAPSHOT.jar app.jar
# ENTRYPOINT,为了缩短Tomcat启动时间，添加java.security.egd的系统属性指向/dev/urandom作为ENTRYPOINT
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

使用docker build -t dubbo-admin:1.0 .命令进行构建。
![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-compose-04.png)

在项目根目录下编写docker-compose.yml文件，这个是Compose使用的主模板文件。
``` yml
version: '3.4'
services:
  zk_server:
    image: zookeeper:3.4
    ports:
      - 2181:2181
  dubbo-admin:
    image: dubbo-admin:1.0
    links:
      - zk_server:zookeeper
    depends_on:
      - zk_server
    ports:
      - 7001:7001 
```

在docker-compose.yml文件所在目录执行：
``` bash
docker-compose up
```

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-compose-05.png)

在浏览器中访问http://服务器ip:7001 进行验证，用户名密码为:root/root guest/guest

![](https://ueyao.github.io/image-hosting/blog/2019/8/docker-compose-06.png)

## Compose命令说明
### 命令对象与格式
执行docker-compose [COMMAND] --help或者docker-compose help [COMMAND]可以查看具体某个命令的使用格式。

docker-compose命令的基本的使用格式是：
``` bash
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
```
命令选项

* -f,--file FILE指定模板文件，默认为docker-compose.yml,可以多次指定。
* -p,--project-name NAME指定项目名称，默认将使用所在目录名称作为项目名。
* --x-networking使用Docker的可拔插网络后端特性
* --x-network-driver DRIVER指定网络后端的驱动,默认为bridge
* --verbose输出更多调试信息。
* -v,--version打印版本并退出。

build
```
格式为docker-compose build [options] [SERVICE...]。
构建(重新构建)项目中的服务容器。
可以随时在项目目录下运行docker-compose build来重新构建服务。
选项包括：
* --force-rm 删除构建过程中的临时容器。
* --no-cache 构建镜像过程中不使用cache(将加长构建过程)。
* --pull 始终尝试通过pull来获取更新版本的镜像。
```

logs
```
格式为docker-compose logs [options] [SERVICE...]。
查看服务容器的输出。默认情况下,docker-compose将对不同的服务输出使用不同的颜色来区分。可以通过--no-color来关闭颜色。
```

port
```
格式为docker-compose port [options] SERVICE PRIVATE_PORT。
打印某个容器端口所映射的公共端口。
选项：
 --protocol=proto指定端口协议，tcp(默认值)或者udp。
 --index=index如果同一服务存在多个容器，指定命令对象容器的序号(默认为1)。
```

ps
```
格式为docker-compose ps [options] [SERVICE...]
列出项目中目前的所有容器。
选项：
 -q只打印容器的ID信息。
```

pull
```
格式为docker-compose pull [options] [SERVICE...]。
拉取服务依赖的镜像。
选项：
 --ignore-pull-failures忽略拉取镜像过程中的错误。
```

restart
```
格式为docker-compose restart [options] [SERVICE...]
重启项目中的服务。
选项：
 -t,--timeout TIMEOUT指定重启前停止容器的超时(默认为10秒)。
```

rm
```
格式为docker-compose rm [options] [SERVICE...]
删除所有(停止状态的)服务容器。推荐先执行docker-compose stop命令来停止容器。
选项：
 -f,--force强制直接删除，包括非停止状态的容器。一般尽量不要使用该选项。
 -v删除容器所挂载的数据卷。
```

run
```
格式为docker-compose run[options] [-p PORT...][-e KEY=VAL...] SERVICE [COMMAND] [ARGS...]
在指定服务上执行一个命令。例如：
docker-compose run ubuntu ping docker.com
```

scale
```
格式为docker-compose scal [options] [SERVICE=NUM...]
设置指定服务运行的容器个数。例如：
docker-compose scale web=3 db=2
将启动3个容器运行web服务，2个容器运行db服务。
```

up
```
该命令十分强大，它将尝试自动完成包括构建镜像，(重新)创建服务,启动服务，并关联服务相关容器的一系列操作。链接的服务都将会被自动启动，除非已经处于运行状态。
选项：
 -d 在后台运行服务容器。
 --no-color 不使用颜色来区分不同的服务的控制台输出。
 --no-deps 不启动服务所链接的容器。
 --force-recreate强制重新创建容器，不能与--no-recreate同时使用。
 --no-recreate如果容器已经存在了，则不重新创建，不能与--force-recreate同时使用。
 --no-build 不自动构建缺失的服务镜像。
 -t,--timeout TIMEOUT停止容器时候的超时(默认为10秒)。
```

其它命令如下：

|命令|说明|
|---|----|
|version|格式为docker-compose version,打印版本信息|
|config|验证Compose格式是否正确，若正确则显示配置，若格式错误显示错误原因。|
|exec|进入指定的容器|
|images|列出Compose文件中包含的镜像|
|down|停止up命令所启动的容器，并移除网络。|
|help|获得一个命令的帮助|
|kill|通过发送SIGKILL信号来强制停止服务容器|
|pause|格式为docker-compose pause [SERVICE...],暂停一个服务容器。|
|push|推送服务依赖的镜像到Docker镜像仓库|
|start|格式为docker-compose start[SERVICE...],启动已经存在的服务容器。
|stop|停止已经存在的服务容器。|
|top|查看各个服务容器内运行的进程|
|unpause|格式为docker-compose unpause [SERVICE...],恢复处于暂停状态中的服务。|


## Compose模板文件

模板文件是使用Compose的核心，涉及到的指令关键字也比较多，大部分指令跟docker run 相关参数的含义都类似。

默认的模板文件名称为docker-compose.yml,格式为YAML格式。

注意每个服务都必须通过image指令指定镜像或build指令(需要Dockerfile)等来自动构建生成镜像。

如果使用build指令,在Dockerfile中设置的选项(例如:CMD,EXPOSE,VOLUME,ENV等)将会自动被获取，无需在docker-compose.yml中再次设置。下面介绍常用指令的用法。

``` yml
version: '3'
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

build
``` bash
指令Dockerfile所在文件夹的路径(可以是绝对路径,或者相对docker-compose.yml文件的路径)。
Compose将会利用它自动构建这个镜像,然后使用这个镜像。
使用context指令指定Dockerfile所在文件夹的路径
使用dockerfile指令指定Dockerfile文件名
使用arg指令指定构建镜像时的变量
```

command

覆盖容器启动后默认执行的命令。
``` bash
command:echo "hello world"
```

Container_name

指令容器名称。默认将会使用项目名称_服务名称_序号这样的格式。
``` bash
container_name:docker-web-container
```

configs

仅用于Swarm mode, 详细内容后页面swarm mode会进到。

deploy

仅用于Swarm mode,详细内容后面swarm mode会进到。

devices

指定设备映射关系。
``` bash
devices:
    - "/dev/ttyUSB1:/ttyUSB0"
```

depends_on

解决容器的依赖、启动先后的问题。

dns

自定义DNS服务器。可以是一个值，也可以是一个列表。
``` bash
dns: 8.8.8.8
dns:
  - 8.8.8.8
  - 114.114.114.114
```

environment

设置环境变量。你可以使用数组或字典两种格式。
只给定名称的变量会自动获取运行Compose主机上对应变量的值，可以用来防止泄露不必要的数据。
``` bash
environment:
  RACK_ENV: development
  SESSION_SECRET:
environment:
  - RACK_ENV=development
  - SESSION_SECRET  
```

expose

显露端口，但不映射到宿主机，只被连接的服务访问。仅可以指定内部端口为参数

``` bash
expose:
  - "3000"
  - "8000"
```

extra_hosts

类似Docker中的--add-host参数，指定额外的host名称映射信息。
会在启动后的服务容器中/etc/hosts文件中添加一条条目。8.8.8.8 googledns
``` bash
extra_hosts:
  - "googledns:8.8.8.8"
```

healthcheck

通过命令容器是否健康运行。
``` bash
healthcheck:
   test:["CMD","curl","-f","http://localhost"]
   interval:1m30s
   timeout:10s
   retries:3
```

image

指定为镜像名称或镜像ID。如果镜像在本地不存在，Compose将会尝试拉去这个镜像
``` bash
image:session-web:latest
```

lables

为容器添加Docker元数据(metadata)信息。例如可以为容器添加辅助说明信息。
``` bash
labels:
  com.study.department:"devops department"
  com.study.release:"v1.0"
```

links

连接到其他容器。注意：不推荐使用该指令。
应该使用docker network,建立网络,而docker run --network来连接特定网络。或者使用version:'2'和更高版本的docker-compose.yml直接定义自定义网络并使用。

network_mode

设置网络模式。使用和docker run的--network参数一样的值。
``` bash
network_mode:"bridge"
network_mode:"host"
network_mode:"none"
```

networks

配置容器连接的网络。
``` bash
version:"3"
services:
  some-service:
    networks:
      - some-network
networks:
  some-network:      
```

ports

暴露端口信息。使用宿主端口：容器端口(HOST:CONTAINER)格式，或者仅仅指定容器的端口(宿主将会随机选择端口)都可以。
``` bash
ports:
  - "3000"
  - "8000:8000"
```

volumes

数据卷所挂载路径设置，可以设置宿主机路径，同时支持相关路径。
``` bash
volumes:
  - /var/lib/mysql
  - cache/:/tmp/cache
  - ~/configs:/etc/configs/:ro
```

ulimits

指定容器的ulimits限制值。
例如，指定最大进程数为65535,指定文件句柄数为20000(软限制,应用可以随时修改,不能超过硬限制)和40000(系统硬限制，只能root用户提高)。
``` bash
ulimits:
  nproc: 65535
  nofile:
    soft: 20000
    hard: 40000
```

此外，还有包括domainname,entrypoint,hostname,ipc,mac_address,privileged,read_only,shm_size,restart,stdin_open,tty,user,working_dir等指令，基本跟docker run中对应参数的功能相同。

指定服务容器启动后执行的入口文件
``` bash
entrypoint: /code/entrypoint.sh
```

指定容器中运行应用的用户名
``` bash
user:nginx
```

指定容器中工作目录
``` bash
working_dir: /code
```

指定容器中搜索域名、主机名、mac地址等
``` bash
domainname:your_website.com
hostname:test
mac_address:08-00-27-00-0C-0A
```

允许容器中运行一些特权命令
``` bash
privileged:true
```

指定容器退出后的重启策略为始终重启。在生产环境中推荐配置为always或者unless-stopped
``` bash
restart:alwarys
```

以只读模式挂载容器的root文件系统，意味着不能对容器内容进行修改
``` bash
read_only:true
```

打开标准输入，可以接受外部输入
``` bash
stdin_open:true
```

模拟一个伪终端
``` bash
tty:true
```

Compose模板文件支持动态读取主机的系统环境变量和当前目录下的.env文件中的变量。例如，下面的Compose文件将从运行它的环境中读取变量${MONGO_VERSION}的值，并写入执行的指令中。
``` bash
version: "3"
services:
  db:
    image: "mongo:${MONGO_VERSION}"
```

如果执行MONGO_VERSION=3.2 docker-compose up则会启动一个mongo:3.2镜像的容器。若当前目录存在.env文件，执行docker-compose命令时将从该文件中读取变量。



