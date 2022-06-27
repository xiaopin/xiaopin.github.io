---
title: mockjs 的入门使用
abbrlink: 25c5823c
date: 2022-06-23 10:04:38
tags:
    - Vue
    - JavaScript
    - TypeScript
---

在做 web 项目时，常常会通过一些手段来模拟后台返回的数据，今天主要是介绍下 mockjs 的使用。

## 安装

```
npm i mockjs -D
```

## 使用

- 新建 index.js 文件

在 `src` 目录下新建 `mock` 文件夹，然后在该文件夹下新建 `index.js` 文件，并写入如下代码：

```JavaScript
import MockJS, { mock } from 'mockjs'

mock('/api/home', 'get', {"code": 0, "data": {}})
mock('/api/user/login', 'post', {"code": 0, "data": {}})
```

- 使 mockjs 正常工作

在 `main.js` 中导入刚才新建的 `index.js` 即可：

```JavaScript
if (process.env.NODE_ENV !== 'production') {
    require('@/mock/index')
}
```

> 这里通过环境变量 NODE_ENV 来判断，在生产环境下不需要引入mock，更推荐的做法是在 `.env.development` 中配置一个自定义环境变量，通过判断该环境变量来决定是否需要引入mock，这样在使用上更加灵活。

经过以上配置，当你通过 `axios` 调用 `/api/home` 接口时，就能获取到 Mock 返回的 `{"code": 0, "data": {}}` 这个数据了。

## 延时响应

如果你希望模拟更真实的网络请求或你的项目需要模拟一个弱网环境，你可以设置 timeout 参数来实现。

在刚才的 index.js 中添加如下代码：

```JavaScript
MockJS.setup({
    timeout: 400
})
MockJS.setup({
    timeout: '200-600'
})
```

`timeout` 参数用来指定被拦截的 Ajax 请求的响应时间，单位是毫秒。值可以是正整数，例如 `400`，表示 `400` 毫秒后才会返回响应内容；也可以是横杠 '-' 风格的字符串，例如 `'200-600'`，表示响应时间介于 `200` 和 `600` 毫秒之间；默认值是`'10-100'`。

## 数据模板定义

mockjs 除了能`拦截`后台接口返回数据外，还能按照规则生成一些随机数据，感兴趣可以查看官方文档中关于 [数据模板定义](http://mockjs.com/examples.html)、[数据占位符定义](http://mockjs.com/examples.html#DPD)、[Mock.Random](https://github.com/nuysoft/Mock/wiki/Mock.Random) 的详细说明。

以上就是简单的 mockjs 入门教程，实际的项目数据可能会比较复杂，所以需要你编写特定的数据模板、方法来处理返回的数据，这个建议你多参考一下开源项目的代码，如：vue-element-admin等。

## 参考

- [mockjs官网](http://mockjs.com)
- [mockjs wiki](https://github.com/nuysoft/Mock/wiki)
- [mock.js语法规范](https://www.jianshu.com/p/4579f40e6108)
