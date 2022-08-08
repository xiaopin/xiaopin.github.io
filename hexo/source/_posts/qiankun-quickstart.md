---
title: 微前端框架--qiankun初探
abbrlink: 6e6ea1fd
date: 2022-08-02 13:50:13
tags:
    - JavaScript
    - TypeScript
    - Vue
    - React
---

## 什么是微前端

> Techniques, strategies and recipes for building a modern web app with multiple teams that can ship features independently. --[Micro Frontends](https://micro-frontends.org/)
> 微前端是一种多个团队通过独立发布功能的方式来共同构建现代化 web 应用的技术手段及方法策略。

微前端架构具备以下几个核心价值：

- 技术栈无关
主框架不限制接入应用的技术栈，微应用具备完全自主权

- 独立开发、独立部署
微应用仓库独立，前后端可独立开发，部署完成后主框架自动完成同步更新

- 增量升级
在面对各种复杂场景时，我们通常很难对一个已经存在的系统做全量的技术栈升级或重构，而微前端是一种非常好的实施渐进式重构的手段和策略

- 独立运行时
每个微应用之间状态隔离，运行时状态不共享

微前端架构旨在解决单体应用在一个相对长的时间跨度下，由于参与的人员、团队的增多、变迁，从一个普通应用演变成一个巨石应用(Frontend Monolith)后，随之而来的应用不可维护的问题。这类问题在企业级 Web 应用中尤其常见。

## qiankun又是什么

qiankun（乾坤）是一个基于 [single-spa](https://github.com/single-spa/single-spa) 的[微前端](https://micro-frontends.org/)实现库，旨在帮助大家能更简单、无痛的构建一个生产可用微前端架构系统。qiankun 孵化自蚂蚁金融科技基于微前端架构的云产品统一接入平台。

官方 Example 的使用

- clone代码
```shell
$ git clone https://github.com/umijs/qiankun.git
$ cd qiankun
```

- 安装并运行示例程序
```shell
$ yarn install
$ yarn examples:install
$ yarn examples:start
```

## 创建qiankun项目

### 搭建项目结构

- 新建一个 `qiankun-examples` 空目录：

```shell
mkdir qiankun-examples
```

- 新建主应用入口程序(Vite + Vue3 + TypeScript)，项目名称为 `app`

```shell
$ npm create vite@latest
```

输入命令后，根据提示创建对应的项目，控制台输出如下信息：
```
✔ Project name: … app
✔ Select a framework: › vue
✔ Select a variant: › vue-ts
```

- 新建第一个微应用（Webpack + Vue3 + TypeScript），名称叫 `home`

```shell
$ vue create home
```

根据提示创建好项目即可。

- 再次新建第二个微应用（Webpack + Vue3 + TypeScript），名称叫 `profile`

```shell
$ vue create profile
```

同样的，根据提示创建好项目即可。

搭建好后，项目目录结构如下所示：

```
qiankun-examples
├── .editorconfig
├── .gitignore
├── .prettierignore
├── .prettierrc.js
├── app
├── home
└── profile
```

`app` 目录是 qiankun 框架的主应用项目，`home` 和 `profile` 目录是两个微应用项目（你可以根据实际情况添加 N 多个微应用，并且不限技术栈）；`.editorconfig` 为编辑器配置，`.prettierignore` 和 `.prettierrc.js` 为 `prettier` 代码格式化插件的配置信息，点开头的这几个配置文件不是必要的，根据个人喜好。

### 主应用安装依赖

项目整体结构搭建完毕后，我们就可以开始集成 `qiankun` 框架了。

1、在主应用中集成 `qiankun` 框架

```shell
$ cd app
$ npm install qiankun --save
```

2、在主应用中注册微应用

编辑 `app/src/App.vue` 文件，代码如下：

```vue
<template>
    <div>
        <div class="main-app">
            <span @click="open('/home')">Home</span>
            <span>|</span>
            <span @click="open('/profile')">Profile</span>
        </div>
        <div id="subapp-container"></div>
    </div>
</template>

<script setup lang="ts">
import { registerMicroApps, start, setDefaultMountApp } from 'qiankun'

// 注册微应用
registerMicroApps([
    {
        name: 'Home Micro App',
        entry: '//localhost:9001',
        container: '#subapp-container',
        activeRule: '/home'
    },
    {
        name: 'Profile Micro App',
        entry: '//localhost:9002',
        container: '#subapp-container',
        activeRule: '/profile'
    }
])
// 设置默认挂载的应用
setDefaultMountApp('/home')
// 启动应用
start()

const open = (path: string): void => {
    window.history.replaceState(null, path, path)
}
</script>
```

### 配置微应用

微应用虽然不需要额外安装任何其他依赖即可接入 `qiankun` 主应用，但是需要做一些相关的配置才能与主应用一起工作。

这里只介绍 [基于 Vue 的微应用](https://qiankun.umijs.org/zh/guide/tutorial#vue-%E5%BE%AE%E5%BA%94%E7%94%A8) 的接入方式，其他框架的微应用请参考：

- [React 微应用](https://qiankun.umijs.org/zh/guide/tutorial#react-%E5%BE%AE%E5%BA%94%E7%94%A8)
- [Vue 微应用](https://qiankun.umijs.org/zh/guide/tutorial#vue-%E5%BE%AE%E5%BA%94%E7%94%A8)
- [Angular 微应用](https://qiankun.umijs.org/zh/guide/tutorial#angular-%E5%BE%AE%E5%BA%94%E7%94%A8)
- [非 webpack 构建的微应用](https://qiankun.umijs.org/zh/guide/tutorial#%E9%9D%9E-webpack-%E6%9E%84%E5%BB%BA%E7%9A%84%E5%BE%AE%E5%BA%94%E7%94%A8)

1、新建 `home/src/public-path.js` 文件，并写入如下内容：

```js
if (window.__POWERED_BY_QIANKUN__) {
  // eslint-disable-next-line no-undef
  __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}
```

2、入口文件 `main.ts` 修改，为了避免根 id `#app` 与其他的 DOM 冲突，需要限制查找范围，并导出 [bootstrap、mount、unmount 三个生命周期钩子](https://qiankun.umijs.org/zh/guide/getting-started#1-%E5%AF%BC%E5%87%BA%E7%9B%B8%E5%BA%94%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E9%92%A9%E5%AD%90)，以供主应用在适当的时机调用。

```ts
import './public-path'
import * as Vue from 'vue'
import { createRouter, createWebHistory, RouterHistory, Router } from 'vue-router'
import App from './App.vue'
import routes from './router'
import store from './store'

declare global {
    interface Window {
        __POWERED_BY_QIANKUN__: string | boolean
    }
}

let router: Router | undefined = undefined
let instance: Vue.App<Element> | undefined = undefined
let history: RouterHistory | undefined = undefined

function render(props?: { container: HTMLElement | null | undefined }) {
    const { container } = props || {}

    history = createWebHistory(window.__POWERED_BY_QIANKUN__ ? '/home' : '/')
    router = createRouter({
        history,
        routes
    })

    instance = Vue.createApp(App)
    instance.use(router)
    instance.use(store)
    instance.mount(container ? (container.querySelector('#app') as HTMLElement) : '#app')
}

// 独立运行时
if (!window.__POWERED_BY_QIANKUN__) {
    render()
}

// 导出 bootstrap、mount、unmount 三个生命周期钩子

export async function bootstrap() {
    console.log('[vue] vue app bootstraped')
}

export async function mount(props: any) {
    console.log('[vue] props from main framework', props)
    render(props)
}

export async function unmount() {
    instance?.unmount()
    instance && instance._container && (instance._container.innerHTML = '')
    instance = undefined
    router = undefined
    history?.destroy()
    history = undefined
}
```

3、修改打包配置（`vue.config.js`）

为了便于开发，指定了启动的端口，这样确保项目每次都运行在相同的端口上，并且由于 `qiankun` 是通过 `fetch` 去获取微应用的静态资源(图片、js文件等等)，所以必须要求这些静态资源支持跨域。

```js
const { name } = require('./package')

module.exports = {
    devServer: {
        port: 9001, // 指定端口
        headers: {
            'Access-Control-Allow-Origin': '*' // 配置跨域
        }
    },
    configureWebpack: {
        output: {
            library: `${name}-[name]`,
            libraryTarget: 'umd', // 把微应用打包成 umd 库格式
            jsonpFunction: `webpackJsonp_${name}`
        }
    }
}
```

至此，我们的微应用已经配置完毕了；按照如上步骤将 `profile` 这个微应用也进行修改，并且端口指定为 `9002`。

### 启动项目

经过以上的配置，我们可以把主应用和微应用都跑起来查看效果了，但是每个应用都需要我们 `cd xxx && npm run serve` 这么操作一遍的话，岂不是累死人，像我们现在只有三个应用还好，如果是十个八个呢，光启动项目都得半天了。所以这里通过 `npm-run-all` 库来帮我们进行偷懒，一键安装依赖 & 一键运行项目。

- 新建 `qiankun-examples/package.json`

```json
{
    "name": "qiankun-examples",
    "version": "1.0.0",
    "main": "index.js",
    "scripts": {},
    "devDependencies": {}
}
```

- 安装 `npm-run-all`

```shell
$ npm install npm-run-all -D
```

- 配置 `scripts` 脚本命令

```
{
    "name": "qiankun-examples",
    "version": "1.0.0",
    "main": "index.js",
    "scripts": {
        "install": "npm-run-all --serial install:*",
        "start": "npm-run-all --parallel start:*",
        "install:app": "cd app && npm install",
        "start:app": "cd app && npm run dev",
        "install:home": "cd home && npm install",
        "start:home": "cd home && npm run serve",
        "install:profile": "cd profile && npm install",
        "start:profile": "cd profile && npm run serve"
    },
    "devDependencies": {
        "npm-run-all": "^4.1.5"
    }
}
```

- 配置完成，我们可以一键安装依赖和一键启动项目了

```shell
$ cd qiankun-examples
$ npm run install
$ npm run start
```

项目启动成功后，浏览器访问 `http://localhost:5173/` 即可看到效果（这个端口是Vite默认的，你根据实际情况访问你的地址即可）。

至此，我们的整个项目搭建就算告一段落了，更多的隐藏技能则需要你自行去探索了（比如：全局数据共享、主应用和微应用之间的数据交互等）。

## 参考

- [qiankun](https://qiankun.umijs.org/zh)
- [qiankun-examples 示例代码](/images/2022/qiankun-examples.zip)