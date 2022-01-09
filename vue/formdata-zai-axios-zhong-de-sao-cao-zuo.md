---
title: FormData在axios中的骚操作
date: 2019-08-20T21:34:46.000Z
categories:
  - vue
tags:
  - axios
  - FormData
---

# FormData在axios中的骚操作

做一个博客，最起码你要可以更换用户头像吧，不然谁知道你是谁啊。

当时有过用ajax上传图片的经验，但是这次就不一样了，因为我用了axios的拦截器，详细可以看我另一篇帖子。

由于我上传图片的方式post，而我在拦截器上的post的请求都自动加上 Content-Type = 'application/json; charset=utf-8'

axios拦截器的代码在下面

```

service.interceptors.request.use(
    config => {
        const token = store.getters.getToken ? store.getters.getToken : localStorage.getItem('token');

        if (token) {
            config.headers["Authentication"] = token
        }
        if (config.method === 'post' || config.method === 'put') {
            // config.data = qs.stringify({...config.data });

                config.headers['Content-Type'] = 'application/json; charset=utf-8';
                config.data = JSON.stringify({...config.data })

        } else if (config.method === 'get') {
            config.headers['Content-Type'] = 'application/x-www-form-urlencoded';
            config.params = {...config.params }
        }


        return config;
    }, error => {

        return Promise.reject(error);
    }

)
```

从上面可以看出来，这个代码是不行的，因为我们上传图片等文件的时候我们需要我们的请求头 Content-Type 是'multipart/form-data'才可以上传成功。

## 解决办法

经过查阅知道，axios不仅可以在拦截器上设置请求头信息，也可以在封装后的请求体上面加入请求头，headers:{}属性，所以，我上传的图片代码是这样

```

userApi.uploadAvatar = (file) => {
    let data = new FormData();
    data.append("avatar", file)
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

可以看到，我们加上了请求头，但是这样并没有什么效果，因为我在拦截器上已经设置死了 只要是post方法，必须是另一种请求头，但是如果将拦截器上面的删除掉的话，每个post方法都要加上headers，很麻烦。所以我们可以采取直接照顾特殊方法的方法，直接修改拦截器post部分的代码：

```
if (config.method === 'post' || config.method === 'put') {
            // config.data = qs.stringify({...config.data });
            if (config.headers['Content-Type'] === 'multipart/form-data') {

            } else {
                config.headers['Content-Type'] = 'application/json; charset=utf-8';
                config.data = JSON.stringify({...config.data })
            }

        }
```

可以看到如果我们的请求头是 config.headers\['Content-Type'] === 'multipart/form-data' 不做任何操作，问题解决。
