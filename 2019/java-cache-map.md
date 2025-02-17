---
title: "Java内存缓存-通过Map定制简单缓存"
date: 2019-08-22 12:37:04
draft: false
categories: ["Java"]
---

## 缓存

在程序中，缓存是一个高速数据存储层，其中存储了数据子集，且通常是短暂性存储，这样日后再次请求此数据时，速度要比访问数据的主存储位置快。通过缓存，可以高效地重用之前检索或计算的数据。

### 为什么要用缓存

![](https://ueyao.github.io/image-hosting/blog/2019/8/cache-define-simple-01.png)



### 场景

在Java应用中，对于访问频率高，更新少的数据，通常的方案是将这类数据加入缓存中，相对从数据库中读取，读缓存效率会有很大提升。

在集群环境下，常用的分布式缓存有Redis、Memcached等。但在某些业务场景上，可能不需要去搭建一套复杂的分布式缓存系统，在单机环境下，通常是会希望使用内部的缓存(LocalCache)。



### 方案

* 基于JSR107规范自研

* 基于ConcurrentHashMap实现数据缓存

#### JSR107规范目标

* 为应用程序提供缓存Java对象的功能。

* 定义了一套通用的缓存概念和工具。

* 最小化开发人员使用缓存的学习成本。

* 最大化应用程序在使用不同缓存实现之间的可移植性。

* 支持进程内和分布式的缓存实现。



#### JSR107规范核心概念

- Java Caching定义了5个核心接口，分别是CachingProvider, CacheManager, Cache, Entry 和 Expiry。

-  CachingProvider定义了创建、配置、获取、管理和控制多个CacheManager。一个应用可以在运行期访问多个CachingProvider。

- CacheManager定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache存在于- CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。

- Cache是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。

- Entry是一个存储在Cache中的key-value对。

- 每一个存储在Cache中的条目有一个定义的有效期，即Expiry Duration。

一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。

### 小例子

使用Map来实现一个简单的缓存功能

> MapCacheDemo.java

``` java
package me.xueyao.cache.java;

import java.lang.ref.SoftReference;
import java.util.Optional;
import java.util.concurrent.ConcurrentHashMap;


/**
 * @author simon
 * 用map实现一个简单的缓存功能
 */
public class MapCacheDemo {

    /**
     * 使用  ConcurrentHashMap，线程安全的要求。
     * 我使用SoftReference <Object>  作为映射值，因为软引用可以保证在抛出OutOfMemory之前，如果缺少内存，将删除引用的对象。
     * 在构造函数中，我创建了一个守护程序线程，每5秒扫描一次并清理过期的对象。
     */
    private static final int CLEAN_UP_PERIOD_IN_SEC = 5;

    private final ConcurrentHashMap<String, SoftReference<CacheObject>> cache = new ConcurrentHashMap<>();

    public MapCacheDemo() {
        Thread cleanerThread = new Thread(() -> {
            while (!Thread.currentThread().isInterrupted()) {
                try {
                    Thread.sleep(CLEAN_UP_PERIOD_IN_SEC * 1000);
                    cache.entrySet().removeIf(entry ->
                            Optional.ofNullable(entry.getValue())
                                    .map(SoftReference::get)
                                    .map(CacheObject::isExpired)
                                    .orElse(false));
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });
        cleanerThread.setDaemon(true);
        cleanerThread.start();
    }

    public void add(String key, Object value, long periodInMillis) {
        if (key == null) {
            return;
        }
        if (value == null) {
            cache.remove(key);
        } else {
            long expiryTime = System.currentTimeMillis() + periodInMillis;
            cache.put(key, new SoftReference<>(new CacheObject(value, expiryTime)));
        }
    }

    public void remove(String key) {
        cache.remove(key);
    }

    public Object get(String key) {
        return Optional.ofNullable(cache.get(key)).map(SoftReference::get).filter(cacheObject -> !cacheObject.isExpired()).map(CacheObject::getValue).orElse(null);
    }

    public void clear() {
        cache.clear();
    }

    public long size() {
        return cache.entrySet().stream().filter(entry -> Optional.ofNullable(entry.getValue()).map(SoftReference::get).map(cacheObject -> !cacheObject.isExpired()).orElse(false)).count();
    }

    /**
     * 缓存对象value
     */
    private static class CacheObject {
        private Object value;
        private long expiryTime;

        private CacheObject(Object value, long expiryTime) {
            this.value = value;
            this.expiryTime = expiryTime;
        }

        boolean isExpired() {
            return System.currentTimeMillis() > expiryTime;
        }

        public Object getValue() {
            return value;
        }

        public void setValue(Object value) {
            this.value = value;
        }
    }
}
```

> 代码测试类MapCacheDemoTests.java

``` java
package me.xueyao.cache.java;

public class MapCacheDemoTests {
    public static void main(String[] args) throws InterruptedException {
        MapCacheDemo mapCacheDemo = new MapCacheDemo();
        mapCacheDemo.add("uid_10001", "{1}", 5 * 1000);
        mapCacheDemo.add("uid_10002", "{2}", 5 * 1000);
        mapCacheDemo.add("uid_10003", "{3}", 5 * 1000);
        System.out.println("从缓存中取出值:" + mapCacheDemo.get("uid_10001"));
        Thread.sleep(5000L);
        System.out.println("5秒钟过后");
        System.out.println("从缓存中取出值:" + mapCacheDemo.get("uid_10001"));
        // 5秒后数据自动清除了~
    }
}
```

