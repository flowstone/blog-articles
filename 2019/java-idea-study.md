---
title: "Java编程思想学习总结一(一切都是对象)"
date: 2019-02-07 12:24:05
draft: false
categories: ["JS"]
---

## 存储位置

* 寄存器
* 堆栈  存储对象引用，堆栈指针向下移动，分配新的内存，向上移动，释放内存
* 堆  存储Java对象
* 常量存储  存储常量值
* 非RAM存储      存储流对象和持久化对象

## 基本类型所占存储空间

|基本类型|大小|包装器类型|默认值|
|-------|--|---------|-----|
|boolean|16bit|Boolean|false|
|char|16bit|Character|‘\u0000’（null）|
|byte|8bit|Byte|0|
|short|16bit|Short|0|
|int|32bit|Integer|0|
|long|64bit|Long|0L|
|float|32bit|Float|0.0F|
|double|64bit|Double|0.0D|

永远不需要销毁对象

执行new来创建对象时，数据存储空间才被分配

static作用于某个字段时，都只有一份存储空间

## 文档注释

javadoc只能为public和protected成员进行文档注释

@see  : 引用其化类

{@link package.class#member label}  用于行内，并且是用label作为超链接

{@docRoot} 该标签产生到文档根目录的相对路径，用于文档树页面的显式超链接

{@inheritDoc} 该标签从当前这个类的最直接的基类中继承相关文档到当前的文档注释中

 @version  生成版本

 @author  作者信息

 @since  该标签允许你指定程序代码最早使用的版本

 @parm  参数列表的标识符

 @return  用来描述返回值的含义，可以延续数行

 @throws  异常说明

 @deprecated  该标签用于指出一些旧特性已由改进的新特性所取代

## 编码风格

类名的首字母要大写，如果几个单词构成，每个单词首字母都采用大写形式