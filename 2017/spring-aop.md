---
title: "Spring的AOP编程"
date: 2017-10-16 12:06:56
draft: false
categories: ["Spring"]
---

AOP为Aspect Oriented Programming(面向切面编程)

AOP的好处：在不修改源代码的情况下，可以实现功能的增强

## JDK动态代理
缺点：只能针对实现了接口的类实现代理
``` java
/**
 * Jdk的动态代理
 * @author kevin
 */
public class JdkProxy implements InvocationHandler{

    //要代理的对象
    private CustomerDao customerDao;

    public JdkProxy(CustomerDao customerDao){
        this.customerDao = customerDao;
    }
    /**
    * 生成代理对象的方法
    * @return
    */
    public CustomerDao createProxy(){
        CustomerDao proxy = (CustomerDao) Proxy.newProxyInstance(customerDao.getClass().getClassLoader(), customerDao.getClass().getInterfaces(), this);
        return proxy;
    }

    /**
    * 增强的方法
    */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("权限校验...");
        return method.invoke(customerDao, args);
    }
}

//编写测试代码：

@Test
public void test(){
    CustomerDao customerDao = new CustomerDaoImpl();
    JdkProxy jdkProxy = new JdkProxy(customerDao);
    CustomerDao proxy = jdkProxy.createProxy();
    proxy.save();
}
```

## CGLIB动态代理(了解)
可以针对那些没有实现接口的类实现代理
``` java
public class CglibProxy implements MethodInterceptor{

    //要代理的对象
    private LinkManDao linkManDao;

    public CglibProxy(LinkManDao linkManDao) {
        this.linkManDao = linkManDao;
    }

    public LinkManDao createProxy(){
        //创建Cglib核心类
        Enhancer enhancer = new Enhancer();
        //设置父类
        enhancer.setSuperclass(linkManDao.getClass());
        //设置回调
        enhancer.setCallback(this);
        //生成代理
        LinkManDao proxy = (LinkManDao) enhancer.create();
        return proxy;
    }

    @Override
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("日志记录");
        Object obj = methodProxy.invokeSuper(proxy, args);
        return obj;
    }
}

//编写测试代码
@Test
public void test2(){
    LinkManDao linkManDao = new LinkManDao();
    CglibProxy cglibProxy = new CglibProxy(linkManDao);
    LinkManDao proxy = cglibProxy.createProxy();
    proxy.save();
}
```

术语
1. Target: 目标对象。要对类增强，增强的类就称为目标对象。
2. JoinPoint: 连接点。可以被增强的点就称为JoinPoint。
3. PointCut: 切入点。在实际开发中真正被增强了的点就称为PointCut。
4. Advice: 增强(通知)。现在需要对save方法增强一个日志记录的功能，就需要编写一个日志记录的方法。那么这个日志记录的方法就称为Advice。
5. Weaving: 织入。 把Advice应用到目标的过程叫织入。
6. Introduction: 引介。 指的是类层面的增强。
7. Proxy: 代理。 增强之后的对象。
8. Aspect: 切面。 多个增强和多个切入点的组合就叫切面。

``` java
public class UserDaoImpl{
    public void save(){}
    public void list(){}
    public void delete(){}
    public void update(){}
}
```

## SpringXML形式的AOP入门

### 配置
导入jar包（11）
![](https://ueyao.github.io/image-hosting/blog/2017/10/2017-10-15_170353.png)

红线画起来的表示：

AOP联盟的jar包：com.springsource.org.aopalliance-1.0.0.jar

Spring提供的AOP的jar包：spring-aop-4.2.4.RELEASE.jar

AspectJ的jar包：com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar

Spring整合AspectJ的jar包：spring-aspects-4.2.4.RELEASE.jar

配置文件
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="customerDao" class="me.xueyao.dao.impl.CustomerDaoImpl"></bean>
    <bean id="myAspectXml" class="me.xueyao.aspect.MyAspectXml"></bean>
    <!-- AOP配置 -->
    <aop:config>
            <!-- 配置切入点 -->
            <aop:pointcut expression="execution(* me.xueyao.dao.impl.CustomerDaoImpl.save(..))" id="pointcut1"/>
            <!-- 配置切面 -->
            <aop:aspect ref="myAspectXml">
                <aop:before method="checkPrivilege" pointcut-ref="pointcut1"/>
            </aop:aspect>
    </aop:config>
</beans>
```

切入点表达式

语法：[修饰符] 返回类型 包名.类名.方法名(形式参数)

常见写法：
``` java
execution(public * *(..))  所有的public方法
execution(* set(..))    所有set开头的方法
execution(* com.xyz.service.AccountService.*(..))   AccountService类中的所有方法
execution(* com.xyz.service.*.*(..))    com.xyz.service包下所有的方法
execution(* com.xyz.service..*.*(..))   com.xyz.service包及其子包下所有的方法
```

Spring中AOP的通知类型

1.前置通知： 在方法执行之前增强。可以获得切入点信息。
``` xml
<!-- 配置切面 -->
<aop:aspect ref="myAspectXml">
    <aop:before method="checkPrivilege" pointcut-ref="pointcut1"/>
</aop:aspect>
```

2.后置通知： 在方法执行完之后增强。可以获取返回值信息。
``` xml
<aop:aspect ref="myAspectXml">
    <!-- 前置通知 -->
    <aop:before method="checkPrivilege" pointcut-ref="pointcut1"/>
    <!-- 后置通知 -->
    <aop:after-returning method="afterReturn" pointcut-ref="pointcut2" returning="result"/>
</aop:aspect>
```

3.环线通知: 在方法执行前后都进行增强。可以阻止方法的执行。
``` java
public class CustomerDaoImpl implements CustomerDao {
    @Override
    public void update() {
        System.out.println("持久层：客户更新");
    }

}

public class MyAspectXml {
    public Object around(ProceedingJoinPoint joinpoint){
        System.out.println("环绕前执行");
        Object obj = null;
        try {
            obj = joinpoint.proceed();
        } catch (Throwable e) {
            e.printStackTrace();
        }
        System.out.println("环绕后执行");
        return obj;
    }
}
```
``` xml
<!-- AOP配置 -->
<aop:config>
    <!-- 配置切入点 -->
    <aop:pointcut expression="execution(* me.xueyao.dao.impl.CustomerDaoImpl.update(..))" id="pointcut3"/>
    <!-- 配置切面 -->
    <aop:aspect ref="myAspectXml">
        <!-- 环绕通知 -->
        <aop:around method="around" pointcut-ref="pointcut3"/>
    </aop:aspect>
</aop:config>
```

4.异常抛出通知： 当发生异常之后增强，可以获取异常信息
``` java
public class CustomerDaoImpl implements CustomerDao {
    @Override
    public void find() {
        System.out.println("持久层：查询");
        int i = 10/0;
    }
}

public class MyAspectXml {
    public void afterThrowing(Exception ex){
        System.out.println("抛出异常通知:" + ex.getMessage());
    }
}
```
``` xml
<!-- AOP配置 -->
<aop:config>
    <!-- 配置切入点 -->
    <aop:pointcut expression="execution(* me.xueyao.dao.impl.CustomerDaoImpl.find(..))" id="pointcut4"/>
    <!-- 配置切面 -->
    <aop:aspect ref="myAspectXml">
        <!-- 抛出异常通知 -->
        <aop:after-throwing method="afterThrowing" pointcut-ref="pointcut4" throwing="ex"/>
    </aop:aspect>
</aop:config>
```

5.最终通知： 不管是否有异常，都会执行的
``` java
public class CustomerDaoImpl implements CustomerDao {
    @Override
    public void find() {
        System.out.println("持久层：查询");
        int i = 10/0;
    }
}

public class MyAspectXml {
    public void after(){
        System.out.println("最终通知");
    }
}
```
``` xml
<!-- AOP配置 -->
<aop:config>
    <!-- 配置切入点 -->
    <aop:pointcut expression="execution(* me.xueyao.dao.impl.CustomerDaoImpl.find(..))" id="pointcut4"/>
    <!-- 配置切面 -->
    <aop:aspect ref="myAspectXml">
        <!-- 最终通知 -->
        <aop:after method="after" pointcut-ref="pointcut4"/>
    </aop:aspect>
</aop:config>
```

最终通知和后置通知的区别：

最终通知，不管异常与否，都执行；而后置通知在异常时不执行。

Spring注解形式的AOP入门
创建工程，引入jar包，创建核心配置文件(和XML引入的包相同)

配置文件

applicationContext.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

       <context:component-scan base-package="me.xueyao"></context:component-scan>
        <!-- 开启自动代理注解 -->
       <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```
创建接口和实现类
``` java
public interface CustomerDao {
    /**
     * 持久层：客户保存
     */
    public void save();

    public int delete();

    public void update();

    public void find();

}

@Repository("customerDao")
public class CustomerDaoImpl implements CustomerDao {

    @Override
    public void save() {
        System.out.println("持久层：客户保存...");
    }

    public int delete(){
        System.out.println("持久层：客户删除...");
        return 100;
    }

    @Override
    public void update() {
        System.out.println("持久层：客户更新");
    }

    @Override
    public void find() {
        System.out.println("持久层：查询");
    }

}
```

编写切面类

``` java
@Component("myAspectAnnotation")
@Aspect //可以省略id
public class MyAspect {

    @Before("execution(* me.xueyao.dao.impl.CustomerDaoImpl.save(..))")
    public void checkPrivilege(JoinPoint joinPoint){
        System.out.println("权限校验..." + joinPoint.toString());
    }

}
```

Spring的AOP中注解通知

1.前置通知
``` java
/**
* 前置通知
* @param joinPoint
*/
@Before("execution(* me.xueyao.dao.impl.CustomerDaoImpl.save(..))")
public void checkPrivilege(JoinPoint joinPoint){
    System.out.println("权限校验..." + joinPoint.toString());
}
```

2.后置通知
``` java
@Aspect
public class MyAspect {
    /**
    * 后置通知
    * @param result
    */
    @AfterReturning(value="execution(* me.xueyao.dao.impl.CustomerDaoImpl.delete(..))",returning="result")
    public void afterReturning(Object result){
        System.out.println("后置通知:" + result);
    }
}
```

3.环绕通知
``` java
@Around("execution(* me.xueyao.dao.impl.CustomerDaoImpl.update(..))")
public Object after(ProceedingJoinPoint joinpoint) throws Throwable{
    System.out.println("环绕通知前增强");
    Object obj = joinpoint.proceed();
    System.out.println("环绕通知后增强");
    return obj;
}
```

4.异常通知
``` java
@AfterThrowing(value="execution(* me.xueyao.dao.impl.CustomerDaoImpl.find(..))",throwing="ex")
public void afterThrowing(Exception ex){
    System.out.println("抛出异常通知");
}
```

5.最终通知
``` java
@After("execution(* me.xueyao.dao.impl.CustomerDaoImpl.find(..))")
public void after(){
    System.out.println("最终通知");
}
```

6.PointCut注解(了解)

作用：用于定义切入点表达式的一个注解。
``` java
@Component("myAspectAnnotation")
@Aspect
public class MyAspect {

    /**
        * 前置通知
        * @param joinPoint
        */
    @Before("execution(* me.xueyao.dao.impl.CustomerDaoImpl.save(..))")
    public void checkPrivilege(JoinPoint joinPoint){
        System.out.println("权限校验..." + joinPoint.toString());
    }

    @AfterReturning(value="execution(* me.xueyao.dao.impl.CustomerDaoImpl.delete(..))",returning="result")
    public void afterReturning(Object result){
        System.out.println("后置通知:" + result);
    }

    @Around("execution(* me.xueyao.dao.impl.CustomerDaoImpl.update(..))")
    public Object aroung(ProceedingJoinPoint joinpoint) throws Throwable{
        System.out.println("环绕通知前增强");
        Object obj = joinpoint.proceed();
        System.out.println("环绕通知后增强");
        return obj;
    }

    //  @AfterThrowing(value="execution(* me.xueyao.dao.impl.CustomerDaoImpl.find(..))",throwing="ex")
    @AfterThrowing(value="MyAspect.pointcut()",throwing="ex")
    public void afterThrowing(Exception ex){
        System.out.println("抛出异常通知");
    }

    //  @After("execution(* me.xueyao.dao.impl.CustomerDaoImpl.find(..))")
    @After("MyAspect.pointcut()")
    public void after(){
        System.out.println("最终通知");
    }

    @Pointcut("execution(* me.xueyao.dao.impl.CustomerDaoImpl.find(..))")
    public void pointcut(){

    }

}
```






