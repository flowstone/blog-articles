---
title: "Bug集锦-Spring Cloud Feign调用其它接口报错"
date: 2019-10-27 12:44:13
draft: false
categories: "Bug"
---


## 问题描述

Spring Cloud Feign调用其它服务报错，错误提示如下:Failed to instantiate [java.util.List]: Specified class is an interface。

## 解决方案

通过查询一些资料，得到的结论，是定义接口传递的参数时，没有用@RequestBody修饰，查看定义接口有用@RequestBogy修饰，Feign的接口实现里没有用@RequestBody修饰，添加后问题就解决了，以后还是要仔细看待每个问题。

![](https://ueyao.github.io/image-hosting/blog/2019/10/WX20191027-115928.png)



![](https://ueyao.github.io/image-hosting/blog/2019/10/WX20191027-120945.png)

