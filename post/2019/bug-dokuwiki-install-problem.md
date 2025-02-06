---
title: "DokuWiki安装问题"
date: 2019-01-17 12:21:20
draft: false
categories: "Bug"
---

工作了几年，虽然在程序员这个道路上才算开始，希望以后能够有所成长，为了把平时遇到的技术问题，记录下来，第一个想到的就是写wiki，博客虽然创建了许久，但是没有坚持写下去，在网上找了许多的wiki程序，dokuwiki是最符合的。 但是，下载下来，安装就有许多问题，最主要的是通过浏览器访问dukuwiki目录时，出现问题，图没有保存，找到一段官网的问题


``` js
DokuWiki Setup Error

The datadir ('pages') at ./data/pages is not found, isn't accessible or writable. You should check your config and permission settings. Or maybe you want to run the installer?
```

原本这个问题，我在官网找到了https://forum.dokuwiki.org/thread/9893,刚开始的时候，看到英文就头疼，随手用Google翻译了一下，没看懂，然后就在网上又重新的搜索许多相似的问题，都没有解决。最后，回过头来，仔细看了一下英文的意思，原来是权限问题，采用它给出的解决方法，完美解决问题。
``` bash
chmod -R 777 data/
chmod -R 777 lib/
chmod -R 777 conf/
```

做开发，英文非常重要。

