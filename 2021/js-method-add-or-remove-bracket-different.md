---
title: "JS中绑定方法加括号和不加括号的区别"
date: 2021-01-03 13:03:55
draft: false
categories: ["JS"]
---

### 问题描述
在React开发中，遇到一个页面不停调用接口问题，如下图所示
![](https://ueyao.github.io/image-hosting/blog/2021/1/20210120130954126_1090784815.png)

### 问题解决
查找哪里调用了接口，最后在刷新按钮上找到了问题所在，如下所示
``` js
<Button
    type="text"
    icon={<ReloadOutlined />}
    title="刷新"
    onClick={loadData()}
  />
```
按钮的点击事件上绑定了一个方法，`loadData`这是一个普通的查询接口，问题就是loadData()后面加了一对括号，如果我把括号去掉，问题就可以完美解决了，如下所示
``` js
<Button
    type="text"
    icon={<ReloadOutlined />}
    title="刷新"
    onClick={loadData}
  />
```
### 问题思考
为什么会出现这个问题？

Google了一会，得到了我想要的答案
* onClick中绑定方法加括号：相当于直接把函数的返回值给onClick方法，会直接触发点击事件，不需要用户点击
* onClick中绑定方法不加括号：相当于把整个函数赋值给onClick方法，不会触发点击事件，需要用户点击