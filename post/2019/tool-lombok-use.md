---
title: "Lombok的使用"
date: 2019-02-03 12:23:17
draft: false
categories: "Tool"
---

近期在众多微信公众号中，都看到了许多大牛，写了Lombok的文章，我看了一下，基本上就围绕着如何减少代码来做说明，我来总结一下。

公司现在的项目没有使用Lombok，一些实体类都是我们用IDEA提供的快捷方式生成的，后来，公司新来了一个大牛，看到我们每当在一个实体类中新添加一个属性，就要重新生成一个Getter/Setter方法，浪费了大量的时间，一个实体类改了好多遍。后来，他叫我们使用Lombok插件，这款插件可以在编译的时候生成一些常用的方法，我们就不用那么浪费时间。

公司使用的是Maven包管理工具

``` xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

在使用的编辑器中添加lombok插件，我使用的是IntelliJ IDEA这款软件，国内用的比较多，大家应该知道安装一个插件。

在实体类上添加Lombok相关注解，来实现，生成对应方法

``` java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor
@Data
public class LoginRequest {
    private String username;
    private String password;
}
```

@Getter 在类上说明生成该类中所有属性的Getter方法

@Setter 在类上说明生成该类中所有属性的Setter方法

上面两个方法，也可以使用在属性上,就是生成属性的Getter/Setter方法

@NoArgsConstructor注解，是生成无参构造方法

@AllArgsConstructor注解，是生成有参构造方法

@RequiredArgsConstructor注解，是生成不能为空属性的构造方法

@Data注解是生成Setter、Getter、toString等常用的方法

一般来说，@Data注解就够我们使用的了，如果有特殊的Getter、Setter需求，则需要自己生成对应的方法。最后，总结一些话，Lombok插件虽然能够帮助我们生成对应的方法，但是我们不能足够的依赖一个插件，如果实体类的属性命名不规范，Lombok生成的方法还是会有一些问题的，往往一个小的问题，是一个潜在的危险，我们应当能避免就避免。

常见问题，当项目运行时，如果某个属性没有Getter、Setter方法时，一定要看一下Lombok插件有没有安装。