---
title: "关于equals()方法两边变量如何放置"
date: 2017-07-07 11:29:08
draft: false
categories: "Java"
---

##### 两个变量放置位置

如果是两个都是变量，可以放在equals任意一边，没有区别

##### 常量、变量放置位置

如果有一个是常量，equals()方法在使用时，建议equals()方法前面放常量。因为equals()是Object类中定义的，任何对象都可以调用equals()方法，但是，如果对象的值是null的话，会引起空指针异常。

如果变量放在前面也就相当调用变量的equals()方法，变量为空时，就会报空指针异常。所以把常量放在equals()方法前面，是非常好的习惯。

再者，如果变量放在equals()方法的括号内，变量为空时，equals()方法和null做比较，不会出现异常。

如下：
``` java
//constant为常量，variable为变量constant.equals(variable);  
//建议这样使用variable.equals(constant);  //不建议使用，当变量为空时，会出现空指针异常
```