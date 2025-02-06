---
title: "JPA使用-实体类上常用注解"
date: 2019-12-11 12:45:59
draft: false
categories: "Java"
---

## @SQLDelete

### 场景描述

JPA中提供了简单的CRUD操作，其中删除操作是物理删除，但是实际应用中，系统中的数据是一种资源，不能直接删除，应该做到逻辑删除，JPA中删除操作是不可取的。

### 场景示例

调用JPA的删除方法，如下代码所示：

``` java
@Test
public void testJpaDelete() {
  //此处根据id删除角色信息
  roleRepository.deleteById(1);
}
```

执行上面的测试方法，数据表中主键为1的数据，已经被删除掉，看下JPA的执行SQL如下所示:

``` mysql
delete from role where id=?
```

此语句为JPA删除操作的默认执行语句。

### 解决方案

JPA的默认删除方法，并不可取，可以在Role实体上加上@SQLDelete注解，并写SQL语句，如下所示:

``` java
@SQLDelete(sql = "update role set is_deleted = 1 where id = ?")
```

上面的注解代表着，只要执行JPA的删除操作，执行的SQL语句为我们自己定义的SQL语句。

#### 测试一下

``` java
 @Test
public void testJpaDelete() {
  roleRepository.deleteById(2);
}
```

结果打印的SQL执行语句，如下所示

``` mysql
update role set is_deleted = 1 where id = ?
```

## @DynamicInsert

### 场景描述

在JPA中添加/更新都是使用save()方法，一般情况下，创建数据表的时候，会给某些字段设置默认的值，避免在插入的时候手动赋值，如创建时间，是否删除等等。

save方法会把没有值的对象，默认赋空值，造成，原数据表的默认值失效。

### 场景示例

### 解决方案



## @DynamicUpdate

