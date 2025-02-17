---
title: "Spring自定义注解详解"
date: 2019-02-13 12:24:52
draft: false
categories: ["Spring"]
---

下面是RequestBody注解源码

``` java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestBody {

	/**
	 * Whether body content is required.
	 * <p>Default is {@code true}, leading to an exception thrown in case
	 * there is no body content. Switch this to {@code false} if you prefer
	 * {@code null} to be passed when the body content is {@code null}.
	 * @since 3.2
	 */
	boolean required() default true;

}
```
现在讲一讲@RequestBody注解中用到其它注解

@Target注解

从字面上理解这个就是目标的意思，说明@RequestBody注解是作用于哪个上面

ElementType
``` java
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```
它的值有下列

|值	|说明(适用范围)|
|---|-----------|
|TYPE|	类，接口(包括注解类型)，枚举|
|FIELD|	字段|
|METHOD|	方法|
|PARAMETER|	参数|
|CONSTRUCTOR|	构造方法|
|LOCAL_VARIABLE|	局部变量|
|ANNOTATION_TYPE|	注解类型|
|PACKAGE|	包|
|TYPE_PARAMETER|	类型参数，从1.8开始|
|TYPE_USE|	类型的使用，从1.8开始|

@Retention注解

从字面上理解这个就是保留的意思，说明@RequestBody注解是在哪些环境下保留此注解

RetentionPolicy
``` java
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```


* SOURCE  在编译时不保留此注解，此注解存在源代码中|
* CLASS   在编译时保留注解于class文件中，VM运行时不需要保留，此值为默认值|
* RUNTIME 在编译时保留注解于class文件中，VM运行时保留此注解，所以它们可以被反射阅读|

@Documented注解

此注解说明，如果生成javadoc文档时，会将该注解加入到文档中。
