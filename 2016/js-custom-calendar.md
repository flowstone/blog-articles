---
title: "Ajax自定义日历"
date: 2016-09-27 22:19:51
draft: false
categories: ["JS"]
---

## 需求分析

在一些购物网站中，都会有促销活动，这些活动都在日历上标注出来，如何通过Ajax让日历

通过读取数据库中的信息，正确的把促销活动标注在日历上，本文通过自定义日历来实现这个问题。

## 技术难点

* 日历的布局

* 日历的初始化

* 日历的动态变化

* 日历的促销定制



## 实现方法

1、 先创建一个固定的日历，效果如下

![自定义日历初始效果图](https://ueyao.github.io/image-hosting/blog/2016/201160927_232635.png)

html代码如下

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style type="text/css">
        * {margin: 0; padding: 0;}
        body {font-size: 13px;}
        .calendar {width: 330px; margin: 0 auto;}
        .calendar .title {
            position: relative;
            width: 100%;
            height: 30px;
            line-height: 30px;
            background: #17a4eb;
        }
        .title div {position: absolute;}
        .prev {left: 10px; }
        .now {left: 40%;}
        .next {right: 10px;}
        input {height: 30px; width: 300px; margin: 100px 475px 0px;}
        table {width: 100%; border-collapse: collapse;}
        table th {border: 1px solid #ccc;}
        table td {text-align: center; border: 1px solid #ccc;}
    </style>
</head>
<body>
    <input type="text">
    <div class="calendar">
        <div class="title">
            <div class="prev">
                <span>08</span>月
            </div>
            <div class="now">
                <span>2016</span>年
                <span>09</span>月
            </div>
            <div class="next">
                <span>10</span>月
            </div>
        </div>
        <table>
            <!--星期部分-->
            <thead>
            <tr>
                <th>日</th>
                <th>一</th>
                <th>二</th>
                <th>三</th>
                <th>四</th>
                <th>五</th>
                <th>六</th>
            </tr>
            </thead>
            <!--日期部分-->
            <tbody>
            <tr>
                <td>1</td>
                <td>1</td>
                <td>1</td>
                <td>1</td>
                <td>1</td>
                <td>1</td>
                <td>1</td>
            </tr>
            <tr>
                <td>1</td>
                <td>1</td>
                <td>1</td>
                <td>1</td>
                <td>1</td>
                <td>1</td>
                <td>1</td>
            </tr>
            </tbody>
        </table>
    </div>
</body>
</html>
```

当创建固定日历后，把日历的html部分注释掉(title和table)，保留css部分

2、 通过javascript动态生成日历

``` javascript
window.onload = function () {
       var oInput = document.getElementsByTagName('input')[0];
       var oCalendar = document.getElementById('calendar');
//           console.log(oCalendar);
       var oDate = new Date();
       var year = oDate.getFullYear();
       var month = oDate.getMonth()+1;

       //日期td
       var oTds = oCalendar.getElementsByTagName('td');

       var flag = false;
       oInput.onfocus = function () {
           showDate(year,month);
       }
       //显示日历
       function showDate(year,month) {
           if (false == flag) {

               var oTitle = document.createElement('div');
               oTitle.className = 'title';

               oTitle.innerHTML = '<div class="prev"> <span>'+(month-1)+'</span>月 </div> ' +
                       '<div class="now"> <span>'+year+'</span>年 <span>'+month+'</span>月 </div> ' +
                       '<div class="next"> <span>'+(month+1)+'</span>月 </div>';

               oCalendar.appendChild(oTitle);

               //月份span
                ospans = oCalendar.getElementsByTagName('span');
              // console.log(ospans);
                prevMonth = ospans[0];
                nextMonth = ospans[3];
                nowMonth = ospans[2];
                nowYear = ospans[1];

               //创建星期
               var otable = document.createElement('table');
               var othead = document.createElement('thead');
               var otr = document.createElement('tr');
               var arr = ['日','一','二','三','四','五','六'];
               for (var i=0; i<arr.length; i++) {
                   //创建th
                   var oth = document.createElement('th');
                   oth.innerHTML = arr[i];
                   otr.appendChild(oth);
               }
               othead.appendChild(otr);
               otable.appendChild(othead);
               oCalendar.appendChild(otable);


               //先获得当前月有多少天
               if (1 == month || 3 == month || 5 == month || 7 == month || 8 == month || 10 == month || 12 == month) {
                   var dayNum = 31;
               } else if (4 == month || 6 == month || 9 == month || 11 == month) {
                   var dayNum = 30;
               } else if (2 == month && isLeapYear(year)) {
                   var dayNum = 29;
               } else {
                   var dayNum = 28;
               }
               //确定当前月的1号是星期几
               oDate.setFullYear(year);
               oDate.setMonth(month-1);
               oDate.setDate(1);

               //日期
               var otbody = document.createElement('tbody');
               for (var i=0; i<6; i++) {
                   var oTr = document.createElement('tr');
                   //每行里面有7列
                   for (var j=0; j<7; j++) {
                       var oTd = document.createElement('td');
                       //oTd.innerHTML = 1;
                       oTr.appendChild(oTd);
                   }
                   otbody.appendChild(oTr);
               }
               otable.appendChild(otbody);

               //获得今天1号对应的是星期几
               var week = oDate.getDay();
               var oTds = oCalendar.getElementsByTagName('td');
               //alert(week);
               for (var i=0; i<dayNum; i++) {
                   oTds[i+week].innerHTML = i+1;
               }

               //如果当前月month 是12或者1
               if (1 == month) {
                   prevMonth.innerHTML = 12;
               } else if (12 == month) {
                   nextMonth.innerHTML = 1;
               }
               //让当前日期显示红色、后面的显示蓝色
               showColor();
               //给左右月份绑定点击事件
                monthEvent();
               //给所有的td绑定点击事件
               tdClick();
               //判断最后一行是否全为空
               lastTr();
               //获得促销信息
               getPromotion();
               flag = true;
           }
       }
       //最后一行如果全部为空就将其隐藏
       function lastTr() {
           //查找最后一行的所有td
           var flag = true;
           for (var i=35; i<42; i++) {
               if (oTds[i].innerHTML != '') {
                   //有任何一个td不为空就设置为false
                   flag = false;
               }
           }
           //全部是空的
           if (flag) {
               for (var i=35; i<42; i++) {
                   oTds[i].style.display = 'none';
               }
           }
       }
       //给所有的td绑定点击事件
       function tdClick() {
           for (var i=0; i<oTds.length; i++) {
               oTds[i].onclick = function() {
                    if ('red' == this.className ||'blue' == this.className) {
                        var year = nowYear.innerHTML;
                        var month = nowMonth.innerHTML;
                        var date = this.innerHTML;
                        oInput.value = year +'-'+month+'-'+date;
                        flag = false;
                        oCalendar.innerHTML = '';
                    } else {
                        alert('您只能选择红色或蓝色区域');
                    }
               }
           }
       }
       //当前日期显示红色、后面的显示蓝色
       function showColor() {
           //当前的日期
           var oday = new Date().getDate();
           for (var i=0; i<oTds.length; i++) {
               if (oday == oTds[i].innerHTML) {
                   oTds[i].className = 'red';
                   var oindex = i;
               }
           }
           for (var j=oindex+1; j<oTds.length; j++) {
               oTds[j].className = 'blue';
           }
       }

       //给左右月份绑定点击事件
       function monthEvent() {
           //向左的月份div
           prevMonth.parentNode.onclick = function () {
               //alert('向左');
               flag = false;
               oCalendar.innerHTML = '';
               if (12 == prevMonth.innerHTML) {
                   showDate(year -= 1, 12);
               } else {
                   showDate(year,parseInt(prevMonth.innerHTML));
               }
           }
           //向左的月份div
           nextMonth.parentNode.onclick = function () {
               //alert('向右');
               flag = false;
               oCalendar.innerHTML = '';
               if (1 == nextMonth.innerHTML) {
                   showDate(year+=1,1);
               } else {
                   showDate(year,parseInt(nextMonth.innerHTML));
               }
           }
       }
       //判断是否是闰年
       function isLeapYear(year) {
           if (0 == year%100 && 0 == year%400) {
               return true;
           }else if (year%100 != 0 && year%4 ==0) {
               return true;
           } else {
               return false;
           }
       }
}
```

3、 从服务器获取促销的信息并在日历中显示

``` javascript
//从服务器获取促销信息
  function getPromotion() {
       $.request({
          method:"post", //获取方式
           url:"promotion.php", //从哪个文件中获取
           data:"",  //是否传递数据
           callback:function (res) {
               eval("var obj="+res);
               if (obj.status) {
                   var dates = obj.dates;
                   for (var i=0; i<dates.length; i++) {
                       for (var j=0; j<oTds.length; j++) {
                           if (oTds[j].innerHTML == dates[i]) {
                               oTds[j].innerHTML += "促销";
                               oTds[j].style.background = 'red';
                           }
                       }
                   }
               }
           }
       });
  }
```

$.request()是封装在js里的Ajax方法,代码如下：

``` javascript
var $ = {
    request:function(obj){
        var xhr;
        try {
            //主流浏览器里面的ajax对象
            xhr = new XMLHttpRequest();
        } catch(e) {
            //IE低版本的浏览器
            xhr = new ActiveXObject("Microsoft.XMLHTTP");
        }

        //建立和服务器的连接
        if (obj.method == 'get') {
            xhr.open(obj.method,obj.url+'?'+obj.data+'&'+Math.random(),true);
            xhr.send();
        } else if (obj.method == 'post') {
            xhr.open(obj.method,obj.url,true);
            xhr.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
            xhr.send(obj.data);
        }
        //监视服务器的处理状态
        xhr.onreadystatechange = function(){
            if (4 == xhr.readyState && 200 == xhr.status) {
                //说明请求成功了，输出服务器返回的数据
                obj.callback(xhr.responseText);
            }
        }
    }
}
```

promotion.php

```php
<?php
    $data['status'] = 1;
    //促销时间
    $data['dates'] = array(28,29,30);

    echo json_encode($data);
```

最终效果图如下，样式不是很美观

![自定义日历最终效果图](https://ueyao.github.io/image-hosting/blog/2016/20160927_232715.png)


[代码托管于GitHub](https://github.com/flowstone/2016-Code/tree/master/day075AJax)