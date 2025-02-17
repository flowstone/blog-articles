---
title: "JS特效-轮播图"
date: 2016-09-17 22:17:35
draft: false
categories: ["JS"]
---

## 效果如下
![轮播图](https://ueyao.github.io/image-hosting/blog/2016/601849-20160917220146523-52456133.png)

## 功能分析
1. 每隔1秒换一张图片
2. 鼠标移入停止切换、鼠标离开继续切换
3. 鼠标移入到数字上面的时候,显示和数字对应的图片,并且停止切换,被选中的数字,背景显示橙色
4. 鼠标离开数字,从该数字后面继续显示


## 代码如下
``` js
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title></title>
    <link rel="stylesheet" href="">
    <style type="text/css">
        div,
        img,
        ul,
        li {
            padding: 0px;
            margin: 0px;
        }
        .content {
            width: 480px;
            height: 300px;
            border: 1px solid red;
            margin: 100px auto;
        }
        img {
            width: 100%;
            height: 100%;
            padding-bottom: 10px;
        }
        ul li {
            list-style: none;
            float: left;
            border: 1px solid orange;
            height: 30px;
            width: 58px;
            text-align: center;
            line-height: 30px;
        }
    </style>
</head>
<body>
    <div class="content">
        <img src="./img/1.jpg" alt="">
        <ul>
            <li>1</li>
            <li>2</li>
            <li>3</li>
            <li>4</li>
            <li>5</li>
            <li>6</li>
            <li>7</li>
            <li>8</li>
        </ul>
    </div>
    <script type="text/javascript">
        var oImg = document.getElementsByTagName('img')[0];
        var count = 1;
        function changePic(){
            count ++;
            if (count > 8) {
                count = 1;
            }
            oImg.src = 'img/'+count+ '.jpg';
        }
        var interID = setInterval(changePic, 1000);
        //鼠标移入停止播放
        oImg.onmouseover = function(){
            clearInterval(interID);

        }
        //鼠标移出继续播放
        oImg.onmouseout = function(){
                //console.log(interID);
                clearInterval(interID);
                interID = setInterval(changePic, 1000);

        }
        //鼠标移入到数字上的时候,显示对应的图片
        var oLi = document.getElementsByTagName('li');
        //console.log(oLi.length);
        for (var num = 0; num < oLi.length; num++) {
            //给每个li标签增加属性，保存当前的索引位置
            oLi[num].index = num;
            //移到到数字上,停止播放
            oLi[num].onmouseover = function(){
                //停止播放
                clearInterval(interID);
                this.style.background = 'orange';
                count = this.index;
                //调用循环播放图片方法
                changePic();
            }
            //移出时,继续从停止的地方播放
            oLi[num].onmouseout = function(){
                clearInterval(interID);
                interID = setInterval(changePic, 1000);
                this.style.background = 'white';
                count = this.index;
                changePic();
            }
        }
    </script>
</body>
```