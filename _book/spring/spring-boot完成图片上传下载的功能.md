---
title: spring boot完成图片上传下载的功能
date: 2019-08-19 19:34:23
categories:
- springboot
tags:
- java IO
- springboot
---

# springboot完成图片上传下载功能

由于我的博客项目最起码用户的更换头像操作要做出来啊，所以这个地方即使是我再怎么不想去碰也是不可能的。所以今天就拿出来唠一唠，这个springboot是如何操作的

## 图片上传

上传这个好说，springMVC如果都学过的话，应该都好理解 MultipartFile这个类，上传文件当然用它了

我的后端项目路径目前是这样的
> D:\code\java\eblog

而我想把图片存到他的子目录 avatar下面
也就是：
> D:\code\java\eblog\avatar


下面就是一些纯粹的IO读写操作了。
首先，要保证我们的文件夹要存在，毕竟有一个avatar呢，所以我们需要在启动这个项目的时候先确定一下是否存在，不存在就创建。
> 这里提一点，为什么我们不直接在Controller层判断，因为，一遍一遍判断太烦了，还浪费资源，启动的时候检查一遍就OK了



```java
//  启动器代码
@Slf4j
@Configuration
@Order(Ordered.HIGHEST_PRECEDENCE)
public class StartLinster implements ApplicationListener<ApplicationEvent> {

    @Autowired
    TokenService tokenService;

    @Override
    public void onApplicationEvent(ApplicationEvent applicationEvent) {
        String filePath = System.getProperty("user.dir") + File.separator + "avatar";

        File file = new File(filePath);
        if(!file.exists()){
            file.mkdirs();
        }
    }
}
```

现在，我们有了文件夹，后面我们保存文件就可以了

```java
@ApiOperation("上传用户图片")
    @PostMapping("/updateAvatar")
    public BaseResponse updateProfile(@RequestParam(value = "avatar",required = true)MultipartFile avatar,
                                      HttpServletRequest request){
        BaseResponse baseResponse = new BaseResponse();

//  获取并创建文件夹
        String property = System.getProperty("user.dir");

        String filePath = property + File.separator + "avatar";

//  随机生成文件名称
        String trueFilename = UUID.randomUUID().toString();


//  获取本来文件名称
        String filename = avatar.getOriginalFilename();

// 截取 文件后缀名
        String type = filename.indexOf(".") != -1 ? filename.substring(filename.lastIndexOf(".")+1,filename.length()):null;

// 重新构建文件名
        String trueFile = null == type ? filePath + File.separator + trueFilename :
                filePath + File.separator + trueFilename + "." + type;

        try {
          //  重点！！！！  将MultipartFile文件存到普通文件中！！！
            avatar.transferTo(new File(trueFile));
        } catch (IOException e) {
            e.printStackTrace();
            baseResponse.setStatus(HttpStatus.BAD_REQUEST.value());
            baseResponse.setMessage("图片上传失败");

            return baseResponse;
        }

```


## 图片下载

下载的话，我一开始是想直接把我的磁盘目录写进数据库中，然后读取的时候直接读，但是这样就暴露了我们的磁盘结构，很不安全，所以不得不放弃，而且也不现实。

后来，我决定直接创建一个虚拟映射就可以了啊，而且简单的一匹

只要把我们存到磁盘的文件名称保存到数据库就ok，到时候直接让前端访问我们的这个服务器ip+名称就行了啊

明白了思路，我们开始设置，其实很简单，在我们的springboot的项目配置中添加两个属性就行

```yaml
spring:
  resources:
    static-locations: classpath:/META-INF/resources/,classpath:/resources/, classpath:/static/, classpath:/public/,file:${web.upload-path}

web:
  upload-path: ${user.dir}/avatar
```

这个web.upload-path是我自己定义的，完全可以忽略，上面的那个抄下来就好了，file: 表示是项目的物理磁盘。
之后访问 项目ip地址/文件名 就可以访问到了。
