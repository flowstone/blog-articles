---
title: "AngularJS动态特效之验证码按钮倒计时"
date: 2017-11-11 12:08:39
draft: false
categories: "JS"
---
## 功能需求

当用户注册或者找回密码时，输入注册的手机号发送验证码到手机中，点击发送验证码按钮倒计时这个功能是如何实现呢？

## 效果如下

![](https://ueyao.github.io/image-hosting/blog/2017/11/2017-11-11_204515.png)

## 表单代码如下

``` html
<div class="signup" ng-app="signupApp" ng-controller="signupCtrl">
    <div class="col-md-9 signupbox">
        <form id="signupForm" action="customer_regist.action" method="post" class="form col-md-6">
            <div class="form-group">
                <label for="inputaccount" class="col-sm-3 control-label">
                    <b>*</b>验证码</label>
                <div class="col-sm-5">
                    <input type="text" name="checkcode" class="form-control" id="inputaccount" placeholder="请输入验证码">
                </div>
                <div class="col-sm-3 song">
                    <button type="button" id="checkCode" class="btn btn-default" ng-bind="checkcodemsg" ng-click="getCheckCode()">获取验证码</button>
                </div>
            </div>
        <form>
    </div>
</div>
```

## JS代码如下

``` javascript
<!--验证码倒计时-->
<script type="text/javascript">
    angular.module("signupApp", [])
        .controller("signupCtrl", ["$scope", function ($scope) {
            //按钮初始化的名字
            $scope.checkcodemsg = "获取验证码";
            //倒计时变量，默认60秒
            var second = 5;
            //定时器对象
            var secondInterval = undefined;
            //是否允许发送验证码的标识 
            var enableFlag = true;
            //获取验证码点击事件
            $scope.getCheckCode = function () {
                //允许发送验证码
                if (enableFlag) {
                        //发送验证码的标识为false，在重新发送时间里不允许再发送
                        enableFlag = false;
                        //禁止按钮
                        $("#checkCode").attr("disabled","true");
                        //开启定时器
                            secondInterval = window.setInterval(function () {
                            //如果时间为0秒时
                            if (second === 0) {
                                //重新初始化按钮
                                $scope.checkcodemsg = "获取验证码";
                                //强制更新视图
                                $scope.$digest();
                                //清除定时器
                                window.clearInterval(secondInterval);
                                //清空变量
                                secondInterval = undefined;
                                //重置秒数
                                second = 5;
                                //重置验证码标识 
                                enableFlag = true;
                                //按钮可用
                                $("#checkCode").removeAttr("disabled");
                            } else {
                                //发送验证码
                                $scope.checkcodemsg = "重新发送("+second + ')秒';
                                //强制更新视图
                                $scope.$digest();
                                //秒数倒计时
                                second--;
                            }
                        }, 1000);
                } 
            };
        }]);
</script>
```

