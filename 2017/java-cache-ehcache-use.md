---
title: "Ehcache缓存简单使用"
date: 2017-11-25 12:10:42
draft: false
categories: ["Java"]
---

#### SPRING整合EHCACHE
##### 1.引入坐标

pom.xml

``` xml
<!-- ehcache的缓存框架 -->
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache-core</artifactId>
    <version>2.6.10</version>
</dependency>
<!-- spring整合第三方框架的 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>${spring.version}</version>
</dependency>
```

##### 2.配置Ehcache

ehcache.xml
``` xml
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
    <!-- 硬盘上缓存的临时目录 -->
    <diskStore path="java.io.tmpdir"/>
    <!-- 
    maxElementsInMemory:内存中最大存放的元素的个数
    eternal：是否永生，默认是false
    timeToIdleSeconds：发呆闲置的时间，超过该时间，被清除，单位秒
    timeToLiveSeconds：存活的事件，超过该时间被清除
    maxElementsOnDisk：如果内存满了，溢出到硬盘上的临时目录中的存放的元素的个数
    diskExpiryThreadIntervalSeconds：轮询时间，巡视组
    memoryStoreEvictionPolicy：内存对象的清理策略，如果满了，怎么办?
    策略有三个：LRU、LFU、FIFO
    LRU:最少使用被清理，次数
    LFU：时间，闲置最长的时间
    FIFO：管道策略，先进先出
     -->

    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxElementsOnDisk="10000000"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
        <persistence strategy="localTempSwap"/>
    </defaultCache>
    <!-- Spring整合的菜单缓存 -->
     <cache name="project_menu_cache"
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxElementsOnDisk="10000000"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
        <persistence strategy="localTempSwap"/>
    </cache>
</ehcache>
```

##### 3.配置Spring整合Ehcache

applicationContext-cache.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
    xmlns:cache="http://www.springframework.org/schema/cache"
    xsi:schemaLocation="
http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context.xsd
http://www.springframework.org/schema/cache
http://www.springframework.org/schema/cache/spring-cache.xsd">
    <!-- 配置ehcache的对象EhCacheManager -->
    <bean id="ehCacheManager"
        class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">
        <!-- 注入ehcache核心配置文件的位置 Default is "ehcache.xml" in the root of the 
            class path, or if not found, "ehcache-failsafe.xml" in the EhCache jar (default 
            EhCache initialization). 可以不配置，默认找类路径下的ehcache.xml -->
        <property name="configLocation" value="classpath:ehcache.xml" />
    </bean>
    <!-- Spring整合Ehache -->
    <!-- Spring的平台缓存管理器 -->
    <bean id="springCacheManager"
        class="org.springframework.cache.ehcache.EhCacheCacheManager">
        <!-- 注入ehcache的对象 -->
        <property name="cacheManager" ref="ehCacheManager"></property>
    </bean>
    <!-- spring的缓存注解驱动 -->
    <cache:annotation-driven cache-manager="springCacheManager" />
</beans>
```

##### 4.在需要操作缓存数据的方法上添加注解
``` java
保存缓存

@Cacheable(value=”cache”) //要缓存查询的结果到Ehchache中,参数是缓存区域

清除缓存

@CacheEvict(value=”cache”,allEntries=true) //
```
#### SHIRO整合EHCACHE
##### 1.引入ehcache，配置ehcache

ehcache.xml
``` xml
<!-- Shiro权限缓存-认证 -->
    <cache name="project_realm_authentication_cache"
        maxElementsInMemory="10000"
        eternal="false"
        timeToIdleSeconds="120"
        timeToLiveSeconds="120"
        maxElementsOnDisk="10000000"
        diskExpiryThreadIntervalSeconds="120"
        memoryStoreEvictionPolicy="LRU">
    <persistence strategy="localTempSwap"/>
</cache>
<!-- Shiro权限缓存-授权 -->
    <cache name="project_realm_authorization_cache"
        maxElementsInMemory="10000"
        eternal="false"
        timeToIdleSeconds="120"
        timeToLiveSeconds="120"
        maxElementsOnDisk="10000000"
        diskExpiryThreadIntervalSeconds="120"
        memoryStoreEvictionPolicy="LRU">
    <persistence strategy="localTempSwap"/>
</cache>
```

##### 2.shiro整合ehchache

pom.xml
``` xml
<!-- shiro整合ehcache -->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-ehcache</artifactId>
    <version>1.3.2</version>
</dependency>
```

##### 3.spring中配置ehcache缓存

applicationContext-shiro.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:jpa="http://www.springframework.org/schema/data/jpa" xmlns:task="http://www.springframework.org/schema/task"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/data/jpa 
        http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

    <!-- 配置Shiro核心Filter  --> 
    <bean id="shiroFilter" 
        class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <!-- 安全管理器 -->
        <property name="securityManager" ref="securityManager" />
        <!-- 未认证，跳转到哪个页面  -->
        <property name="loginUrl" value="/login.html" />
        <!-- 登录页面页面 -->
        <property name="successUrl" value="/index.html" />
        <!-- 认证后，没有权限跳转页面 -->
        <property name="unauthorizedUrl" value="/unauthorized.html" />
        <!-- shiro URL控制过滤器规则  -->
        <property name="filterChainDefinitions">
            <value>
                /login.html* = anon
                /user_login.action* = anon 
                /validatecode.jsp* = anon
                /css/** = anon
                /js/** = anon
                https://ueyao.github.io/image-hosting/blog/** = anon
                /services/** = anon 
                /pages/base/courier.html* = perms[courier:list]
                /pages/base/area.html* = roles[base]
                /** = authc
            </value>
        </property>
    </bean>
    <!-- 安全管理器  -->
    <bean id="securityManager" 
        class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <!-- 注入自定义realm -->
        <property name="realm" ref="bosRealm"></property>
        <!-- 开启Shiro缓存功能，需要在shiro安全管理器中注入shiro的 平台缓存管理器 -->
        <property name="cacheManager" ref="shiroCacheManager" />
    </bean>

    <!-- 配置Shiro的bean后处理器：用来初始化Shiro的bean在spring中-->
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>
    <!-- 开启Shiro注解 -->
    <!-- Enable Shiro Annotations for Spring-configured beans.
    Only run after -->
    <!-- the lifecycleBeanProcessor has run:
    depends-on：当前bean初始化时，必须依赖于指定的bean，（指定的
    bean必须先初始化）
    下面的两个bean配置：传统的aop编程：增强、切点、切面
    -->
    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
    depends-on="lifecycleBeanPostProcessor"/>
    <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
        <!-- 必须注入安全管理器 -->
        <property name="securityManager" ref="securityManager" />
    </bean>

    <!-- shiro整合echcache的缓存配置 -->
    <!-- 配置Shiro的平台缓存管理 -->
    <bean id="shiroCacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
        <!-- 注入ehcache的对象 -->
        <property name="cacheManager" ref="ehCacheManager" />
    </bean>

</beans>
```

##### 4.在reaml对象中指定缓存权限的数据的区域
``` java
package me.xueyao.project.realms;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import me.xueyao.project.dao.system.PermissionRepository;
import me.xueyao.project.dao.system.RoleRepository;
import me.xueyao.project.dao.system.UserRepository;
import me.xueyao.project.domain.system.Permission;
import me.xueyao.project.domain.system.Role;
import me.xueyao.project.domain.system.User;

@Component("projectRealm")
public class ProjectRealm extends AuthorizingRealm{

    //只需要向父类注入缓存域即可
    //认证缓存区域
    @Value("project_realm_authentication_cache")
    //方法注入按照参数注入和方法名无关
    public void setSuperAuthenticationCacheName(String authenticationCacheName) {
        super.setAuthenticationCacheName(authenticationCacheName);
    }

    //授权缓存区域
    @Value("project_realm_authorization_cache")
    public void setSuperauthorizationCacheName(String authorizationCacheName){
        super.setAuthorizationCacheName(authorizationCacheName);
    }
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private RoleRepository roleRepository;

    @Autowired
    private PermissionRepository permissionRepository;

    //提供授权数据
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        // TODO Auto-generated method stub
        System.out.println("授权数据提供中...");
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        //写死安全数据
        //添加功能权限：底层是set集合
        //authorizationInfo.addStringPermission("courier:list");
        //添加角色权限
        //authorizationInfo.addRole("base");

        //2.根据登录用户动态获取授权数据 
        //方法一
        //User user = (User)SecurityUtils.getSubject().getPrincipal();
        //方法二
        User user = (User)principals.getPrimaryPrincipal();
        //判断是否是管理员
        if ("admin".equals(user.getUsername())) {

            for (Role role :roleRepository.findAll()) {
                //添加角色
                authorizationInfo.addRole(role.getKeyword());
            }

            for (Permission permission : permissionRepository.findAll()) {
                //添加功能权限
                authorizationInfo.addStringPermission(permission.getKeyword());
            }
        } else {
            //不是管理员
            for (Role role : roleRepository.findByUsers(user)) {
                //添加角色
                authorizationInfo.addRole(role.getKeyword());

                //添加角色具有的权限
                for (Permission permission : role.getPermissions()) { //此处涉及持久态问题，只有持久态后才可以查询 
                    authorizationInfo.addStringPermission(permission.getKeyword());
                }
            }
        }
        return authorizationInfo;
    }

    //提供认证数据
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        // 目标：根据用户名查询用户对象，封装成认证信息对象，返回交给安全管理器
        //用户名
        String username = ((UsernamePasswordToken)token).getUsername();
        //查询数据库：根据用户查询 对象
        User user = userRepository.findByUsername(username);
        //判断
        if (null == user) {
            //如果返回null，安全管理器认为用户名不存在
            return null;
        } else {
            //用户名存在，将用户信息封装到认证信息对象中，交给安全管理器
            /**
             * @param1 principal 负责人，就是用户对象，以后会保存到session中
             * @param2 credential 凭证，这里就是真实的密码
             * @param3 realm的对象的一个唯一的名字，类似uuid，底层就是一个随机数
             */
            SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(user, user.getPassword(), super.getName());
            return authenticationInfo;
        }
    }

}
```