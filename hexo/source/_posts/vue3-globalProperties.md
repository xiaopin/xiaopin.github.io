---
title: 在 Vue3 中挂载全局属性
abbrlink: f651d15a
date: 2022-04-29 15:29:48
tags:
    - Vue
    - Vue3
---

## 定义全局属性

在 Vue2 中， Vue.prototype 通常用于添加所有组件都能访问的 property。

```JavaScript
Vue.prototype.$http = axios
Vue.prototype.$name = 'xiaoming'
Vue.prototype.$print = function(d) {
    console.log(d)
}
```

但是在 [Vue3](https://v3.cn.vuejs.org) 中这种方式被废弃了，与之对应的是 `config.globalProperties`。这些 property 将被复制到应用中，作为实例化组件的一部分。

```JavaScript
const app = createApp({})
app.config.globalProperties.$http = axios
app.config.globalProperties.$name = 'xiaoming'
app.config.globalProperties.$print = function(d) {
    console.log(d)
}
```

## TypeScript 语法提示

如果项目中使用了 TypeScript，为了告诉 TypeScript 这些新 property，我们可以使用 [模块扩充(module augmentation)](https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation)，

在上述示例中，我们可以添加以下类型声明(在项目中新建一个 d.ts 声明文件)：

```TypeScript
import axios from 'axios'
declare module '@vue/runtime-core' {
  export interface ComponentCustomProperties {
    $http: typeof axios
    $name: string
    $print: (data: any) => void
  }
}
```

## 通过插件形式扩展(个人推荐做法)

可在项目中新建 `plugins/VueEnhance.ts` 插件文件，然后编写插件内容：

```TypeScript
import axios from 'axios'

export default {
    install: function (app: App<Element>, options: any): void {
        const properties = app.config.globalProperties
        properties.$http = axios
        properties.$name = 'xiaoming'
        properties.$print = function(d) {
            console.log(d)
        }
    }
}
```

添加声明文件，提供 TypeScript 语法提示

```TypeScript
import { createApp } from 'vue'

declare module '@vue/runtime-core' {
    interface ComponentCustomProperties {
        $api: typeof import('axios')['default']
        $name: string
        $print: (data: any) => void
    }
}
```

最后在 main.ts 中使用插件

```TypeScript
import { createApp } from 'vue'
import App from './App.vue'
import VueEnhance from './plugins/VueEnhance'

createApp(App).use(VueEnhance).mount('#app')
```

## 使用 properties

前面我们已经定义好了自己的功能扩展，现在就开始在项目中使用吧。

### 在组合式API（Composition API）中使用

在组合式API中，TypeScript 能正确推导出 this 的类型，所以可以直接通过 `this.$xxx` 的形式来访问。

```TypeScript
<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
    setup() {},
    methods: {
        test() {
            console.log(this.$name)
        }
    }
})
</script>
```

### 在 script-setup 中使用

由于 getCurrentInstance 返回值的类型为 `ComponentInternalInstance | null`，所以我们需要先强制转成 ComponentInternalInstance，这样在使用 proxy 的时候就能得到 TypeScript 的语法提示了。

```TypeScript
<script lang="ts" setup>
import { ComponentInternalInstance, getCurrentInstance } from 'vue'

const { proxy } = getCurrentInstance() as ComponentInternalInstance
console.log(proxy?.$name)
```

## 参考文档

- [globalproperties](https://v3.cn.vuejs.org/api/application-config.html#globalproperties)
- [为 globalProperties 扩充类型](https://v3.cn.vuejs.org/guide/typescript-support.html#为-globalproperties-扩充类型)
- [Vue.prototype 替换为 config.globalProperties](https://v3.cn.vuejs.org/guide/migration/global-api.html#vue-prototype-替换为-config-globalproperties)
