---
title: 记录一次利用cropperjs修建图片并上传的过程
date: 2019-09-01 19:39:36
categories:
- vue
tags:
- 前端
- vue
---

# vuejs和cropperjs所走过的坑

> 用户上传自定义头像的功能是做完了，但是有的时候自己想要的图片不是这么'正好'，所以决定需要编辑图片

目标已经确定了，用户上传完图片之后打开一个遮罩层,里面用cropper这个插件来编辑，最后将编辑之后的图片传给服务器


## 环境准备

这个不要多说了，我们已经决定用cropperjs了，但是需要注意一点的是，不要和vue-cropper弄混.
cropperjs的github传送门：[link](https://github.com/fengyuanchen/cropperjs)

官网上也有文档可以查阅，不过本人的英语现在还是菜鸟级别，所以只能现在记录下来。

下载cropperjs:

```bash
npm install cropper --save
# or
cnpm install cropper --save
```


注册组件：
> 注意这个组件必须要有jquery

```javaScript
import 'cropperjs/dist/cropper.css';
import $ from 'jquery'
import Cropper from 'cropperjs'
```

一定要注意！！！ 上面这个css文件除非你用cdn在下面的style标签里面引入，像是这个

```css
<style>
@import "https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.5.5/cropper.min.css";
</style>

```
如果你没有引入css文件的话，就会出现下面的尴尬现象
![](https://s2.ax1x.com/2019/09/01/npjtKO.png)
而正常的应该是这样

![](https://s2.ax1x.com/2019/09/01/npjbsU.png)


## 使用cropper的正确姿势


cropperjs需要新建一个cropper对象，所以我们需要cropper变量

可以看一下目前我的vue关于图片上传这方面的参数

```bash

  cropper:"",  # cropper对象，一会需要新建
  open：false, # 裁剪图片的模态框，按自己需要添加
  idBefore: "", # img标签里面的src属性
  avatar: null  # 需要利用这个变量上传给服务器

```


> 需要注意的是，之前我们利用FormData上传图片不是直接上传img的src属性里面的图片么,为什么这次不是直接上传？

因为cropper对象获取编辑之后的图片是base64编码的，我们现在需要上传的是二进制图片


* 创建cropper对象

cropper对象我一开始的时候是在created(){}方法里面创建的，具体代码是这样

```html
/* html DOM树 */
<!-- 照片遮罩层 -->
    <Modal
        v-model="open"
        @on-ok="crppper"
        title="建议是200x200规格的哦"
        >
        <div style="width:360px;height:360px;">
            <img :src="idBefore" style="max-width:100%;height:auto;" id="avatar" alt="">
        </div>
    </Modal>

/* 方法 */
created(){
  let that = this;
    let image  = document.getElementById("avatar");
    this.cropper = new Cropper(image, {
           aspectRatio: 1,
           zoomable:false,
           scalable:false,
           movable:false,
           minContainerWidth:360,
           minContainerHeight:360
          });
}
```
> 但是一直会报错未找到img元素，因为vue的DOM树是在mounted(){}才会加载,created还未加载，所以代码应该是这样

```html
/* html DOM树 */
<!-- 照片遮罩层 -->
    <Modal
        v-model="open"
        @on-ok="crppper"
        title="建议是200x200规格的哦"
        >
        <div style="width:360px;height:360px;">
            <img :src="idBefore" style="max-width:100%;height:auto;" id="avatar" alt="">
        </div>
    </Modal>

/* 方法 */
mounted(){
  let that = this;
    let image  = document.getElementById("avatar");
    this.cropper = new Cropper(image, {
           aspectRatio: 1,
           zoomable:false,
           scalable:false,
           movable:false,
           minContainerWidth:360, # 规定大小
           minContainerHeight:360
          });
}
```

需要注意的是,现在的img里面的src只有在用户上传完图片之后才会正常。

当我们用户上传完图片的时候
```javaScript
  if(this.cropper){
    // 换cropper对象里面的img的src路径
    this.cropper.replace(this.idBefore);
// 开启遮罩层
    this.open = true;
  }
```



## 准备上传图片


裁剪完图片之后,我们需要获取编辑之后的图片，具体代码如下：

```javaScript
// 遮罩层点了确定按钮之后的函数
crppper(){
            let that = this;
            //这个代码返回值是 HTMLCanvasElement
            const canvas = this.cropper.getCroppedCanvas();
            // 换成了可以看成了src属性的图片
            this.idBefore = canvas.toDataURL("image/png");
            //  将avatar换成了blob对象
             canvas.toBlob(function(blob){
                 that.avatar = blob;
             });

        }
```


这个[HTMLCanvasElement](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement)对象是个重点，具体代码可以看官网。


由于FormData对象有两个append()方法

FormData.append("需要转换成的名称",变量)

FormData.append("需要转换成的名称",blob变量,"转换成的名称")


我们的avatar已经换成了blob对象


上传具体代码
```javaScript

userApi.uploadAvatar = (file) => {
    let data = new FormData();
    data.append("avatar", file, "avatar.jpg");
    return service({
        url: `${baseUrl}/updateAvatar`,
        data: data,
        headers: {
            'Content-Type': 'multipart/form-data'
        },
        method: 'post'

    })
}
```
