---
title: "TypeError: this.getOptions is not a function"
abbrlink: ae57eb29
date: 2021-09-28 09:59:27
tags:
    - Vue
---

昨天在把 uni-app 小程序项目从 HBuilderX 开发方式迁移到 VSCode 中来，当把项目依赖都安装好，执行 `npm run dev:mp-weixin` 开始编译到微信小程序平台时，终端报了以下错误信息：

```
 ERROR  Failed to compile with 1 
 
 error  in ./src/pages/flag/insert.vue?vue&type=style&index=0&lang=scss&

TypeError: this.getOptions is not a function
    at runMicrotasks (<anonymous>)


 @ ./src/pages/flag/insert.vue?vue&type=style&index=0&lang=scss& 1:0-840 1:856-859 1:861-1698 1:861-1698
 @ ./src/pages/flag/insert.vue
 @ ./src/main.js?{"page":"pages%2Fflag%2Finsert"}

 ERROR  Build failed with errors.
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! vbemall-mp@0.1.0 dev:mp-weixin: `cross-env NODE_ENV=development UNI_PLATFORM=mp-weixin vue-cli-service uni-build --watch`
npm ERR! Exit status 1
npm ERR! 
npm ERR! Failed at the vbemall-mp@0.1.0 dev:mp-weixin script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     ~/.npm/_logs/2021-09-27T06_38_01_801Z-debug.log
```

关键错误信息：`TypeError: this.getOptions is not a function`

搜索了一下，都是说由于 sass-loader / less-loader 版本过高导致的，想到我刚才安装了 sass-loader，并且是 12.1.0 的版本。那我把版本降下来就好了。

```
npm uninstall sass-loader
npm i sass-loader@10.1.1 -D
```

把 sass-loader 版本降到 10.1.1 版本就好了，问题解决。