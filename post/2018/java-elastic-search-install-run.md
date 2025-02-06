---
title: "ElasticSearch从入门到放弃(一)-安装&运行"
date: 2018-10-18 12:19:00
draft: false
categories: "Java"
---
这几天看到新闻，说ElasticSearch上市了，真的没有想到这个非常流行的开源项目竟然上市了，这个公司的程序员都走上了财务自由的道路。
以前用过ElasticSearch全文搜索引擎，但是时间长也忘记了差不多了，以前而且还是用的老版本，不知道新版本如何使用，废话不多说了，开始：

### 本地环境及版本

* Java1.8
* ElasticSearch6.4.2
* kibana6.4.2

### 步骤
1. 下载ElasticSearch
![](https://ueyao.github.io/image-hosting/blog/20191214170137796.png)

2. 选择自己操作系统所对应的版本
  ![](https://ueyao.github.io/image-hosting/blog/20191214170230123.png)
3. 解压后的目录
  ![](https://ueyao.github.io/image-hosting/blog/20191214170312691.png)
4. 进入bin目录下，运行elasticsearch命令
  ![](https://ueyao.github.io/image-hosting/blog/20191214170400358.png)
5. 看到下图，就说明elasticsearch启动成功，如果你的电脑是Linux，会提示不允许使用root运行，请使用elasticsearch用户启动程序
  ![](https://ueyao.github.io/image-hosting/blog/20191214170510673.png)
6. 浏览器查看http://127.0.0.1:9200,出现下图则说明ElasticSearch启动成功
  ![](https://ueyao.github.io/image-hosting/blog/2017/11/2017-11-20_212810.png)
7. 现在ElasticSearch启动好了，再安装一个图形化操作插件Kibana,原本准备安装Sense，结果安装失败，后来知道，6.4.2版本里已经不支持安装Sense，现在Kibana集成了Sense功能，只需要安装Kibana,下载地址如下
  ![](https://ueyao.github.io/image-hosting/blog/20191214170706807.png)
8. 解压Kibana文件到ElasticSearch目录下(个人习惯)
  ![](https://ueyao.github.io/image-hosting/blog/2019121417073882.png)
9. 运行Kibana项目(/bin/kibana)，看到下图，红色警告部分，因为ElasticSearch服务没有启动，所以kibana项目报告，心跳检测有没有ElasticSearch服务
  ![](https://ueyao.github.io/image-hosting/blog/20191214170815994.png)
10. 当启动ElasticSearch服务，kibana程序检测到有活着的elasticSearch服务，立即注册上
  ![](https://ueyao.github.io/image-hosting/blog/20191214170850998.png)
11. 浏览器查看http://127.0.0.1:5601,出现下图则说明Kibana启动成功
   ![](https://ueyao.github.io/image-hosting/blog/20191214171002432.png)

## 总结
ElasticSearch和Kibana安装和运行是非常容易的，但是还有许多细节需要注意，操作系统和ElasticSearch版本都要弄清楚。

