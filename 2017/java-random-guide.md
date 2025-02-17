---
title: "Java Random入门"
date: 2017-07-02 11:28:02
draft: false
categories: ["Java"]
---

#### 什么是Random？
当用户要产生一个随机数，java本身提供了丰富的Random类。用户可以根据需求创建Random对象，根据Random类下的方法创建特殊的随机数。

如何创建Random?

首先用户要创建一个Random对象
``` java
Random random = new Random();
```

不过在使用对象之前，要导入对应的包
``` java
import java.util.Random;
```

当用户要创建一个整型的随机数，范围在0-100之间
``` java
/*
创建随机数时，如nextInt(m),里面的范围有一定的规律，如果用户要创建一个m-n(包括m,n)的随机数时，
公式：nextInt(n-m+1)+m;如果用户要创建一个m-n(不包括m,n)的随机数时，公式：nextInt(n-m);
*/
int num = random.nextInt(101);
```

用户可以创建各种类型的随机数，只需要使用对应的方法，即可生成。

