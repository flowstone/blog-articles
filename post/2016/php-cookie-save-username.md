---
title: "Cookie实例-保存用户登录时的用户名"
date: 2016-08-19 22:13:29
draft: false
categories: "PHP"
---


实现一个登录界面,记录用户名的信息,10小时内，不需要重新输入用户名的信息。(请使用Cookie完成)

界面代码如下
``` html
<!DOCTYPE html><html><head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title></title>
    <link rel="stylesheet" href=""></head><body>
    <form action="user.php?a=check" method='post'>
        u: <input type="text" name='username' value="<?php echo $username;?>"><br/>
        p: <input type="password" name='password'><br/>
        是否保存用户名 <input type="radio" value='yes' name='save'>保存10小时        <input type="radio" value='no' name='save'>不保存<br/>
        <input type="submit" value="登录">
    </form></body></html>
```
代码如下：
``` php
<?php
header('content-type:text/html;charset=utf-8');

$action = isset($_GET['a']) ? $_GET['a'] :'login';

if ($action == 'login') {
    $username = '';
    if (isset($_COOKIE['username'])){
        $username = $_COOKIE['username'];
    }
    require 'myLogin.html';
} elseif ($action == 'check') {

    $username = isset($_POST['username']) ? $_POST['username'] : '';
    $pwd = isset($_POST['password']) ? $_POST['password'] : '';
    $is_save = isset($_POST['save']) ? $_POST['save'] : '';

    if ($pwd == '123') {//默认密码为123
        if ($is_save == 'yes') {//如果点击保存用户名，则设置cookie
            //设置cookie,保存用户名
            setcookie('username',$username,time() + 30);
            $info = '恭喜登录成功，保存了你的用户名到cookie';
            require 'myLoginOk.html';
        }elseif ($is_save == 'no'){//点击不保存用户名，如果cookie中有数据，则删除cookie
            if(isset($_COOKIE['username'])){
                setcookie('username','',time() - 1);
                unset($_COOKIE['username']);
            }
            $info = '恭喜登录成功，删除你的用户名cookie';
            require 'myLoginOk.html';
        }else { //如果什么都没选择,则直接显示登陆成功
            $info = '恭喜登录成功';
            require 'myLoginOk.html';
        }
    }else{//密码不正确
        $info = '用户名,密码不正确';
        require 'myloginError.html';
    }
}
```

