---
title: "MySQL中INNER JOIN详解"
date: 2019-05-19 12:25:38
draft: false
categories: "Mysql"
---

## 前言
MySQL中联合查询有join、inner join、left join、right join、full join这五种组成。join和inner join是相等的。一般涉及到多张数据表之前的关联查询，必须要用连接操作符。本次来讲讲inner join的使用。

## 使用
首先，为了能够形象的展示Inner join的效果，创建两张相关联的数据表，学生表和成绩表

学生表创建语句如下：
``` sql
create table student(
    id int not null auto_increment,
    name varchar(64) not null default '' comment '姓名',
    primary key(id)
 ) charset=utf8mb4 comment '学生表';
```

成绩表创建语句如下：
``` sql
create table score(
    id int not null auto_increment,
    student_id varchar(64) not null default '' comment '学生id',
    chinese varchar(4) not null default '' comment '中文',
    primary key(id)
 ) charset=utf8mb4 comment '学生表';
```

学生表的数据如下：

|ID|NAME|
|--|----|
|1|小明|
|2|小华|
|3|小丽|
4||小花|

成绩表数据如下

|ID	|STUDENT_ID|CHINESE|
|---|----------|--------|
|1|	1	|89|
|2	|2	|78|
|3	|3	|64|
|4	|5	|96|
问题1: 查询有成绩的学生信息

### 步骤1
查询两个表连接的数据
``` sql
select * from student inner join score
```

结果如下：
![查询结果](https://ueyao.github.io/image-hosting/blog/2019/2019-05-19-7.44.24.png)


这个结果有许多相同的数据，这个情况叫做笛卡尔积，如何避免笛卡积，需要在内连接查询上添加查询条件就可以解决这个问题。

### 步骤2
查询两个表连接的数据带条件
``` sql
select s.id,s.name,sc.chinese  from student s inner join score sc on s.id = sc.student_id
```
查询的结果如下：
![查询结果](https://ueyao.github.io/image-hosting/blog/2019/2019-05-19-7.57.07.png)

从上图中，可以看到没有小花的成绩数据，因为内连接只会显示两个表中都有的数据。用集合的方式来说就是取两个表数据的交集。

总结
开发中常常会查询相关联的数据，并且查询中只保留两表中共有的数据，这样就可以使用inner join来关联查询。

