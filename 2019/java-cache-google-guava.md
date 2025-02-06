---
title: "Java内存缓存-通过Google Guava创建缓存"
date: 2019-08-23 12:37:51
draft: false
categories: "Java"
---

## 谷歌Guava缓存

### Guava介绍

Guava是Google guava中的一个内存缓存模块，用于将数据缓存到JVM内存中。实际项目开发中经常将一些公共或者常用的数据缓存起来方便快速访问。

![](https://ueyao.github.io/image-hosting/blog/2019/8/cache-guava-01.png)

Guava Cache是单个应用运行时的本地缓存。它不把数据存放到文件或外部服务器。如果不符合需求，可以选择Memcached、Redis等工具。

### 小案例

> pom.xml添加guava依赖

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>me.xueyao.cache</groupId>
    <artifactId>java-demo</artifactId>
    <version>1.0.0</version>

    <dependencies>
        <dependency>
            <groupId>javax.cache</groupId>
            <artifactId>cache-api</artifactId>
            <version>1.1.0</version>
        </dependency>

        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>27.0.1-jre</version>
        </dependency>
    </dependencies>
</project>
```

> GuavaCacheDemo.java 代码如下：

``` java
package me.xueyao.cache.java.guava;

import com.google.common.cache.*;
import me.xueyao.cache.java.pojo.User;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

/**
 * @author simon
 * https://github.com/google/guava
 */
public class GuavaCacheDemo {
    public static void main(String[] args) throws ExecutionException {
        //缓存接口这里是LoadingCache，LoadingCache在缓存项不存在时可以自动加载缓存
        LoadingCache<String, User> userCache
                //CacheBuilder的构造函数是私有的，只能通过其静态方法newBuilder()来获得CacheBuilder的实例
                = CacheBuilder.newBuilder()
                //设置并发级别为8，并发级别是指可以同时写缓存的线程数
                .concurrencyLevel(8)
                //设置写缓存后8秒钟过期
                .expireAfterWrite(8, TimeUnit.SECONDS)
                //设置写缓存后1秒钟刷新
                .refreshAfterWrite(1, TimeUnit.SECONDS)
                //设置缓存容器的初始容量为5
                .initialCapacity(5)
                //设置缓存最大容量为100，超过100之后就会按照LRU最近虽少使用算法来移除缓存项
                .maximumSize(100)
                //设置要统计缓存的命中率
                .recordStats()
                //设置缓存的移除通知
                .removalListener(new RemovalListener<Object, Object>() {
                    @Override
                    public void onRemoval(RemovalNotification<Object, Object> notification) {
                        System.out.println(notification.getKey() + " 被移除了，原因： " + notification.getCause());
                    }
                })
                //build方法中可以指定CacheLoader，在缓存不存在时通过CacheLoader的实现自动加载缓存
                .build(
                        new CacheLoader<String, User>() {
                            @Override
                            public User load(String key) throws Exception {
                                System.out.println("缓存没有时，从数据库加载" + key);
                                return new User("tony" + key, key);
                            }
                        }
                );

        // 第一次读取
        for (int i = 0; i < 10; i++) {
            User user = userCache.get("uid" + i);
            System.out.println(user);
        }

        // 第二次读取
        for (int i = 0; i < 10; i++) {
            User user = userCache.get("uid" + i);
            System.out.println(user);
        }
        System.out.println("cache stats:");
        //最后打印缓存的命中率等 情况
        System.out.println(userCache.stats().toString());
    }
}
```

> User.java 代码如下：

``` java
package me.xueyao.cache.java.pojo;

import java.io.Serializable;

/**
 * @author simon
 */
public class User implements Serializable {
    private String userName;
    private String userId;

    public User(String userName, String userId) {
        this.userName = userName;
        this.userId = userId;
    }

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public String getUserName() {
        return userName;
    }

    @Override
    public String toString() {
        return userId + " --- " + userName;
    }
}
```

运行后的结果如下：

![](https://ueyao.github.io/image-hosting/blog/2019/8/cache-guava-02.png)

第一次循环时缓存中没有数据，构建了缓存，第二次直接命中缓存。如果程序需要单机内存缓存，可以用该方式构建缓存。