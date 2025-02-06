---
title: "Docker入门-介绍和安装"
date: 2019-08-14 12:27:50
draft: false
categories: "Docker"
---

## Docker容器

### Docker是什么

Docker最初是dotCloud公司创建人Solomon Hykes在法国期间发起的一个公司内部项目，它是基于dotCloud公司多年云服务技术的一次革新，并于2013年3月以Apache2.0授权协议开源，主要项目代码在Github上进行维护。Docker项目后来加入了Linux基金会，并成立推动开放容器联盟(OCI).

Docker使用Google公司推出的Go语言进行开发实现，基于Linux内核的cgroup,namespace,以及AUFS类的Union FS等技术，对进程进行封装隔离，属于操作系统层面的虚拟化技术。由于隔离的进程独立于宿主和其它的隔离的进程，因此也称为容器。

Docker在容器的基础上，进行了进一步的封装，从文件系统、网络互联至进程隔离等待，极大的简化了容器的创建和维护。使得Docker技术比虚拟机技术更为轻便、快捷。

### Docker和传统虚拟机

![docker和传统虚拟机比较](https://ueyao.github.io/image-hosting/blog/2019/docker-vm.png)

传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；

而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

### 为什么要使用Docker

#### Docker优势
* 更高效的利用系统资源
* 更快速的启动时间
* 一致的运行环境
* 持续交付和部署
* 更轻松的迁移
* 更轻松的维护和扩展

#### 对比传统虚拟机总结

|特性|容器|虚拟机|
|--|-----|----|
|启动|秒级|分钟级|
|硬盘使用|一般为MB|一般为GB|
|性能|接近原生|较弱|
|系统支持量|单机支持上千个容器|一般几十个|

### Docker架构

![](https://ueyao.github.io/image-hosting/blog/2019/docker-framework.png)

Docker使用客户端-服务器(C/S)架构模式，使用远程API来管理和创建Docker容器。

### Docker基本概念

#### Docker镜像

我们都知道，操作系统分为内核和用户空间。对于Linux而言，内核启动后，会挂载root文件系统为其提供用户空间支持。而Docker镜像(Image),就相当于是一个root文件系统。比如官方镜像centos:7.6就包含了完整的一套centos7.6最小系统的root文件系统。

Docker镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数(如匿名半卷、环境变量、用户等)。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

#### Docker镜像分层存储

因为镜像包含操作系统完整的root文件系统，其体积往往是庞大的，因此在Docker设计时将其设计为分层存储的架构。镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。

镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。

#### Docker容器

镜像(Image)和容器(Container)的关系，就像Java中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

前面讲过镜像使用的是分层存储，容器也是如此。每一个容器运行时，是以镜像基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为容器存储层。

容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。

按照Docker最佳实践的要求，容器不应该向其存储内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用Volume数据卷、或者绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主(或网络存储)发生读写，其性能和稳定性更高。

数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。

#### Docker仓库

镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，Docker Registry就是这样的服务。

一个Docker Registry中可以包含多个仓库(Repository)；每个仓库可以包含多个标签(Tag)；每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应软件的各个版本。我们可以通过<仓库名>:<标签>的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以latest作为默认标签。

以centos镜像为例，centos是仓库的名字，其内包含有不同的版本标签，如，6.9，7.5。我们可以通过centos:6.9，或者centos:7.5来具体指定所需哪个版本的镜像。如果忽略了标签，比如centos,那将视为centos:latest。

仓库名经常以两段式路径形式出现，比如study/nginx，前者往往意味着Docker Registry多用户环境下的用户名，后者则往往是对应的软件名。但这并非绝对，取决于所使用的具体Docker Registry的软件或服务。

##### Docker Registry公开仓库

常用的Registry是官方的Docker Hub，这也是默认的Registry。除此以外，还有CoreOS的Quay.io，CoreOS相关的镜像存储在这里；Google的Google Container Registry,Kubernetes的镜像使用的就是这个服务。

国内的一些云服务商提供了针对Docker Hub的镜像服务。这些镜像服务被称为加速器。常见的有阿里加速器、DaoCloud加速器等。使用加速器会直接从国内的地址下载Docker Hub的镜像，比直接从Docker Hub下载速度会提高很多。

国内也有一些云服务商推荐类型于Docker Hub的公开服务。如网易云镜像服务、
DaoCloud镜像市场、阿里云镜像库等。

## 安装

### Docker版本命名

Docker在1.13版本之后，从2017年的3月1日开始，版本命名规则变为如

|项目|说明|
|---|----|
|版本格式|YY.MM|
|Stable版本|每个季度发行|
|Edge版本|每个月发行|
|当前Docker CE Stable版本|18.09|
|当前Docker CE Edge版本|18.09|

同时Docker划分为CE和EE。CE即社区版(免费，支持周期三个月)强调安全，付费使用。

### CentOS安装Docker

1、系统要求

Docker CE支持64位版本CentOS7,并且要求内核版本不低于3.10。
``` bash
# 查看当前系统内核
uname -r
```

2、卸载旧版本

旧版本的Docker称为docker或者docker-engine,使用以下命令卸载旧版本：
``` bash
sudo yum remove docker docker-common docker-selinux docker-engine
```

3、使用yum安装

``` bash
sudo yum install docker-ce
```

4、使用脚本安装

在测试或开发环境中Docker官方为了简化安装流程,提供了一套便捷的安装脚本，系统上可以使用这套脚本安装：

``` bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```

执行这个命令后，脚本就会自动的将一切做准备工作做好，并且把Docker CE的Edge装在系统中。

5、启动Docker CE

``` bash
sudo systemctl enable docker #设置开启启动
sudo systemctl start docker
```

6、建立docker用户组

默认情况下，docker命令会使用Unix socket与Docker引擎通讯。而只有root用户和docker组的用户才可以访问Docker引擎的Unix socket。一般Linux系统上不会直接使用root用户进行操作。因此，需要将使用docker的用户加入docker用户组。

``` bash
sudo groupadd docker #建立docker组
sudo usermod -aG docker #USER #将当前用户加入docker组
```

7、测试Docker是否安装正确

``` bash
docker run hello-world #启动一个基于hello-world镜像的容器
```

若能正常输出以上信息，则说明安装成功。

### CentOS卸载Docker

1、 删除docker安装包

``` bash
sudo yum remove docker-ce
```

2、删除docker镜像

``` bash
sudo rm -rf /var/lib/docker
```

### 镜像加速器

国内从Docker Hub拉取镜像有时会遇到困难，此时可以配置镜像加速器。Docker官方和国内很多云服务商都提供了国内加速器服务，例如：
* Docker官方提供的中国registry mirror
* 阿里云加速器
* DaoCloud加速器
* 163加速器

接下来我们以163加速器为例进行介绍。

#### CentOS7配置镜像加速

对于使用systemd的系统，请在/etc/docker/daemon.json中写入如下内容(如果文件不存在请新建该文件)

``` json
{
    "registry-mirrors":[
        "http://hub-mirror.c.163.com"
    ]
}
```

重新启动服务生效

``` bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

查看当前docker信息

``` bash
docker info
```

![docker信息](https://ueyao.github.io/image-hosting/blog/2019/docker-163.png)

