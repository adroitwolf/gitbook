---
title: vue请求后端的一般思路
date: 2019-07-22 06:18:44
categories:
- vue
tags:
- 前后端分离
- vue
---

# vue请求后端的一般思路


这里说的vue指的是利用vue-cli搭建的前端项目，并且后端的项目是利用springboot搭建。


做前后端分离的时候，我们不可避免的会和后端出现交互，那么如何结构化的进行交互是这篇文章的重点。


> 这里没有采用ajax，而是利用了axios

## axios 封装


首先如果进行交互，那么我们肯定不能只考虑交互成功的时候，肯定也要考虑因为各种因素导致的失败，但是如果每次访问都要考虑同一因素导致的失败就会产生代码冗杂。

因此，我们可以考虑利用axios的拦截器对一些信息进行统一处理。

并且如果进行交互，那么vuex也是必须的，所以，如果不熟悉vuex的，要先进行了解。

## 下载相应的模块


首先，现在axios是必须的

```bash
npm install axios --save
npm install vue-axios --save # vue版的axios的插件
npm install qs.js --save # 可以对字符串进行json处理

npm install  js-md5 # md5加密
```


## service.js进行封装
在我的项目中，我新建了一个util工具包，并且新建了一个service.js来封装axios，具体代码如下:


```javaScript
import axios from 'axios'
import qs from "qs"
import store from '../store';
import { Message } from 'iview';


// axios相应的封装
const service = axios.create({
    timeout: 5000,
    baseURL: "http://localhost:8099"
});


service.interceptors.request.use(
    config => {
        const token = store.state.token;
        if (token) {
            config.headers["Authentication"] = token
        }

        config.method === 'post' ?
            config.data = qs.stringify({...config.data }) :
            config.params = {...config.params };
        config.headers['Content-Type'] = 'application/x-www-form-urlencoded'
        return config;
    }, error => {

        return Promise.reject(error);
    }

)

service.interceptors.response.use(
    response => {
        return response;
    },
    error => {
      // 这里只是简单的判断了是不是超时，真实的项目肯定还复杂
        if (error.message.includes('timeout')) { // 判断请求异常信息中是否含有超时timeout字符串
            Message.error("网络连接超时");
            //return Promise.reject(error); // reject这个错误信息
        } else {
            Message.error("服务异常！");

        }
        return Promise.reject(error);
    }
);

export default service
```


## vuex进行相应的模块封装

比如说本次的项目中，我们可以写一个user模块，来控制用户登陆。


代码如下:


```javaScript
import axios from 'axios'
import userApi from '@/api/user'

const state = {
    username: '',
    password: '',
    token: null
};

const getters = {
    getToken: state => state.token
};


const mutations = {

    setToken: (state, token) => {
        state.token = token
    }
};

const actions = {
    login({ commit }, { username, password }) {
        return new Promise((resolve, reject) => {
            userApi.login(username, password)
                .then(response => {
                    const token = response.data.token;
                    commit('setToken', token);
                    resolve(response)
                })
                .catch(error => {
                    reject(error)
                })
        })
    }
};

export default { state, getters, mutations, actions };
```


上面的action里面的login方法是单独抽离出来的代码，用来实现登陆功能。


# 方法的封装


为了对应结构化的处理，我们需要将方法单独抽离出来。


比如此次项目，我们新建一个api文件夹，里面新建一个user.js

里面具体代码如下:


```javaScript
import service from '@/util/service'
import md5 from 'js-md5'
const userApi = {}

// 登陆函数
userApi.login = (username, password) => {
    return service({
        url: '/login',
        data: {
            username: md5(username),
            password: md5(password)
        },
        method: 'post'
    })
}


export default userApi
```
