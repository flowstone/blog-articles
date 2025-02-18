---
title: "Quartz作业调度的入门使用"
date: 2017-11-14 12:09:41
draft: false
categories: ["Java"]
---

## 概念
1.Job

表示一个工作，要执行的具体的内容。此接口中只有一个方法

2.JobDetail

JobDetail表示一个具体的可执行的调度程序，Job是这个可执行调度程序所要执行的内容

3.Trigger

Trigger代表一个调度参数的配置

4.Scheduler

Scheduler代表一个调度容器，一个调度容器中可以注册多个JobDetail和Trigger.

说明：

* 编写job实现业务，要做什么具体事情
* 使用JobDetail包装job，是任务对象，可以被调度
* 使用Trigger定制什么时候去调用某任务对象
* 使用Scheduler结合任务对象和触发器对象

## 第一个任务
1.引入依赖坐标

pom.xml
``` xml
<dependencies>
    <dependency>
        <groupId>org.quartz-scheduler</groupId>
        <artifactId>quartz</artifactId>
        <version>2.2.3</version>
    </dependency>
    <dependency>
        <groupId>org.quartz-scheduler</groupId>
        <artifactId>quartz-jobs</artifactId>
        <version>2.2.3</version>
    </dependency>
    <!-- slf4j log4j -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.7</version>
    </dependency>
</dependencies>
```

2.简单测试类

QuartzTest.java

``` java
package me.xueyao.quartz.test;

import org.junit.Test;
import org.quartz.Scheduler;
import org.quartz.impl.StdSchedulerFactory;

/**
 * @author XueYao
 * @date 2017-11-26
 */
public class QuartzTest {
    //quartz容器启动和关闭
    @Test
    public void testFirst() throws Exception {
        //Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
        Scheduler scheduler = new StdSchedulerFactory().getScheduler();
        // and start it off
        scheduler.start();
        //任务写在此处
        scheduler.shutdown();

    }
}
```

## SIMPLETRIGGER简单任务
1.创建业务任务job：

HelloJob.java
``` java
package me.xueyao.simpletrigger.test;

import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

/**
 * @author XueYao
 * @date 2017-11-26
 */
public class HelloJob  implements Job{
    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.out.println("hello quartz... thanks");
    }
}
```

2.编写调度代码：

SimpleTriggerTest.java
``` java
package me.xueyao.simpletrigger.test;

import org.junit.Test;
import org.quartz.*;
import org.quartz.impl.StdSchedulerFactory;

import static org.quartz.JobBuilder.newJob;
import static org.quartz.SimpleScheduleBuilder.simpleSchedule;
import static org.quartz.TriggerBuilder.newTrigger;

/**
 * @author XueYao
 * @date 2017-11-26
 */
public class SimpleTriggerTest {
    //简单任务
    @Test
    public void testSimpleTrigger() throws SchedulerException {
        SchedulerFactory sf = new StdSchedulerFactory();
        Scheduler scheduler = sf.getScheduler();

        // define the job and tie it to our HelloJob class
        //构建调度任务详情对象:包装job，并指定任务的组group1和任务名字job1
        JobDetail job = newJob(HelloJob.class)
                .withIdentity("job1", "group1")
                .build();

        // Trigger the job to run now, and then repeat every 40 seconds
        //构建一个触发器(触发规则、什么时候如何调用任务的规则)
        /*
        * @param1 解决器的名字
        * @param2  触发任务的组的名字
        * */
        Trigger trigger = newTrigger()
                .withIdentity("trigger1", "group1")
                .startNow() //马上开始执行任务
                .withSchedule(simpleSchedule()//简单任务
                        .withIntervalInSeconds(5) //间隔5秒
                        .repeatForever()) //永远执行下去
                .build();

        // Tell quartz to schedule the job using our trigger
        scheduler.scheduleJob(job, trigger);
        scheduler.start();
        //死循环，让程序一直执行
        while (true) {

        }

    }

}
```

## CRONTRIGGER定时任务
cron表达式
通过7个字符表示计划，字符间用空格分隔即可

1. Seconds 秒
2. Minutes 分钟
3. Hours 小时
4. Day-of-Month 当前月第几天
5. Month 月
6. Day-of-Week 当前周第几天
7. Year 年(可省略)
举例：

0 0 12 ？ * WED

解决： 0秒0分12点每个月星期三

值的解释：
* 0代表数字0
* ?代表未知，不存在,一般用于月的某一天和周的某一天，两者不能同时使用
* WED,代表星期几，通常用1-7表示，1是星期日
* /代表每隔多长刻度，例如0/15，放在分钟上，就表示每隔15分钟执行

测试代码：

``` java
package me.xueyao.crontrigger.test;

import me.xueyao.simpletrigger.test.HelloJob;
import org.junit.Test;
import org.quartz.*;
import org.quartz.impl.StdSchedulerFactory;

import static org.quartz.JobBuilder.newJob;
import static org.quartz.SimpleScheduleBuilder.simpleSchedule;
import static org.quartz.TriggerBuilder.newTrigger;

/**
 * 定时任务
 * @author XueYao
 * @date 2017-11-26
 */
public class CronTriggerTest {

    @Test
    public void testCronTrigger() throws SchedulerException {
        SchedulerFactory sf = new StdSchedulerFactory();
        Scheduler scheduler = sf.getScheduler();

        // define the job and tie it to our HelloJob class
        //构建调度任务详情对象:包装job，并指定任务的组group1和任务名字job1
        JobDetail job = newJob(HelloJob.class)
                .withIdentity("job1", "group1")
                .build();

        //构建一个触发器(触发规则、什么时候如何调用任务的规则)
        /*
        * @param1 解决器的名字
        * @param2  触发任务的组的名字
        * */
        Trigger trigger = newTrigger()
                .withIdentity("trigger1", "group1")
                .startNow() //马上开始执行任务
                .withSchedule(
                        //每隔5秒运行一次
                        //CronScheduleBuilder.cronSchedule("0/5 * * * * ?")
                        CronScheduleBuilder.cronSchedule("0/5 51 18 ? * 1")

                ) //永远执行下去
                .build();

        // Tell quartz to schedule the job using our trigger
        scheduler.scheduleJob(job, trigger);
        scheduler.start();
        while (true) {

        }
    }
}
```

## SPRING整合QUARTZ

1.创建一个Maven java 项目

2.引入依赖坐标

pom.xml

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>me.xueyao.quartz</groupId>
    <artifactId>springQuartz</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- spring context -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.2.9.RELEASE</version>
        </dependency>
        <!-- spring context support -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>4.2.9.RELEASE</version>
        </dependency>
        <!-- spring tx -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>4.2.9.RELEASE</version>
        </dependency>
        <!-- quartz核心 -->
        <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz</artifactId>
            <version>2.2.3</version>
        </dependency>
        <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz-jobs</artifactId>
            <version>2.2.3</version>
        </dependency>
        <!-- slf4j log4j -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.7</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>4.2.9.RELEASE</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```

3.编写job业务任务

HelloJob.java

``` java
package me.xueyao.quartz.job;

import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

/**
 * @author XueYao
 * @date 2017-11-26
 */
public class HelloJob implements Job{

    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        System.out.println("hello quartz Spring ... thanks");
    }
}
```

4.Spring配置文件

applicationContext.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd ">

    <context:component-scan base-package="me.xueyao" />
    <!-- 配置任务详情对象 -->
    <bean id="helloJobDetail" class="org.springframework.scheduling.quartz.JobDetailFactoryBean">
        <!-- 注入job的类型 -->
        <property name="jobClass" value="me.xueyao.quartz.job.HelloJob"/>
        <!-- 数据存放策略 -->
        <property name="jobDataAsMap">
            <map>
                <!-- 任务执行超时时间 -->
                <entry key="timeout" value="5"/>
            </map>
        </property>
    </bean>

    <!-- ======================== 调度触发器 ======================== -->
    <!-- 简单任务 -->
    <bean id="simpleTrigger" class="org.springframework.scheduling.quartz.SimpleTriggerFactoryBean">
        <!-- see the example of method invoking job above
        注入任务
        -->
        <property name="jobDetail" ref="helloJobDetail" />
        <!-- 10 seconds
        第一次开始时间，即准备的时间，毫秒
        -->
        <property name="startDelay" value="5000" />
        <!-- repeat every 50 seconds
        每次执行的间隔时间
        -->
        <property name="repeatInterval" value="3000" />
    </bean>
    <!-- 计划任务 -->
    <!-- <bean id="cronTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
        <property name="jobDetail" ref="exampleJob" />
        run every morning at 6 AM
        <property name="cronExpression" value="0 0 6 * * ?" />
    </bean> -->

    <!-- ======================== 调度工厂 ======================== -->
    <bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
        <property name="triggers">
            <list>
                <ref bean="simpleTrigger" />
            </list>
        </property>
    </bean>
</beans>
```

5.测试代码

SpringQuartzTest.java
``` java
package me.xueyao.springquartz;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

/**
 * @author XueYao
 * @date 2017-11-26
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class SpringQuartzTest {

    @Test
    public void test() {
        while (true) {

        }
    }
}
```

