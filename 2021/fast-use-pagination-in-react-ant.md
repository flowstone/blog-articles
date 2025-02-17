---
title: "React Ant框架中分页的快速使用"
date: 2021-01-01 13:00:01
draft: false
categories: ["JS"]
---

### 使用环境
* React
* Ant Design
* SpringBoot
* Spring Data JPA

### 前端页面

首先，我们要查看Ant官方文档，一般涉及到分页的页面都是列表查询，在表格组件中找到API，可以从API中看到使用方法，查看文档后，编写代码如下
``` js
<Pagination
    //默认当前页是接口返回的当前页
    defaultCurrent={responseParam.pageNum}
    //每页条数 取得是接口返回的每页条数
    pageSize={responseParam.pageSize}
    //总条数  取得是接口返回的总条数
    total={responseParam.count}
    //指定每页可以显示多少条	
    pageSizeOptions={["2","4","6"]}
    //是否展示 pageSize 切换器
    showSizeChanger
    /**页码改变的回调，参数是改变后的页码及每页条数
    * @param page 改变后页码
    * @param pageSize 每页条数
    */
    onChange={(page: any, pageSize: any) => {
      console.log('page = ', page, pageSize);
      setRequestParam(param => ({
        ...param,
        //修改页码
        pageNum: page,
        //修改每页条数
        pageSize: pageSize
      }));
    }}/>
```

requestParam定义格式如下:
``` js
const [requestParam, setRequestParam] = useState({
   //给页码默认值
    pageNum: 1,
    //每页条数默认值
    pageSize: 10
  });
```

当每次我们修改requestParam的值时，就会重新调用接口渲染页面，如下所示:
``` js
useEffect(() => {
    //调用查询接口
    loadData();
    
  }, [requestParam]);
```

### 后端接口

接口的请求参数，主要包含pageNum和pageSize,如下所示:
``` java
@GetMapping("/list/{pageNum}/{pageSize}")
public R list(@PathVariable("pageNum") Integer pageNum,
              @PathVariable("pageSize") Integer pageSize) {
    
    return smsTemplateService.list(PageRequest.of(pageNum-1, pageSize));
}
```
PageRequest.of()静态方法会返回一个PageRequest对象，为什么pageNum-1？查看源码如下:
``` java
/**
 * Creates a new unsorted {@link PageRequest}.
 *
 * @param page zero-based page index, must not be negative.
 * @param size the size of the page to be returned, must be greater than 0.
 * @since 2.0
 */
public static PageRequest of(int page, int size) {
	return of(page, size, Sort.unsorted());
}
```
从上述静态方法中，可以得知，pageNum是从0开始计算，故pageNum-1,下面是接口具体实现

``` java
/**
* @param pageable  此处pageable为接口定义，前面PageRequest是它的一种实现
*/
@Override
public R list(Pageable pageable) {
    Page<SmsTemplate> smsTemplates =  smsTemplateRepository.findAll(pageable);
    
    List<SmsTemplateVo> smsTemplateVos = smsTemplates.getContent().stream().map(smsTemplate -> {
        SmsTemplateVo smsTemplateVo = new SmsTemplateVo();
        BeanUtils.copyProperties(smsTemplate, smsTemplateVo);
        return smsTemplateVo;
    }).collect(Collectors.toList());
    return R.ofSuccess("查询成功", new PageResult<>(smsTemplateVos, smsTemplates));
}
```

接口返回的结果的示例，如下所示
``` json
{
    "code":200,
    "message":"查询成功",
    "data":{
        "pageNum":1,
        "pageSize":2,
        "count":6,
        "result":[
            {
                "id":2,
                "sendAccount":"13130000000",
                "smsSign":"方方[已审核]",
                "type":1,
                "content":"你的短信验证码是123456,请勿将验证码泄露给他人。",
                "reason":"测试111",
                "notice":1,
                "createTime":"2021-01-15 14:02:55",
                "updateTime":"2021-01-15 14:02:55",
                "creator":"xiaoming",
                "updater":null
            }
        ]
    }
}
```




