---
title: "EasyUI-表单combobox下拉框回显"
date: 2017-12-01 12:11:57
draft: false
categories: "Bug"
---

EasyUI是基于Jquery开发的前端框架，适用于各个网站的后台页面，它提供多种丰富的功能，其中表单就有许多的功能。

#### 问题
表单的修改功能，可以用EasyUI中的内置方法load来回显数据，但是，下拉框combobox却不能正确回显数据，通过百度搜索了许多相似的问题，网上给出的答应都是好多年前的，不适用于现在的版本。

#### 解决方法
问了我的良师益友，他告诉我，EasyUI就为了combobox提供了setValue方法，来解决下拉框的回显问题
``` js
/**
#cc 下拉框的name值 
setValue 方法名
101 下拉框选择的value值
*/$("#cc").combobox("setValue","101");
```

