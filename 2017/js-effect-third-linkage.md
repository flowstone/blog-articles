---
title: "AJAX省市县三级联动的实现"
date: 2017-09-14 12:04:08
draft: false
categories: ["JS"]
---

省市县数据

本例子中省市县数据保存在MySQL数据库中,部分数据截图如下:

![省市县数据](https://ueyao.github.io/image-hosting/blog/2017/09/2017-09-14_173943.png)



从数据库中读取数据

1、导入需要的jar包

![所需jar包](https://ueyao.github.io/image-hosting/blog/2017/09/2017-09-14_174657.png)

2、连接池配置文件

``` xml
<c3p0-config>
    <!-- 默认配置，如果没有指定则使用这个配置 -->
    <default-config>
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://127.0.0.1:3306/test</property>
        <property name="user">root</property>
        <property name="password">数据库密码</property>
        <property name="checkoutTimeout">30000</property>
        <property name="idleConnectionTestPeriod">30</property>
        <property name="initialPoolSize">10</property>
        <property name="maxIdleTime">30</property>
        <property name="maxPoolSize">100</property>
        <property name="minPoolSize">10</property>
        <property name="maxStatements">200</property>
        <user-overrides user="test-user">
            <property name="maxPoolSize">10</property>
            <property name="minPoolSize">1</property>
            <property name="maxStatements">0</property>
        </user-overrides>
    </default-config>
</c3p0-config>
```

3、JDBCUtils工具类文件

通用JDBCUtils工具类文件,使用时直接引入

``` java
package org.xueyao.ajax.utils;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import javax.sql.DataSource;
import org.apache.tomcat.jni.Thread;
import com.mchange.v2.c3p0.ComboPooledDataSource;
public class JDBCUtils {

    private static ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();

    //使用ThreadLocal存取删链接对象
    private static ThreadLocal<Connection> local = new ThreadLocal<>();


    public static Connection getConnection() throws SQLException{
        return comboPooledDataSource.getConnection();
    }

    public static DataSource getDataSource(){
        return comboPooledDataSource;
    }
    //从ThreadLocal获取链接的方法
    public static Connection getCurrentConnection() throws SQLException{
        //先从ThreadLocal获取中
        Connection connection = local.get();
        if(connection != null){
            System.out.println("从local获取数据："+connection);
            return connection;
        }else{
            //如果没有，在从链接池获取，存入ThreadLocal中
            Connection conn = comboPooledDataSource.getConnection();
            System.out.println("从连接池获取数据："+conn);
            local.set(conn);
            return conn;
        }
    }
    //开启事务的方法
    public static void startTransaction() throws SQLException{
        Connection connection = getCurrentConnection();
        System.out.println(connection);
        connection.setAutoCommit(false);
    }
    //提交事务的方法
    public static void commit() throws SQLException{
        Connection connection = getCurrentConnection();
        System.out.println(connection);
        connection.commit();
    }

    //回滚事务的方法
    public static void rollback() throws SQLException{
        Connection connection = getCurrentConnection();
        System.out.println(connection);
        connection.rollback();
    }

    //释放资源的方法
    public static void close() throws SQLException{
        Connection connection = getCurrentConnection();
        System.out.println(connection);
        local.remove();
        connection.close();
    }

}
```

4、创建JavaBean文件

``` java
package org.xueyao.ajax.domain;
public class Province {
    private int codeid;
    private int parentid;
    private String cityName;
    public int getCodeid() {
        return codeid;
    }
    public void setCodeid(int codeid) {
        this.codeid = codeid;
    }
    public int getParentid() {
        return parentid;
    }
    public void setParentid(int parentid) {
        this.parentid = parentid;
    }
    public String getCityName() {
        return cityName;
    }
    public void setCityName(String cityName) {
        this.cityName = cityName;
    }
    @Override
    public String toString() {
        return "Province [codeid=" + codeid + ", parentid=" + parentid
                + ", cityName=" + cityName + "]";
    }
}
```

5、创建Servlet文件,获取省市县数据

``` java
package org.xueyao.ajax.linkage;
import java.io.IOException;
import java.sql.SQLException;
import java.util.List;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanListHandler;
import org.xueyao.ajax.domain.Province;
import org.xueyao.ajax.utils.JDBCUtils;
import flexjson.JSONSerializer;
public class GetDataServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    protected void doGet(HttpServletRequest request,
            HttpServletResponse response) throws ServletException, IOException {
        int parentid = Integer.parseInt(request.getParameter("parentid"));
        String sql = "SELECT * FROM province WHERE parentid=?";
        QueryRunner qr = new QueryRunner(JDBCUtils.getDataSource());
        try {
            List<Province> list = qr.query(sql, new BeanListHandler<Province>(Province.class), parentid);
            //System.out.println(list);

            //将list集合转换成json格式字符串
            JSONSerializer jsonSerializer = new JSONSerializer();
            String serialize = jsonSerializer.serialize(list);
            //System.out.println(serialize);

            //向页面输入字符串数据
            response.setContentType("text/html;charset=utf-8");
            response.getWriter().write(serialize);
        } catch (SQLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
    protected void doPost(HttpServletRequest request,
            HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}
```

6、从页面显示三级联动

``` jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c"  uri="http://java.sun.com/jsp/jstl/core"%>
<c:set var="root" value="${pageContext.request.contextPath }"></c:set>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>省市县页面</title>
<script type="text/javascript" src="js/jquery-1.8.3.js"></script>
<script type="text/javascript">
$(function() {
    //获得三个下拉菜单元素
    var $pro = $("#province");
    var $city = $("#city");
    var $area = $("#area");

    $.get(
        "${root}/getData?parentid=0",
        function(data) {
            //alert(data);
            //遍历数据
            $(data).each(function(){
                //创建option标签,并追加到省级菜单中
                $pro.append("<option value='"+this.codeid+"'>"+this.cityName+"</option>");
            }); 
        },"json");
    //省级菜单选择事件
    $pro.change(function(){
        //清空市县菜单中数据,保留第一个属性
        $city.prop("length",1);
        $area.prop("length",1);
        $.get(
            "${root}/getData?parentid="+this.value,
            function(data) {
                $(data).each(function() {
                    $city.append("<option value='"+this.codeid+"'>"+this.cityName+"</option>");
                });
            },"json");
    });
    //市级菜单选择事件
    $city.change(function(){
        //清空县菜单中数据,保留第一个属性
        $area.prop("length",1);
        $.get(
                "${root}/getData?parentid="+this.value,
                function(data) {
                    $(data).each(function(){
                        $area.append("<option value='"+this.codeid+"'>"+this.cityName+"</option>");
                    });
                },"json");              
    });

});
</script>
</head>
<body>
    <center>
        <select id="province" name="province">
            <option value="none">--请选择省--</option>
        </select>
        <select id="city" name="city">
            <option value="none">--请选择市--</option>
        </select>
        <select id="area" name="area">
            <option value="none">--请选择县--</option>
        </select>
    </center>
</body>
</html>
```

上面的javascript代码可以优化成这样

``` javascript
function loadData(value, ele){
    $.get(
            "${root}/getData?parentid="+value,
            function(data) {
                //遍历数据
                $(data).each(function(){
                    //创建option标签,追加到菜单中
                    ele.append("<option value='"+this.codeid+"'>"+this.cityName+"</option>");
                }); 
            },"json");
}
$(function() {
    var $pro = $("#province");
    //加载省和直辖市数据
    loadData(0,$pro);
    //加载市
    $("#city,#area").change(function () {
        // 清空原来的数据
        $(this).nextAll().prop("length",1);
        loadData(this.value, $(this).next());
    });

});
```

效果图如下:

![ajax三级联动效果图](https://ueyao.github.io/image-hosting/blog/2017/09/2017-09-14_172045.gif)



[代码托管于GitHub](https://github.com/flowstone/blog-example-code/tree/master/jsp-ajax-three-level-linkage-effect-example)