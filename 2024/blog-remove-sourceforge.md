---
title: "博客快速迁移到Sourceforge平台"
date: 2024-02-24 19:18:42
draft: false
categories: ["日志"]
---

#### 描述
因为个人原因，准备把托管在阿里云虚拟主机上的博客迁移到免费的sourceforget平台上，
在迁移中遇到很多问题，一一记录下来。

#### 操作
##### 1.从虚拟主机中下载博客的相关文件(网站源码&数据库)

##### 2.上传网站源码到FTP中
打开Sourceforget网站获取FTP连接信息，可以通过项目中的Admin->Project Web Hosting->[Documentation](https://sourceforge.net/p/forge/documentation/Project%20Web%20Services/),查找到相关FTP配置,如下图

![FTP配置](https://ueyao.github.io/image-hosting/blog/2024/0224/01.png)

FTP配置中需要注意登录后的文件路径，我已经在图中标出来了

FTP默认路径
![FTP默认路径](https://ueyao.github.io/image-hosting/blog/2024/0224/02.png)

FTP主机空间路径
![FTP最终路径](https://ueyao.github.io/image-hosting/blog/2024/0224/03.png)

##### 3.同步数据库
通过项目中的Admin->Project Web Hosting->MySQL Database，可以查看数据库的相关配置，也可以修改数据库密码，如下图
![数据库配置](https://ueyao.github.io/image-hosting/blog/2024/0224/04.png)

Sourceforge数据库中有很多限制，需要慢慢的把数据导入,还有此数据库不支持远程连接

##### 4.预览网站
Sourceforge提供二级域名，格式是
* 项目名.sourceforge.io
* 项目名.sourceforge.net

两个都可以，访问网站，如下图
![二级域名效果](https://ueyao.github.io/image-hosting/blog/2024/0224/05.png)

##### 5.绑定自定义域名
通过项目中的Admin->Project Web Hosting->Vhost DNS,可以配置绑定的自定义域名，如下图
![配置自定义域名](https://ueyao.github.io/image-hosting/blog/2024/0224/06.png)

我绑定是自己博客的二级域名,然后在域名的DNS解析系统中配置映射，如果按照我们常理来说，应该绑定CNAME记录**flowstone.sourceforge.io**,一开始我却实这样配置了，但是域名却没办法正确显示，提示了一些错误，原来是Sourceforge通过cloudflare进行了一些CDN加速，没办法正确解析到真正的服务器地址。

其实它在帮助文档中写了详细的配置，我没有按照它的说明来配置(IP地址固定)，文档如下：
![配置域名文档](https://ueyao.github.io/image-hosting/blog/2024/0224/07.png)

最终DNS解析配置如下：
![配置域名文档](https://ueyao.github.io/image-hosting/blog/2024/0224/10.png)


如果我们按照它的配置，指向自己的域名，就可以看到域名解析生效了，效果如下
![最终效果图](https://ueyao.github.io/image-hosting/blog/2024/0224/08.png)

值得注意的是当你配置了自定义域名后，Sourceforge提交的二级域名将自己跳转到你的项目目录中，无法查看你的搭建的网站。


#### 注意

在这些操作里，我遇到了另外一个问题，就是我按照上面的操作，打开我的域名，页面竟报出404错误，如下图
![404错误](https://ueyao.github.io/image-hosting/blog/2024/0224/09.png)

就是这个问题，困扰我一天，我一直以为是DNS解析的问题，查询好多资料，也看了官方的一些客户支持问题，从而确定DNS没有配置错误，那是什么问题造成的呢？

说明域名已经找到了服务器，但是指定的目录下并没有我上传的网站源码，真是奇了怪，不科学，我专门去FTP上找了一番，文件也在，最后在一篇博客里发现了问题，原来官方的配置是基于**PHP7**版本，但是我手贱的把PHP版本改成**PHP8**,所以域名解析时并没有找到正确路径，也就是说*PHP8*版本里面有了特殊的配置，我猜想一种是IP的改变，一种是网站源码路径的改变。




