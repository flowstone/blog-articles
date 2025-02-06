---
title: "IntelliJ IDEA意外断电，项目所有文件报错"
date: 2018-09-14 12:14:34
draft: false
categories: "Bug"
---

今天同事的台式机突然重新启动了，打开IntelliJ IDEA时，项目中所有文件报错，但是项目可以运行，大概是软件问题，上网找了好多解决方法，都没有解决这个问题，后来，我和他说了，把软件的配置文件删除，他没有删除，怕软件坏了，我只能笑笑不说话。后来，他在网上找到解决方案，我把这个问题记录下来。

1.首先打开IntelliJ IDEA的配置文件，在当前电脑用户文件夹下的.IntelliJIdea2016.3(具体看各自的版本)
![](https://ueyao.github.io/image-hosting/blog/2018/09/2018-09-14-01.png)

2.删除system目录下的caches文件夹，该目录是软件运行时生成的缓存目录，删除后，问题完美解决。其实和我删除整个软件配置文件差不多。
![](https://ueyao.github.io/image-hosting/blog/2018/09/2018-09-14-02.png)