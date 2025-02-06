---
title: "Postman之token动态获取"
date: 2018-11-13 12:20:20
draft: false
categories: "Tool"
---
目前项目涉及PC及APP端接口共用问题，后台接口给登陆后的用户设置了一个token，接口调用时请求头的参数值必须要动态生成，为了解决这个问题，查看Postman API文档，配置了可以方便后端开发者的Tests脚本，如果你需要，请按下面方式配置。

### 用户登陆

用户登陆页面的请求头参数为固定不变，如图所示

![](https://ueyao.github.io/image-hosting/blog/2018/11/12/2018-11-12-5.36.35.png)

当填写正确的用户名和密码时，系统用返回如下图的数据，里面携带token的值,如图所示

![](https://ueyao.github.io/image-hosting/blog/2018/11/12/2018-11-12-5.49.47.png)

在用户登陆测试接口页面，在点击Tests，在里面添加下面代码，如图所示：

![](https://ueyao.github.io/image-hosting/blog/2018/11/12/2018-11-12-5.51.36.png)

``` javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
var data = JSON.parse(responseBody);

//key值
var key = '要加密的Key';
//current-timestamp
var currentTimestamp =  new Date().getTime().toString();
//nonce-str
var nonceStr = getStr(32);

function getStr(len){
    len = len || 32;
    var chars = '1234567890abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
    var maxPos = chars.length;
    var s = '';
    for (let i = 0; i < len; i++) {
        s += chars.charAt(Math.floor(Math.random() * maxPos));
    }
    return s;
}

//token
var token = data.data.token;
//拼接加密字符串
var signStr = token + currentTimestamp.substring(0,10) + nonceStr.substring(0,16) + key;
var CryptoJS = require('crypto-js');
var lpSign = CryptoJS.MD5(signStr).toString();

// 设置环境变量token，供后面的接口引用
pm.environment.set("token", data.data.token);
// 设置环境变量current-timestamp，供后面的接口引用
pm.environment.set("current-timestamp", currentTimestamp);
// 设置环境变量current-timestamp，供后面的接口引用
pm.environment.set("nonce-str", nonceStr);
// 设置环境变量current-timestamp，供后面的接口引用
pm.environment.set("lp-sign", lpSign);
```

配置环境变量，因为每个接口都涉及请求头，所有我们用不用Postman中的环境变量，来实现，请求头动态更新

## 步骤如下
1、 打开设置
![](https://ueyao.github.io/image-hosting/blog/2018/11/12/2018-11-12-5.55.52.png)

2、 添加新环境
![](https://ueyao.github.io/image-hosting/blog/2018/11/12/2018-11-12-5.59.36.png)

3、 添加环境变量
![](https://ueyao.github.io/image-hosting/blog/2018/11/12/2018-11-12-7.14.38.png)

保存环境变量，在调用其它接口时，先选择环境，如下图所示

![](https://ueyao.github.io/image-hosting/blog/2018/11/12/2018-11-12-7.18.57.png)

当Postman调用登陆接口时，会自动把缺少的环境变量值都添充完整，如下图所示

![](https://ueyao.github.io/image-hosting/blog/2018/11/12/2018-11-12-7.24.30.png)

调用其它接口时，请求头引用环境变量，具体语法如下图所示

![](https://ueyao.github.io/image-hosting/blog/2018/11/12/2018-11-12-8.05.02.png)

这样我们以后，调用其它接口，就不用每次都修改请求头数据，只要引用环境变量就完美解决问题。

注：老版本Postman有问题，本测试版本为6.5.2