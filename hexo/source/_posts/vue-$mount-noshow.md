---
title: 
  You are using the runtime-only build of Vue where the template compiler is not
  available. Either pre-compile the templates into render functions, or use the
  compiler-included build.
tags:
  - JavaScript
  - Vue
abbrlink: 3df0b138
date: 2021-10-13 11:06:03
---

需要指定使用 `vue/dist/vue.esm.js` 完整版。

```JavaScript
import Vue from "vue/dist/vue.esm.js";
```

```JavaScript
const path = require("path");

function resolve(dir) {
    return path.join(__dirname, dir);
}

module.exports = {
    configureWebpack: (config) => {
        config.resolve = {
            extensions: [".js", ".vue", ".json"],
            alias: {
                vue$: "vue/dist/vue.esm.js",
                "@": resolve("src"),
            },
        };
    },
};
```