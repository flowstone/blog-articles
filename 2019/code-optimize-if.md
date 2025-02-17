---
title: "代码优化-多态代替IF条件判断"
date: 2019-12-01 12:45:00
draft: false
categories: ["Java"]
---


## 场景描述

在开发的场景中，常常会遇到打折的业务需求，每个用户对应的等级，他们的打折情况也是不一样的。例如普通会员打9折，青铜会员打8.5折，黄金会员打8折等等。在一般开发中最简单的就是判断用户的等级，然后对订单作对应的打折处理。

## 场景示例

写了一个简单的小示例，如下所示：

```java
//1 代表学生 2老师   3校长
int type = 1;
if (1 == type) {
    System.out.println("学生笑嘻嘻的说话");
} else if (2 == type) {
    System.out.println("老师开心的说话");
} else {
    System.out.println("校长严肃的说话");
}
```

上面的代码，是我们经常的做法，代码少的时候，看起来非常清晰，但是代码多起来或者有了更多的判断条件，那上面的代码会更加的混乱，如果每次有修改，都要改动这部分代码。

## 解决方法

可以把上面的代码改成多态方式，创建三个类，学生Student,老师Teacher,校长HeadMater，父类为Person，这三个类都实现父类的方法say，如下所示:

Person.class

```java
package me.xueyao.service;

/**
 * @author Simon.Xue
 * @date 2019-12-01 14:31
 **/
public interface Person {
    void say();
}
```

Student.class

```java
package me.xueyao.service.impl;

import me.xueyao.service.Person;
import org.springframework.stereotype.Service;

/**
 * @author Simon.Xue
 * @date 2019-12-01 14:34
 **/
@Service
public class Student implements Person {
    @Override
    public void say() {
        System.out.println("学生笑嘻嘻的说话");
    }
}
```

Teacher.class

```java
package me.xueyao.service.impl;

import me.xueyao.service.Person;
import org.springframework.stereotype.Service;

/**
 * @author Simon.Xue
 * @date 2019-12-01 14:37
 **/
@Service
public class Teacher implements Person {
    @Override
    public void say() {
        System.out.println("老师开心的说话");
    }
}
```

HeadMaster.class

```java
package me.xueyao.service.impl;

import me.xueyao.service.Person;
import org.springframework.stereotype.Service;

/**
 * @author Simon.Xue
 * @date 2019-12-01 14:41
 **/
@Service
public class HeadMaster implements Person {

    @Override
    public void say() {
        System.out.println("校长严肃的说话");
    }
}
```

测试方法

```java
@Test
public void testSay() {
    Person student = new Student();
    student.say();

    Person teacher = new Teacher();
    teacher.say();

    Person headMaster = new HeadMaster();
    headMaster.say();
}
```

## 优化

上面的这种做法，基本上是完成了优化，但是我们还会发现了一个问题，就是每次我们还是要创建对应的对象。上面有三个类，我们就要创建有三个对象，能否再次优化一下？

因为现在项目用Sping框架，所以可以用注入来完成优化。

首先，创建一个Person枚举类,如下所示：

```java
package me.xueyao.enums;

import lombok.AllArgsConstructor;
import lombok.Getter;
import me.xueyao.service.impl.HeadMaster;
import me.xueyao.service.impl.Student;
import me.xueyao.service.impl.Teacher;

/**
 * @author Simon.Xue
 * @date 2019-12-01 15:55
 **/
@AllArgsConstructor
@Getter
public enum  PersonEnums {
    STUDENT(1, "学生", Student.class),
    TEACHER(2, "老师", Teacher.class),
    HEADMASTER(3, "校长", HeadMaster.class);

    Integer code;
    String msg;
    Class clazz;

    /**
     * 获得类的名称，因为Spring自动注入时，默认名称是类名(首字母小写)
     * @param code
     * @return
     */
    public static String className(Integer code) {
        for (PersonEnums value : values()) {
            if (value.getCode().equals(code)) {
                String simpleName = value.getClazz().getSimpleName();
                simpleName.substring(1);
                return String.valueOf(simpleName.charAt(0)).toLowerCase() + simpleName.substring(1);
            }
        }
        return "";
    }

}
```

使用方式 ：

```java
@Autowired
private Map<String, Person> personMap = new HashMap<>();
@Test
public void testSay() {
    personMap.get(PersonEnums.className(2)).say();
}
```