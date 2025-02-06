---
title: "React中useState如何给对象中的某一个属性赋值"
date: 2021-01-02 13:03:06
draft: false
categories: "JS"
---

### 问题描述

在React文件中使用了useState，定义了查询条件，这个查询条件实际上是一个对象，如下所示
``` js
const [requestParam, setRequestParam] = useState({
    pageNum: 1,
    pageSize: 2,
    startTime: "",
    endTime: "",
    mobile: "",
    sort: "",
    sortField: ""
     
});
```
这些参数是查询接口的请求参数，在一般情况下，我们只需要更新其中一个属性的值，在使用setRequestParam修改这些值时，会提示需要给所有参数赋值，如何才能更新某一个值呢？

### 问题解决
通过Google查询了一下，终于在stackoverflow网站上，看到了大佬们的解答，问题是两年前的问题，我从中找一个容易理解答案，如下所示

``` js
setRequestParam(param => ({
    ...param,
    //如果我要改变mobile的值可以这样写
    mobile: '13100008888'
}));
```
看到上面的答案，应该知道采用es中的解构赋值来更新某个值