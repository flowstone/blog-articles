---
title: "Ant Design中使用Upload上传组件如何自定义文件列表展示位置"
date: 2021-02-17 13:07:07
draft: false
categories: ["JS"]
---

## 软件环境
* macOS Big Sur 11.1
* React 16.12.0
* Ant Design 4.10.0
    
## 实际效果

现有一个需求，是上传文件，点击浏览文件按钮，选中文件后，在按钮的上方显示，上传的文件列表，如下图所示
![](https://ueyao.github.io/image-hosting/blog/2021/1/20210110125630518_804905005.png)

## 当前效果

目前使用阿里的Ant UI组件库，使用其中的上传组件，官方提供的示例，如下图如示
![](https://ueyao.github.io/image-hosting/blog/2021/1/20210113133631917_394793026.png)

本地使用后，如下图所示
![](https://ueyao.github.io/image-hosting/blog/2021/1/20210110125953680_1038926582.png)

如何才能实现，我们需要的效果呢，Google了好多文章，找到了一种方式，就是重写itemRender方法，自定义文件列表的展示，使用这个方法，需要重写多个action。

后来查看公司前端人员写的代码，看到另一种解决方法。

主要使用两个Upload组件，第一个Upload组件主要是展示文件列表，第二个Upload组件是选择文件上传的这个操作，不过，选择文件后，把文件列表在下方展示隐藏起来。

``` js
showUploadList: false, //不显示上传的列表
```

把得到的文件列表，赋值给第一个Upload组件中，大概如下：

``` js
beforeUpload(file: any, fileList: any) {
            setFileList(fileList); //设置文件列表
            return false; //不要调用上传文件接口
        },
```

``` html 
<!--第一个Upload组件-->
<Upload fileList={fileList}></Upload>
```

部分代码如下：
``` html
 <StyleContent>
    <StyleMainContent>
        <Button onClick={btnOnClick} type="primary">打开上传</Button>
        <Modal visible={isVisible} title="上传附件" footer={[]} closable>

            <div style={{ border: '1px solid #ccc', height: 150, marginBottom: 10 }}>
                <Upload fileList={fileList} onChange={onChange}></Upload>
            </div>
            <div style={{ textAlign: 'right' }}>
                <Upload {...updateProps} ><Button type="primary">浏览文件</Button></Upload>
                <Button style={{marginLeft:2}} onClick={() => { setIsVisible(false) }}>关闭</Button>
            </div>
        </Modal>
    </StyleMainContent>
</StyleContent>
```

``` js
    const [isVisible, setIsVisible] = useState(false);
    const [fileList, setFileList] = useState([]);

    const btnOnClick = () => {
        setIsVisible(true);
    }

    const updateProps = {
        name: 'file',
        beforeUpload(file: any, fileList: any) {
            setFileList(fileList);
            return false;
        },
        showUploadList: false,
        styly: {
            display: 'inline-block'
        }


    }
    const onChange = ({ file, fileList }: { file: any; fileList: any }) => {
        console.log(file, fileList);
        setFileList(fileList);
    };
```