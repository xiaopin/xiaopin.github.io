---
title: 微信小程序扩展 Page 对象
date: 2018-12-11 10:41:51
tags:
	- 微信小程序
---

> 对于 iOS 开发者来说，如果需要对一个类进行扩展，第一反应就是 `runtime` + `category`，有了这组合，我们简直可以为所欲为，为类添加属性、方法、重写某个方法都不在话下。

在小程序中是通过 `Page()` 函数来注册页面的，而 page 对象就是我们所熟知的控制器。如果我们重写了系统的 Page() 函数，并修改所传递进来的 page 对象，那我们不就是达到了扩展的目的了吗。

## 源码

1、新建一个 js 文件，文件名随意（假设叫做 `page_extension.js`），将下面的代码粘贴到该文件中

```JavaScript
/**
 * @method 扩展 Page 对象
 * 
 * @param Page {Object} 系统原始的Page对象
 * @return {function} 返回新的Page构造器
 */
function initPageExtension(Page) {
    
    // 返回一个新的 Page 构造器
    return (function (page) {

        // 获取需要hook的方法, 保存方法的原始实现
        const { onLoad, onUnload, onShow } = page;

        /**
         * @method 重写 onLoad 方法
         */
        page.onLoad = function (options) {
            // 可以在此直接加载模块, this.moduleA = xxxx;
            if (typeof onLoad === 'function') {
                onLoad.call(this, options);
            }
        }

        /// 可以为 page 扩展任意方法, 注意别和系统的方法重名即可
        page.testFunction = function() {
            console.log('test function.');
        }

        // 调用原来的构造方法
        return Page(page);
    });
}

// 扩展 Page 对象
const originalPage = Page;
Page = initPageExtension(originalPage);
```

2、在 `app.js` 中引入该文件即可。

```JavaScript
require('./utils/page_extension.js');
```

## 总结

通过重写 Page() 方法，拿到每个页面的 page 实例对象，我们就可以根据需求做各类扩展了:

- 重写 Page 系统方法，做特定的逻辑处理
- 为 Page 统一增加某个功能函数
- 统一导入某个模块，避免每个js文件都导入一遍，过于繁琐
- ...

更多用途你可以自由发挥哈，我只能帮你到这了 :)