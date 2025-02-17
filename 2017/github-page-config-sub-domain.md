---
title: "GitHub Pages二级域名的配置"
date: 2017-02-03 11:26:31
draft: false
categories: ["GIT"]
---

## 域名配置指南
首先创建一个新的仓库，并在设置中设置GitHub Pages的Source从master主分支上构建，点击保存，如下图所示:

![](https://ueyao.github.io/image-hosting/blog/2017/sp170202_220301.png)

并在Custom domain 里填写自定义的二级域名，如下图所示:

![](https://ueyao.github.io/image-hosting/blog/2017/sp170202_220346.png)

在你的域名注册商那里设置域名的DNS,类型CNAME,具体配置如下:

![](https://ueyao.github.io/image-hosting/blog/2017/sp170202_220021.png)

把写好的代码同步到创建的仓库中，通过浏览器访问二级域名，效果如下：

![](https://ueyao.github.io/image-hosting/blog/2017/sp170202_220444.png)

注意此二级域名的一级域名必须解析到自己flowstone.github.io下，如下图所示：
![](https://ueyao.github.io/image-hosting/blog/2017/sp170202_220120.png)

