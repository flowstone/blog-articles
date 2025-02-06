---
title: "Spring Cloud入门教程一之Eureka Server"
date: 2018-10-13 12:18:13
draft: false
categories: "Spring"
---
### 项目环境
* MacOS
* JDK1.8
* IntelliJ IDEA 2018.2
* Maven 3.5.4

### 创建项目
* 采用Spring Initializr创建项目
* 选择Cloud Discovery->Eureka Discovery->项目名称

``` java
package me.xueyao;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
//注解该类是EurekaServer服务
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

配置文件application.yml
``` yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8762/eureka/
    register-with-eureka: false  #是否在eureka中注册，false不用注册
    fetch-registry: false #是否发现注册，不注册
  server:
    enable-self-preservation: false # false表示在此eureka服务器中关闭自我保护模式#server:#  port: 8761  #服务端口spring:
  application:
    name: eureka-server #应用名
```

启动项目，在浏览器中输入EurekaServer地址http://127.0.0.1:8761,如果页面能正常显示则说明创建Eureka Server成功

