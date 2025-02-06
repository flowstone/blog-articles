---
title: "PHP实现分页功能"
date: 2016-07-16 22:01:31
draft: false
categories: "PHP"
---

#### 如何实现分页功能

1.代码如下
``` php
$total = mysql_num_rows($res1);             
//页面大小               
$pagesize = 3;           
//页面的值           
$page = isset($_GET['page']) ?$_GET['page'] : 1;                
//页面最大数目                  
$maxpage = ceil($total/$pagesize);             
//偏移量             
$offset = ($page-1)*$pagesize;               
$sql = "select * from mess_info limit {$offset},{$pagesize}";
```


``` html
<ul class="pager">
<li class="previous"><a href="list.php?page=1"> 首页</a></li>
<li><a href="list.php?page=<?php echo $page<=1 ? $page : $page-1;?>">上一页</a></li>
<li><a href="list.php?page=<?php echo $page>=$maxpage ? $maxpage : $page+1;?>">下一页</a></li>
<li class="next"><a href="list.php?page=<?php echo $maxpage;?>">末页 </a></li>
</ul>
```