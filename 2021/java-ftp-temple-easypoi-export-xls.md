---
title: "如何使用FTP中的模板文件和EasyPOI来导出Excle?"
date: 2021-07-25 13:31:32
draft: false
categories: ["Java"]
--- 

## 问题描述

因工作需要导出Excel文件，使用技术为EasyPOI,EasyPOI是一个非常好的导出文件工具，官网提供非常详细的使用文档，在项目中使用EasyPOI的模板导出功能，官方提供的示例代码中，模板的路径都是本地，我使用时也是把Excle模板文件放在本地，因为之前需要导出的地方，不是很多，模板文件放在本地也没有太大问题，但是由于现在需求变更，会有大量的模板需要导出，如果放在本地会造成项目容量变大。现在想把导出的模板保存在远程的FTP服务中，EasyPOI读取FTP的中模板文件生成Excle文件。

## 解决步骤

1、 查找解决方式
上网找了许多相关资料，官网上也没有找到解决方法，意外浏览了一篇文章，文章中提到了一句话，说EasyPOI读取模板文件，只支持读取本地模板文件，换句话来说，我只需要把FTP中的模板文件下载到本地指定路径，然后，就可以读取模板文件。

2、创建测试项目
创建一个SpringBoot项目，POM文件中引入需要的Jar包，如下
``` xml
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.7.5</version>
</dependency>

<dependency>
    <groupId>cn.afterturn</groupId>
    <artifactId>easypoi-spring-boot-starter</artifactId>
    <version>4.3.0</version>
</dependency>
<dependency>
    <groupId>commons-net</groupId>
    <artifactId>commons-net</artifactId>
    <version>3.6</version>
</dependency>
```
3、添加一些配置文件
``` yml
ftp:
  host: 192.168.2.66  # IP
  port: 21  # 端口
  username: root # 用户名
  password: 123456 # 密码
  mode: Passive # ftp模式
  remotePath: /root/export/ # ftp模板路径
  localPath: /Users/simonxue/Developer/Temp/ # 本地模板路径
template:
  employee: employee.xlsx #模板文件
```
3、 创建一个FTP下载方法，方法返回地址模板全路径名，如下所示
``` java
@Value("${ftp.host}")
private String host;
@Value("${ftp.port}")
private Integer port;
@Value("${ftp.username}")
private String username;
@Value("${ftp.password}")
private String password;
@Value("${ftp.mode}")
private String mode;
@Value("${ftp.remotePath}")
private String remotePath;
@Value("${ftp.localPath}")
private String localPath;

/**
    * 拷贝FTP中的文件到本地
    * @param fileName  ftp中的文件名
    * @return
    */
@SneakyThrows
public String localPathName(String fileName) {
    Ftp ftp = new Ftp(host, port, username, password, Charset.defaultCharset());
    ftp.setMode(FtpMode.Passive.name().equals(mode)? FtpMode.Passive: FtpMode.Active);
    String localName = localPath + fileName;
    ftp.download(remotePath, fileName, FileUtil.file(localName));
    ftp.close();
    return localName;
}
```

4、需要根据模板导出的地方，使用上面的方法,如下
``` java
@SneakyThrows
@Override
public void templateTest(HttpServletResponse response) {
    String localPathName = ftpUtil.localPathName(employeeTemplateName);
    response.setCharacterEncoding("utf-8");
    response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
    response.setHeader("Content-Disposition", "attachment;filename="+ IdUtil.getSnowflake(0,0).nextIdStr()+".xlsx");
    Map<String, Object> resultMap = new HashMap<>();
    List<Employee> employeeList = new ArrayList<>();
    Employee employee = new Employee();
    employee.setJobNumber("0001");
    employee.setUsername("小码农薛尧");
    employee.setPhone("1234567901");
    employee.setEmail("xueyao.me@gmail.com");
    employeeList.add(employee);
    List<Map<String, Object>> maps = employeeList.stream().map(a -> {
        Map<String, Object> map = BeanUtil.beanToMap(a);
        return map;
    }).collect(Collectors.toList());
    resultMap.put("employees", maps);
    // 此处需要指定模板路径
    TemplateExportParams params = new TemplateExportParams(localPathName, "test");
    Workbook workbook = ExcelExportUtil.exportExcel(params, resultMap);
    OutputStream outputStream = response.getOutputStream();
    workbook.write(outputStream);
    outputStream.close();
}
```
5、运行代码，生成的文件如下
![](https://ueyao.github.io/image-hosting/blog/2021/7/202107751450.png)

## 总结

EasyPOI不提供读取远程模板文件，但是我们可以通过其它方法来实现，下次导出Excle有格式样式改变，我们可以直接调整FTP中的模板文件就可以实现，不用重新部署项目。

**项目代码已存放在Github上**

[链接地址](https://github.com/flowstone/test-code-2021/tree/master/easypoi-export-template)