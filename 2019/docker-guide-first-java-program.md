---
title: "Docker入门-构建第一个Java程序"
date: 2019-08-16 12:29:49
draft: false
categories: ["Docker"]
---
### Docker入门-构建第一个Java程序
#### 定制镜像
准备一个没有第三方依赖的java web项目，可以参考示例maven结构项目：

session-web.war

把该war上传到安装有docker软件的服务器上宿主目录下。在同级目录创建Dockerfile
``` bash
touch Dockerfile
vim Dockerfile
```

按照前面文章所学的Dockerfile定制镜像知识来编写Dockerfile文件内容如下：
``` dockerfile
# 基础镜像使用
tomcat:7.0.88-jre8
FROM tomcat:7.0.88-jre8
# 作者
MAINTAINER simon <xueyao.me@gmail.com>
# 定义环境变量
ENV TOMCAT_BASE /usr/local/tomcat
# 复制war包
COPY ./session-web.war $TOMCAT_BASE/webapps/
```

执行构建：
``` bash
docker bulid -t session-web:latest .
```

如果构建成功，则会显示构建的分层信息及结果。

查看tomcat构建结果

构建成功后使用docker images命令查看本地是否有该镜像

查看是否有该镜像

#### 运行镜像
镜像制作好之后我们就要把它运行起来

docker run --name session-web -d -p 8888:8080 session-web:latest


启动后使用netstat -na|grep 8888 验证端口是否是在监听状态


![查看服务端口有没有启动](https://ueyao.github.io/image-hosting/blog/2019/docker-run-container-port.png)

浏览器中访问http://ip:8888/session-web/user/login

![最终效果图](https://ueyao.github.io/image-hosting/blog/2019/docker-run-java-result.png)

本文中war包在此仓库下https://github.com/flowstone/blog-example-code

