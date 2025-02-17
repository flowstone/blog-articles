---
title: "JS小例子之二级联动"
date: 2017-08-26 11:30:20
draft: false
categories: ["JS"]
---

## 联动原理
当用户点击省级的下拉选项,选择所在省,下一个下拉选项里的选项,则变成用户选择省下的所有市的信息,不会出现其它省市的信息.


## 省市数据
把省市数据,保存在js文件中,以json形式保存,以便读取,下面代码使用部分数据,不影响效果
``` js
var china = [
    {
        "p_name": "吉林省",
        "p_id": "jl",
        "cities": [
            {
                "c_name": "长春",
                "c_id": "cc"
            },
            {
                "c_name": "四平",
                "c_id": "sp"
            },
            {
                "c_name": "通化",
                "c_id": "th"
            },
            {
                "c_name": "松原",
                "c_id": "sy"
            }
        ]
    },
    {
        "p_name": "辽宁省",
        "p_id": "ln",
        "cities": [
            {
                "c_name": "沈阳",
                "c_id": "sy"
            },
            {
                "c_name": "大连",
                "c_id": "dl"
            },
            {
                "c_name": "抚顺",
                "c_id": "fs"
            },
            {
                "c_name": "铁岭",
                "c_id": "tl"
            }
        ]

    },
{
    "p_name": "山东省",
    "p_id": "sd",
    "cities": [
        {
            "c_name": "济南",
            "c_id": "jn"
        },
        {
            "c_name": "青岛",
            "c_id": "qd"
        },
        {
            "c_name": "威海",
            "c_id": "wh"
        },
        {
            "c_name": "烟台",
            "c_id": "yt"
        }
    ]

},
{
    "p_name": "上海市",
    "p_id": "sh",
    "cities": [
        {
            "c_name": "闵行区",
            "c_id": "mh"
        },
        {
            "c_name": "徐汇区",
            "c_id": "xh"
        },
        {
            "c_name": "黄浦区",
            "c_id": "hp"
        },
        {
            "c_name": "浦东新区",
            "c_id": "pd"
        }
    ]

}

];
```

## 获取省市数据
通过js dom编程读取省市数据,把省市数据读取到对应的select选项中.

``` js
<!DOCTYPE html>
<html>
  <head>
    <title>二级联动</title>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8"> 

  </head>
  <body>
     <select id="province" name="province">
       <option value="none">--请选择省--</option>

    </select>

      <select id="city" name="city">
          <option value="none">--请选择市--</option>
      </select>
      <!-- 载入省市数据 -->   
      <script type="text/javascript" src="cities.js"></script>
      <script type="text/javascript" >
        //获得省级下拉框对象
        var province = document.getElementById("province");
        //遍历省市数据,并把里面省的数据追加到option选项中
        for (var i=0; i<china.length; i++) {
            var option = document.createElement("option");
            option.value = china[i].p_id;
            option.innerHTML = china[i].p_name;
            province.appendChild(option);
        }
          //省级下拉框发生改变事件
          province.onchange = function() {
            //获取当前点击对象的值
            var proid = this.value;
            var cities;
            //遍历省市数据,把省级下点击的那一个选项的值和省市数据中的
            //省级数据对比,如果相等,取出当前的市的数据
            for (var i=0; i<china.length; i++) {
                if (proid == china[i].p_id) {
                    cities = china[i].cities;
                }
            }
            //获得市级下拉框对象
            var city = document.getElementById("city");
            //每次点击省级后,市级初始化,避免高级重复追加
            city.innerHTML = "<option value='none'>--请选择市--</option>";
            //遍历市级数据,并取出市级数据,追加到市级对象中
            for (var i=0; i<cities.length; i++) {
                var option = document.createElement("option");
                option.innerHTML = cities[i].c_name;
                option.value = cities[i].c_id;
                city.appendChild(option);
            }
          }
      </script>
  </body> 
</html>
```

二级联动效果

![二级联动效果图](https://ueyao.github.io/image-hosting/blog/2017/2017-08-26-123422.gif)

[代码托管于GitHub](https://github.com/flowstone/blog-example-code/tree/master/2017-08-26-js-linkage-effect-example)