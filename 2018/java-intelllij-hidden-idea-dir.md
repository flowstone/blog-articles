---
title: "IntelliJ IDEA技巧一之隐藏.idea目录"
date: 2018-10-11 12:17:11
draft: false
categories: ["Java"]
---
## 问题场景
通过IntelliJ IDEA软件创建Java Web项目时，项目目录中总会生成.idea配置目录并在软件界面里显示，影响项目美感，如何在软件界面中隐藏.idea目录呢？

## 解决方法
* 打开IDEA软件的设置(Mac是Preferences,Windows是Settings)
* 点击Editor->File Types
* Ignore file and folders中添加.idea;(分号为间隔，以后隐藏文件都在这个地方配置)
