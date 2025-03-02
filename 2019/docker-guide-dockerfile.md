---
title: "Docker入门-Dockerfile的使用"
date: 2019-08-14 12:26:32
draft: false
categories: ["Docker"]
---

## 使用Dockerfile定制镜像

镜像的定制实际上就是定制每一层所添加的配置、文件。我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，这个脚本就是Dockerfile。

Dockerfile是一个文本文件，其内包含了一条条的指令，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

接下来我们以官方nginx镜像为例，使用Dockerfile来定制。

在一个空白目录中，建立一个文本文件，并命名为Dockerfile:
``` bash
mkdir mynginx
cd mynginx
touch Dockerfile
```

其内容为：
``` bash
FROM nginx
RUN echo '<h1> Hello,Docker!</h1>' >/usr/share/nginx/html/index.html
```
这个Dockerfile很简单，一共就两行。涉及到了两条指令，FROM和RUN。

### FROM指定基础镜像

所谓定制镜像，一定是以一个镜像为基础，在其上进行定制。基础镜像是必须指定的，而FROM就是指定基础镜像，因此一个Dockerfile中FROM是必备的指令，并且必须是第一条指令。在Docker Hub上有非常多的高质量的官方镜像，有可以直接拿来使用的服务类的镜像，如nginx、redis、mysql、tomcat等；可以在其中寻找一个最符合我们最终目标的镜像为基础镜像进行定制。

如果没有找到对应服务的镜像，官方镜像中还提供了一些更为基础的操作系统镜像，如ubuntu、debian、centos、alpine等，这些操作系统的软件库为我们提供了更广阔的扩展空间。

除了选择现有镜像为基础镜像外，Docker还存在一个特殊的镜像，名为scratch。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。

``` bash
FROM scratch
...
```

如果你以scratch为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

对于Linux下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接FROM scratch会让镜像体积更加小巧。使用Go语言开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为Go是特别适应容器微服务架构的语言的原因之一。

### RUN执行命令

RUN指令是用来执行命令行命令的。由于命令行的强大能力，RUN指令在定制镜像时是最常用的指令之一。其格式有两种：

shell格式：RUN <命令>
``` bash
RUN echo '<h1>Hello,Docker~</h1>' > /usr/share/nginx/html/index.html
```

exec格式： RUN ["可执行文件",“参数1”,“参数2”]
``` bash
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```

上面我们利用Dockerfile定制了nginx镜像，现在我们明白了这个Dockerfile的内容，接下来我们来构建这个镜像。

在Dockerfile文件所在目录执行:
``` bash
docker build -t nginx:v3 .
```

从命令的输出结果中，我们可以清晰的看到镜像的构建过程。在Step2中，RUN指令启动了一个容器782d25b7c611,执行了所要示的命令，并最后提交了这一层ba38ff665f57,随后删除了所用到的这个容器782d25b7c611。

![dockerfile构建](https://ueyao.github.io/image-hosting/blog/20191214172011541.png)

启动构建的Nginx
``` bash
docker run --name nginx-test -p 8081:80 -d nginx:v3
```

如图所示

![docker启动nginx容器](https://ueyao.github.io/image-hosting/blog/20191214172033511.png)

## Dockerfile指令详解

#### COPY复制文件

格式：
* COPY <源路径>...<目标路径>
* COPY ["<源路径1>",..."<目标路径>"]

COPY指令将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置。比如：

``` bash
COPY package.json /usr/src/app/
```

<源路径>可以是多个，甚至可以是通配符，如：
``` bash
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

#### ADD更高级的复制文件

ADD指令和COPY的格式和性质基本一致。但是在COPY基础上增加了一些功能。比如<源路径>可以是一个URL,这种情况下，Docker引擎会试图去下载这个链接的文件放到<目标路径>去。

在Docker官方的Dockerfile最佳实践文档中要求，尽可能的使用COPY，因此COPY的语义很明确，就是复制文件而已，而ADD则包含了更复杂的功能，其行为也不一定很清晰。最适合使用ADD的场合，就是所提及的需要自动解压缩的场合。

因此在COPY和ADD指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用COPY指令，仅在需要自动解压缩的场合使用ADD。

#### CMD容器启动命令

CMD指令的格式和RUN相似，也是两种格式：
* shell格式：CMD <命令>
* exec格式：CMD ["可执行文件",“参数1”,“参数2”]
* 参数列表格式：CMD [“参数1”,“参数2”...]。在指定了ENTRYPOINT指令后，用CMD指定具体参数。

Docker不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。CMD指令就是用于指定默认的容器主进程启动命令的。

#### ENTRYPOINT入口点

ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及参数。ENTRYPOINT在运行也可以替代，不过比CMD要略显繁琐，需要通过docker run的参数 --entrypoint来指定。

当指定了ENTRYPOINT后，CMD的含义就发生了改变，不再是直接的运行其命令，而是将CMD的内容作为参数传给ENTRYPOINT指令，换句话说实际执行时，将变为：
``` bash
<ENTRYPOINT>"<CMD>"
```

#### ENV设置环境变量

格式有两种：
* ENV <key><value>
* ENV <key1>=<value1><key2>=<value2>...

这个指令很简单，就是设置环境变量而已，无论是后面的其它指令，如RUN，还是运行时的应用，都可以直接使用这里定义的环境变量。

``` bash
ENV VERSION=1.0 DEBUG=on NAME="Happy Feet"
$VERSION #使用环境变量
```

下列指令可以支持环境变量展开：ADD、COPY、ENV、EXPOSE、LABEL、USER、WORKDIR、VOLUME、STOPSIGNAL、ONBUILD。

#### ARG构建参数

格式：
* ARG <参数名>[=<默认值>]
构建参数和ENV的效果一样，都是设置环境变量。所不同的是，ARG所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用ARG保存密码之类的信息，因此docker history还是可以看到所有值的。

Dockerfile中的ARG指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令docker build中用 --build-arg <参数名>=<值>来覆盖。

#### VOLUME定义匿名卷

格式为：
* VOLUME ["<路径1>","[路径2]"...]
* VOLUME <路径>

容器运行时应该尽量保持容器存储层不发生写操作，对于数据库需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中，为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在Dockerfile中,我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据
``` bash
VOLUME /data
```

这里的/data目录就会在运行时自动挂载为匿名卷，任何向/data中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行时可以覆盖这个挂载设置。

比如：
``` bash
docker run -d -v mydata:/data xxxx
```

在这行命令中，就使用了mydata这个命名卷挂载到了/data这个位置，替代了Dockerfile中定义的匿名卷的挂载配置。

#### EXPOSE声明端口

格式为EXPOSE <端口1>[<端口2>...]。

EXPOSE指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应该就会开启这个端口的服务。

在Dockerfile中写入这样的声明有两个好处：
1. 是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；
2. 在运行是使随机端口映射时，也就是docker run -P时，会自动随机映射EXPOSE端口。

#### WORKDIR指定工作目录

格式为WORKDIR <工作目录路径>

使用WORKDIR指令可以来指定工作目录(或者称为当前目录),以后各层的当前目录就被改为指定的目录，如该目录不存在，WORKDIR会帮你建立目录。

之前提到一些初学者常犯的错误是把Dockerfile等同于Shell脚本来书写，这种错误的理解还可能会导致出现下面这样的错误：

``` bash
RUN cd /app
RUN echo "hello">world.txt
```

如果将这个Dockerfile进行构建镜像运行后，会发现找不到 /app/world.txt文件。

原因

在Shell中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令。

而在Dockerfile中，这两行RUN命令的执行环境根本不同，是两个完全不同的容器。这就是对Dockerfile构建分层存储的概念不了解导致的错误。

每一个RUN都是启动一个容器、执行命令、然后提交存储层文件变量。第一层RUN cd /app的执行仅仅是当前进程的工作目录变量，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。

因此如果需要改变以后各层的工作目录的位置，那么应该使用WORKIDR指令。

#### USER指定当前用户
格式：USER <用户名>

USER指令和WORKDIR相似，都是改变环境状态并影响以后的层。WORKDIR是改变工作目录，USER则是改变之后层的执行RUN，CMD以及ENTRYPOINT这类命令的身份。

当前，和WORKDIR一样，USER只是帮助你切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。
``` bash
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN ["redis-server"]
```

#### HEALTHCHECK健康检查
格式：
* HEALTHCHECK [选项] CMD <命令>:设置检查容器健康状况的命令
* HEALTHCHECK NONE：如果基础镜像有健康检查指令，可以屏蔽掉其健康检查指令

HEALTHCHECK指令是告诉Docker应该如何进行判断容器的状态是否正常，这是Docker1.12引入的新指令。通过该指令指定一行命令，用这行命令来判断容器主进程的服务状态是否还正常，从而比较真实的反应容器实际状态。

一个镜像指令HEALTHCHECK指令后，用其启动容器，初始状态会为starting,在执行健康检查成功后变为healthy,如果连续一定次数失败，则会变为unhealthy。

HEALTHCHECK支持下列选项：
* --interval=<间隔>:两次健康检查的间隔，默认为30秒；
* --timeout=<时长>:健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认30秒；
* --retries=<次数>：当连续失败指定次数后，则将容器状态视为unhealthy,默认3次。

为了帮助排障，健康检查命令的输出(包括stdout以及stderr)都会被存储于健康状态里，可以用docker inspect来查看。

#### ONBUILD为他人做嫁衣裳

格式： ONBUILD <其它指令>
ONBUILD是一个特殊的指令，它后面跟的是其它指令，比如RUN,COPY等，而这些指令，在当前镜像构建时并不会被执行。只有当以前镜像为基础镜像，去构建下一级镜像的时候才会被执行。

Dockerfile中的其它指令都是为了定制当前镜像而准备的，唯有ONBUILD是为了帮助别人定制自己而准备的。


### 其他制作镜像方式

#### docker save和docker load
Docker还提供了docker load和docker save命令，用以将镜像保存为一个tar文件，然后传输到另一个位置上，再加载进来。这是在没有Docker Registry时的做法，现在已经不推荐，镜像迁移应该直接使用Docker Registry，无论是直接使用Docker Hub还是使用内网私有Registry都可以。

例如：保存nginx镜像
``` bash
docker save nginx|gzip > nginx-latest.tar.gz
```

然后我们将nginx-latest.tar.gz文件复制到了另一个机器上，再次加载镜像：
``` bash
docker load -i nginx-latest.tar.gz
```