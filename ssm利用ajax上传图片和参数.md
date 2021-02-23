---
title: ssm利用ajax上传图片和参数
date: 2019-03-31 14:33:54
categories:
- j2EE
tags:
- 分布式
- 文件服务器
- nginx反向代理
---


# Ajax选择性上传图片

- 技术选型:
- bootstrap-fileinput 渲染上传框
- FormData 用于传递参数
- bootstrap 前端渲染

<!-- more -->

---------

## 技术需求

> 用于后台CMS更新商品的时候，需要上传图片和参数。但是为了节省流量，这里的图片是选择性上传，也就是说，后台分辨不出来，你到底有没有上传图片。
+ 功能截图：
![前端功能截图](https://s2.ax1x.com/2019/03/31/Arcftf.png)

## 具体实现

### 前端代码部分

``` html

<%--编辑的模态框--%>
<div class="modal fade" id="editContent" tabindex="-1" role="dialog" aria-labelledby="myModalLabel">
    <div class="modal-dialog" role="document">
        <div class="modal-content">
            <div class="modal-header text-center">
                <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
                <h4 class="modal-title">商品信息修改</h4>
            </div>
            <div class="modal-body">
                <form class="form-horizontal" id="editContentForm">
                    <div class="form-group">
                        <label class="col-sm-2 control-label">编号</label>
                        <div class="col-sm-10">
                            <input type="text" class="form-control" name="id"  id="contentId_update" disabled="disabled"/>
                        </div>
                    </div>
                    <div class="form-group">
                        <label class="col-sm-2 control-label">内容标题</label>
                        <div class="col-sm-10">
                            <input type="text" name="title" class="form-control" id="contentTitle_update">
                        </div>
                    </div>
                    <div class="form-group">
                        <label class="col-sm-2 control-label">子标题</label>
                        <div class="col-sm-10">
                            <input type="text" name="subTitle" class="form-control" id="contentSubTitle_update">
                        </div>
                    </div>
                    <div class="form-group">
                        <label class="col-sm-2 control-label">标题描述</label>
                        <div class="col-sm-10">
                            <input type="text" name="titleDesc" class="form-control" id="contentTitleDesc_update">
                        </div>
                    </div>
                    <div class="form-group">
                        <label class="col-sm-2 control-label">链接</label>
                        <div class="col-sm-10">
                            <input type="text" name="url" class="form-control" id="contentUrl_update">
                        </div>
                    </div>
                    <div class="form-group hidden" >
                        <label class="col-sm-2 control-label">所属分类</label>
                        <div class="col-sm-10">
                            <input type="text" name="categoryId" class="form-control" id="contentCategoryId_update" disabled="disabled">
                        </div>
                    </div>
                    <div class="form-group">
                        <label class="col-sm-2 control-label">图片</label>
                        <div class="col-sm-10">
                            <input type="file" name="picFile" id="contentPic_update" />
                        </div>
                    </div>
                    <div class="form-group">
                        <label class="col-sm-2 control-label">图片2</label>
                        <div class="col-sm-10">
                            <input type="file" name="pic2File" id="contentPic2_update"/>
                        </div>
                        </div>
                    <div class="form-group">
                        <label class="col-sm-2 control-label">内容</label>
                        <div class="col-sm-10">
                            <input type="text" name="content" class="form-control" id="contentContent_update">
                        </div>
                    </div>
                </form>
            </div>
            <div class="modal-footer">
                <div class="col-sm-12">
                    <button type="button" class="btn btn-default col-sm-5" data-dismiss="modal">关闭</button>
                    <button type="button" class="btn btn-primary pull-right col-sm-5" onclick= "emp_update_btn()">更新</button>
                </div>
            </div>
        </div>
    </div>
</div>

<script>
//ajax异步上传
function emp_update_btn () {
        //没有检验数据是否正确 请注意

        var formdata = new FormData($("#editContentForm")[0]);
        formdata.append("id",$("#contentId_update").val());


        $.ajax({
          url:'${pageContext.request.contextPath}/content/updateContent.do',
            type:'post',
            dataType:'json',
            contentType: false,
            processData: false,
            data:formdata,
            success:function (result) {
                if(result.result === "success"){
                    swal("成功","成功更新数据！","success");
                    listAll();
                    $("#editContent").modal("hide");
                }
            }

        });
</script>

```

+ 需要注意的是 这里和通常的ajax不一样,他传输的不是一般的JSON对象,而是FormData对象,他的写法也和平常的写法不一样
+ ![ajax注意点](https://s2.ax1x.com/2019/03/31/Arge3D.png)
+ 关于那个图片到底有没有上传的功能,前端不需要考虑

-------

### 后台代码部分

```java

    @Value("${FILE_URL}")
    private  String FILE_URL;

    /**
     * function:更新商品
     * @param tbContent
     * @param pic
     * @param pic2
     * @return
     */
    @RequestMapping(value = "/updateContent.do")
    @ResponseBody
    public Map<String,String> updateContent(TbContent tbContent, @RequestParam(value = "picFile",required = false)MultipartFile pic, @RequestParam(value = "pic2File",required = false)MultipartFile pic2) throws IOException {

        HashMap<String,String> map = new HashMap<>();

        System.out.println(pic + ":"+FILE_URL);

        String picFileID = new String();
        String pic2FileID =new String();
        if(!pic.isEmpty()){
             picFileID = FastDFSClient.uploadFile(pic.getInputStream(),pic.getOriginalFilename());
            if(picFileID == null){
                map.put("result","failed");
                return map;
            }
            tbContent.setPic(FILE_URL+picFileID);
            System.out.println(tbContent.getPic());
        }

        if(!pic2.isEmpty()){
            pic2FileID = FastDFSClient.uploadFile(pic2.getInputStream(),pic2.getOriginalFilename());
            if(pic2FileID == null){
                map.put("result","failed");
                return map;
            }
            tbContent.setPic2(pic2FileID);
        }

        try {
            contentService.updateContent(tbContent);
        } catch (IOException e) {
            e.printStackTrace();
            map.put("result","failed");
            return map;
        }
        map.put("result","success");
        return map;
    }
}

```
>之前我用TbContent实体类去接收前端参数的时候，会加上@RequestBody,但是会报出400/415错误，因为FormData的编码并不是application/json;charset = utf-8 ，需要把这个注解删除。
