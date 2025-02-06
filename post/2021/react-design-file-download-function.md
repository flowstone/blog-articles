---
title: "React中如何实现文件下载功能"
date: 2021-01-05 13:05:30
draft: false
categories: "JS"
---

### 场景描述
在前端页面中，经常会使用到文件下载功能，我们如何实现下载功能呢？

### 开发环境
* React ^16.12.0
* Ant Design ^5.0.12
* SpringBoot 2.3.0

### 问题分析
首先，想到下载，肯定要涉及文件流(字节流)，后端接口返回文件流，前端页面把文件流保存成对应的文件。

### 后端接口设计
写一个无返回值的方法，方法参数是HttpServletResponse，代码如下所示
``` java
//SmsTemplateController.java
@GetMapping("/download")
public void download(HttpServletResponse response) throws IOException {
    smsTemplateService.download(response);
}
```
接口的具体实现，如下所示
``` java
//SmsTemplateServiceImpl.java
 @Override
public void download(HttpServletResponse response) throws IOException {
    //设置字符集
    response.setCharacterEncoding("utf-8");
    //response.setContentType("application/vnd.ms-excel");
    //设置响应的头
    response.setHeader("Content-Disposition", "attachment;filename = " + "helloworld.png");
    //获得响应的字节流
    OutputStream outputStream = response.getOutputStream();
    //文件写死，根据需求改变
    File file = new File("/Users/simonxue/Developer/Temp/file.png");
    //把文件内容复制到字节输出流中
    FileIoUtil.copyFile(file, outputStream);
}
```
我们的文件流主要是放在接口响应中，其中涉及到一个工具类，如下所示
``` java
//FileIoUtil.java
/**
 * 复制file中的内容到输出流中
 * @param file 文件内容
 * @param out 输出流
 * @throws IOException
 */
public static void copyFile(File file, OutputStream out) throws IOException {
    InputStream input = null;
    try {
        input = new FileInputStream(file);
        byte[] buf = new byte[1024];
        int bytesRead;
        while ((bytesRead = input.read(buf)) > 0) {
            out.write(buf, 0, bytesRead);
        }
    } finally {
        input.close();
        out.close();
    }
}
```
使用Postman调用接口，效果如下

![](https://ueyao.github.io/image-hosting/blog/2021/1/20210121223257127_1184200995.png)

接口响应体中，包含了大量的乱码，如果出现这个效果，则代表接口已返回文件流。

### 前端页面设计
首先，创建一个下载按钮，如下所示
``` tsx
//index.tsx
<Button type="primary" onClick={downloadOnClick}>
    下载
</Button>
```
并且绑定一个点击事件，如下所示
``` tsx
//index.tsx
const downloadOnClick = async () => {
    await imgDownload();
};
```

下载文件具体操作，如下所示
``` ts
//service.ts
export const imgDownload = (data?: any) =>
  new Promise((resolve, reject) => {
    axios
      //接口请求地址
      .get('/smsTemp/download', {
        //请求参数中，带上响应类型，否则文件损坏
        responseType: 'blob',
      })
      //接口响应成功后
      .then(res => {
        const content = res.data;
        const blob = new Blob([content], { type: 'charset=utf-8' });
        //定义文件名称，写死，根据需求自行更改
        const fileName = 'helloworld.png';
        //判断是不是IE浏览器
        if (!window.navigator.msSaveOrOpenBlob) {
          // 创建一个a标签
          const downlink = document.createElement('a');
          // 设置a标签download属性
          downlink.download = fileName;
          // 设置a标签display属性
          downlink.style.display = 'none';
          // 设置a标签href属性
          downlink.href = URL.createObjectURL(blob);
          // 添加a标签到Body中
          document.body.appendChild(downlink);
          //点击a标签
          downlink.click();
          //清除a标签中的href属性
          URL.revokeObjectURL(downlink.href);
          //从body中删除该a标签
          document.body.removeChild(downlink);
        } else {
          //IE浏览器
          window.navigator.msSaveBlob(blob, fileName);
        }
      })
      //异常处理
      .catch(e => {
        reject('下载失败');
      });
  });

```