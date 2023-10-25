---
title: 为 ElementUI 中的 Message 设置全局的偏移量
abbrlink: cc59bea6
date: 2023-02-17 14:17:51
tags:
  - JavaScript
  - Vue
---

部分系统会在顶部有导航栏，element-ui 中 Message 消息提示的默认偏移量是 `20px`，会遮挡顶部内容，所以需要将偏移量进行全局调整。

## 环境

- Vue 2.x
- ElementUI 2.x

## 源码

```JavaScript
import { Message as ElementUI_Message } from 'element-ui'

const OffsetMessage = function(options, type) {
    options = options || {}
    if (typeof options === 'string') {
        options = { message: options }
    }
    if (options.offset === undefined || options.offset === null) {
        options.offset = 90
    }
    if (typeof type === 'string') {
        options.type = type
    }
    return ElementUI_Message(options)
}

for (let prop in ElementUI_Message) {
    if (prop === 'success' || prop === 'warning' || prop === 'info' || prop === 'error') {
        OffsetMessage[prop] = function(options) {
            return OffsetMessage(options, prop)
        }
        continue
    }
    OffsetMessage[prop] = ElementUI_Message[prop]
}

export default OffsetMessage
export const Message = OffsetMessage
```

## 替换 Message

修改 main.js 文件，必须在 `Vue.use(ElementUI)` 之后重设 `$message` 才能生效

```JavaScript
import Vue from 'vue'
import ElementUI from 'element-ui'
import OffsetMessage from './offset-message'


Vue.use(ElementUI)

Vue.prototype.$message = OffsetMessage
```

## 调用调整

1、如果是通过全局方法 `$message` 的方式进行调用，则无需进行修改

2、如果是通过单独引用的方式进行调用，需要修改导入方式
```JavaScript
// import { Message } from 'element-ui'
import { Message } from './offset-message'

Message.success('OK')
```
