---
title: "Javamail简单使用案例"
date: 2017-09-13 12:02:47
draft: false
categories: "Java"
---

## 邮件开发环境搭建

### 邮件服务器
* 易邮邮件服务器
* 配置如下
![易邮邮件服务器](https://ueyao.github.io/image-hosting/blog/2017/09/2017-09-13_085915.png)

![易邮邮件服务器配置](https://ueyao.github.io/image-hosting/blog/2017/09/2017-09-13_191012.png)


### 邮件客户端
* Foxmail
* 配置如下
![Foxmail配置](https://ueyao.github.io/image-hosting/blog/2017/09/2017-09-13_192046.png)

### 使用Javamail发送邮件
1、下载

  * javamail-samples.zip
  * javax.mail.jar

官网地址http://www.oracle.com/technetwork/java/javamail/index.html

2、使用javamail

发送到本地的邮件中

``` java
package org.xueyao.email;
import java.util.Date;
import java.util.Properties;
import javax.mail.Message;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
public class MailUtils {
    /**
     * 发送邮件
     * @param to   接收邮件的地址
     * @param subject  邮件主题
     * @param msgText   邮件内容
     */
    public static void send(String to, String subject, String msgText) {
        //发邮件的地址
        String from = "admin@flowstone.com"; 
        //邮件发送服务器地址
        String host = "localhost"; 
        //是否开启debug模式
        boolean debug = true; 
        // 设置发送邮件的配置信息
        Properties props = new Properties();
        props.put("mail.smtp.host", host);
        if (debug) {
            props.put("mail.debug", debug);
        }
        //邮件会话
        Session session = Session.getInstance(props, null);
        session.setDebug(debug);
        try {
            //创建邮件
            MimeMessage msg = new MimeMessage(session);
            msg.setFrom(new InternetAddress(from));
            InternetAddress[] address = { new InternetAddress(to) };
            msg.setRecipients(Message.RecipientType.TO, address);
            //设置主题
            msg.setSubject(subject);
            //设置发送时间
            msg.setSentDate(new Date());
            // If the desired charset is known, you can use
            // setText(text, charset)
            //设置邮件的内容
            msg.setText(msgText);
            //发送邮件
            Transport.send(msg);
        } catch (Exception mex) {
            mex.printStackTrace();
        }
    }
    public static void main(String[] args) {
        String to = "test02@flowstone.com";
        String subject = "如何学习?";
        String msgText = "解决学习困扰,就是天天晚上熬夜学习";
        MailUtils.send(to, subject, msgText);

    }

}
```
效果图

![效果图](https://ueyao.github.io/image-hosting/blog/2017/09/2017-09-13_193447.png)

通过QQ邮箱发送
``` java
package org.xueyao.email;
import java.util.Date;
import java.util.Properties;
import javax.mail.Message;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
public class MailQQUtils {
    /**
     * 发送邮件
     * @param to   接收邮件的地址
     * @param subject  邮件主题
     * @param msgText   邮件内容
     */
    public static void send(String to, String subject, String msgText) {
        //发邮件的地址
        String from = "931330220@qq.com"; 
        String password = "授权码";
        //邮件发送服务器地址
        String host = "smtp.qq.com"; 
        //是否开启debug模式
        boolean debug = true; 
        // 设置发送邮件的配置信息
        Properties props = new Properties();
        props.put("mail.smtp.host", host);

        if (debug) {
            props.put("mail.debug", debug);
        }
        //添加auth认证
        props.put("mail.smtp.auth", "true");
        props.put("mail.smtp.port", "587");

        //邮件会话
        Session session = Session.getInstance(props, null);
        session.setDebug(debug);
        try {
            //创建邮件
            MimeMessage msg = new MimeMessage(session);
            msg.setFrom(new InternetAddress(from));
            InternetAddress[] address = { new InternetAddress(to) };
            msg.setRecipients(Message.RecipientType.TO, address);
            //设置主题
            msg.setSubject(subject);
            //设置发送时间
            msg.setSentDate(new Date());
            //设置邮件的内容
            msg.setText(msgText);
            //发送邮件
            Transport.send(msg,from,password);
        } catch (Exception mex) {
            mex.printStackTrace();
        }
    }
    public static void main(String[] args) {
        String to = "收件邮箱";
        String subject = "如何学习?";
        String msgText = "解决学习困扰,就是天天晚上熬夜学习";
        MailQQUtils.send(to, subject, msgText);

    }
}
```

效果图

![](https://ueyao.github.io/image-hosting/blog/2017/09/2017-09-13_211125.png)