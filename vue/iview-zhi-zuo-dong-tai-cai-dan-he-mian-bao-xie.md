---
title: iview 制作动态菜单和面包屑
date: 2019-07-20T17:41:44.000Z
categories:
  - iview
tags:
  - 前端
---

# iview 制作动态菜单和面包屑

本次项目搭建的后台是这样的，可以动态生成菜单导航和面包屑，故此记录 ![](https://s2.ax1x.com/2019/07/20/ZzOSuq.png)

* 生成动态菜单

首先我们直接粘贴iview官网的导航菜单的demo代码

我的项目结构如下图所示:

![](https://s2.ax1x.com/2019/07/20/ZzOV29.png)

首先，菜单的路由表如下:

```javascript

export default new Router({
    mode: 'history',
    routes: [{
            path: '/index.html',
            meta: { title: '主页' },
            component: () =>
                import ('@/ivews/index'),
            children: [{
                path: 'status',
                meta: ["主页", "状态面板"],
                component: () =>
                    import ('@/components/status')
            }]
        },
        {
            path: '/login.html',
            meta: { title: '登陆' },
            component: () =>
                import ('@/ivews/login')
        },
        {
            path: '/',
            redirect: '/index.html'
        }
    ]
})
```

vuex状态管理menu.js内容如下:

```javascript

const state = {
    menu: [{
            id: 1,
            to: '/index.html/status',
            name: '状态面板',
            icon: 'md-desktop'
        },

    ],

}

const getters = {
    menus: state => state.menu
}

const mutations = {

}

const actions = {

}


export default { state, getters, mutations, actions }
```

header-manager.vue里面写的就是导航菜单，具体代码如下：

```html
<template>
    <div>
    <Menu mode="horizontal" :theme="theme1" >
        <MenuItem  v-for="(menu,index) in menus" :key='index' :name="menu.id" :to="menu.to">
            <Icon :type="menu.icon" />
            {{menu.name}}

        </MenuItem>
    </Menu>
    </div>
</template>

<script>

 import {Menu,MenuItem,Submenu,MenuGroup} from 'iview'
 import {mapGetters} from 'vuex'

    export default {
      name: 'Header',
        data () {
            return {
                theme1: 'dark'
            }
        },
        components:{
          Menu,
          MenuItem,
          MenuGroup,
          Submenu
        },
        methods: {
            query(){
                console.log(menus);
            }
        },
        computed: {
            ...mapGetters({
                menus: 'menus'
            })
        },
    }
</script>
```

> 总结一下思路就是，根据iview的MenuItem里面的to属性，来对应路由表的地址，从而做到点击之后跳转相应页面的效果，然后根据vuex动态生成当前菜单内容就好了

* 动态生成面包屑

本次项目的面包屑主要完成对象当前路径名称的功能，因此，我们可以根据对应当前路由路径来完成，重要代码如下:

```html
<Breadcrumb  :style="{margin: '20px 0'}">
        <BreadcrumbItem v-for="(item,index) in $route.meta" :key='index'>
          {{item}}
</BreadcrumbItem>
```
