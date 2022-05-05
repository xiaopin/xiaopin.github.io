---
title: element-plus Loading 和 Message 组件在自动导入的情况下不显示
abbrlink: dd9d0a4f
date: 2022-05-05 10:45:13
tags:
    - Vue
---

最近在学习 Vue3 + ElementPlus，并且按照 ElementPlus 官方文档上的使用指南，采用了 [自动导入](https://element-plus.org/zh-CN/guide/quickstart.html#按需导入) 的用法，在其他组件比如 `el-button`、`el-icon` 等组件均能正常工作，但是 Loading、Message 等这种[以服务的方式来调用](https://element-plus.org/zh-CN/component/loading.html#以服务的方式来调用)的组件却不能正常显示出来。

示例代码：

```TypeScript
import { ElLoading } from 'element-plus'

ElLoading.service({...})
```

当我们运行代码后，却发现页面没有任何的反馈，并没看到我们的“正在加载中”的提示，但是此时我们去查看页面元素，能清晰的看到我们的 el-loading 组件已经添加在 body 节点下面了，至于不显示的问题，只是样式的问题而已。

在 main.ts 中添加以下代码
```TypeScript
import 'element-plus/theme-chalk/el-loading.css'
import 'element-plus/theme-chalk/el-message.css'
```

> 对于这种以服务的方式进行调用的组件，我们需要在 main.ts 中手动引入 css 样式。
