---
title: "React中如何实现文件上传功能"
date: 2021-01-04 13:04:42
draft: false
categories: "JS"
---

### 场景描述
在前端页面中，经常会使用到文件上传功能，我们如何实现上传功能呢？

### 开发环境
* React ^16.12.0
* Ant Design ^5.0.12
* SpringBoot 2.3.0

### 页面实现文件上传
首先我们要用到Ant框架中自带的上传组件Upload，代码如下所示
``` tsx
//index.tsx
const props = {
    //组件的名字，调用接口时传递的名称
    name: 'file',
    //上传前的操作
    beforeUpload: (file: any, fileList: any) => {
      console.log('file, fileList', file, fileList);
      //上传的文件列表  
      setFileList(fileList);
      //返回false 代表暂停上传 需要我们自己实现
      return false;
    },
};

//下面代码是上传按钮
<Upload {...props}>
    <Button icon={<UploadOutlined />}>选择</Button>
 </Upload>
```
上述代码，可以实现一个上传功能，但是还没有和后端有交互，效果图如下
![](https://ueyao.github.io/image-hosting/blog/2021/1/20210121213849699_1070937386.png)

我们再添加一个上传按钮，来和后端交互，代码如下所示
```tsx
//index.tsx
<Col style={{ margin: '10px 20px' }}>
  <Upload {...props}>
    <Button icon={<UploadOutlined />}>选择</Button>
  </Upload>
</Col>
<Col style={{ margin: '10px 20px' }}>
  <Button type="primary" onClick={uploadOnClick}>
    上传
  </Button>
</Col>
```

上传按钮绑定了一个事件uploadOnClick，如下所示
``` tsx
//index.tsx
//文件列表
const [fileList, setFileList] = useState([]);

const uploadOnClick = async () => {
    //表单上传需要创建一个FormData对象
    const formData = new FormData();
    fileList.forEach(file => {
      formData.append('file', file);
    });
    console.log('formData:', formData.get('file'));
    //调用上传接口
    const result = await upload(formData);
    console.log('data:', result);
  };
```
前端调用的上传接口，如下所示
``` ts
//service.ts
export const upload = (data: any) => {
  //此处对axios做了一层封装
  return post('/smsTemp/upload', data);
};
//----------- 上面代码也可以这样写 ------
export const upload = (data: any) =>
    new Promise(resolve => {
        axios('/smsTemp/upload', {
          method: 'POST',
          data,
        })
          .then((r: any) => resolve(r))
          .catch((e: any) => console.error(e));
     });
```
前端代码，写得差不多了，可以写后端接口了

### 后端实现文件上传

控制器Controller的编写，代码如下所示
``` java
//SmsTemplateController.java
@PostMapping("/upload")
public R upload(MultipartFile file) {
    return smsTemplateService.upload(file);
}
```

具体实现，代码如下所示  

``` java
//SmsTemplateServiceImpl.java
@Override
public R upload(MultipartFile uploadFile) {
    //获得文件名        
    String name = uploadFile.getName();
    //获得文件全名，带后缀
    String fullName = uploadFile.getOriginalFilename();
    //获得文件后缀名
    String suffix = fullName.substring(fullName.lastIndexOf("."));
    //此处写死上传路径，可以根据要求自定义
    String pathFile = "/Users/simonxue/Developer/Temp/"+name + suffix;
    File file = new File(pathFile);
    try {
        //上传文件
        uploadFile.transferTo(file);
    } catch (IOException e) {
        e.printStackTrace();
    }
    return R.ofSuccess("上传文件成功", pathFile);
}
```
