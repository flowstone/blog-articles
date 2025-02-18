---
title: "如何实现Oracle先组内排序然后再组外排序"
date: 2022-04-29 14:18:42
draft: false
categories: ["Oracle"]
---

#### 问题描述
工作中遇到一个问题，因为我本人的SQL技术太差了，写了好久，都没有处理好，大概的需求如下，有一个列表，根据一个字段排序，排序后的结果，再根据字段排序。

#### 问题分析
为了让读者能够充分理解这个问题，先分解问题
原始数据如下：

|序号	|名称	|部门|入职时间  |等级|
|------|-------|----|---------|------|
|1	|小明	|开发部	|2012-10|	1|
|2	|小丽	|账务部	|2013-01|	1|
|3	|小华	|开发部	|2021-01|	3|
|4	|小红	|开发部	|2001-01|	2|
|5	|小张	|账务部	|2022-01|	2|

##### 1、先根据部门分组，然后根据等级排序(正序)
预期结果如下

|序号	|名称	|部门	|入职时间	|等级|
|------|-------|----|---------|------|
|1	|小明	|开发部	|2012-10	|1|
|2	|小红	|开发部	|2001-01	|2|
|3	|小华	|开发部	|2021-01	|3|
|4	|小丽	|账务部	|2013-01	|1|
|5	|小张	|账务部	|2022-01	|2|

##### 2、先根据部门分组，然后根据入职排序(倒序)
预期结果如下

|序号	|名称	|部门	|入职时间	|等级|
|------|-------|----|---------|------|
|1	|小丽	|账务部	|2013-01|	1|
|2	|小张	|账务部	|2022-01|	2|
|3	|小明	|开发部	|2012-10|	1|
|4	|小红	|开发部	|2001-01|	2|
|5	|小华	|开发部	|2021-01|	3|

#### 解决步骤
指定字段分组，组内排序和组外排序
``` sql
select
	T1.*
from
	(
	select
		ID,
		DEPARTMENT,
		CREATE_TM,
		level,
		row_number() over (partition by DEPARTMENT
	order by
		level desc) as NUM
	from
		USER_INFO
	where
		DEPARTMENT is not null) T1
left join (
	select
		ROWNUM SEQ,
		DEPARTMENT
	from
		(
		select
			DEPARTMENT,
			MAX(CREATE_TM)
		from
			USER_INFO
		group by
			DEPARTMENT
		order by
			MAX(CREATE_TM))) T2 on
	T1.DEPARTMENT = T2.DEPARTMENT
order by
	T2.SEQ desc,
	T1.LEVEL asc;
```
说明，T1表是根据DEPARTMENT分组并按照level组内排序(正序)，T2表是根据DEPARTMENT分组并按照创建时间组外排序(倒序)

