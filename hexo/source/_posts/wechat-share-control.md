---
title: 微信小程序转发按钮的显示与隐藏
tags:
    - 微信小程序
abbrlink: 122db714
date: 2018-12-12 09:18:27
---

在小程序就要上线之际，突然接到一个需求：只有登录的用户才能转发小程序。

在一接到这个需求的时候，我内心是很拒绝的，一时间也是一脸懵逼态，无从下手。在我痛定思痛之后，发现了一对 API `wx.showShareMenu`、`wx.hideShareMenu`，这对 API 能够显示/隐藏 **`当前`** 页面的转发按钮。

有了思路，也有了 API，那我只需要在 `onShow()` 方法中判断一下当前用户是否登录即可，登录就调用 wx.showShareMenu 显示转发按钮，未登录就调用 wx.hideShareMenu 隐藏转发按钮。

虽然我们的小程序并不是所有页面都可以转发，但是起码也有十几二十个页面，每个页面都来粘贴一下这代码，别说会被人鄙视了，就连自己看了都恶心啊，这不是作为一个合格的程序员的修养。

我们可以通过拦截系统的 onShow 方法统一处理，这里就要借助前面的一篇文章：[微信小程序扩展 Page 对象](/posts/cd0696c4/index.html)。

下面话不多说，直接上代码，如果看不懂的话，建议看下前面的文章。

```JavaScript
function initPageExtension(Page) {
    return (function (page) {

        const { onShow, onShareAppMessage } = page;

        page.onShow = function (options) {
            if (typeof onShow === 'function') {
                onShow.call(this, options);
            }
            if (Reflect.has(this, 'onShareAppMessage')) {
                // TODO: 判断登录状态
                if ('如果已登录') {
                    wx.showShareMenu();
                } else {
                    wx.hideShareMenu();
                }
            }
        }

        if (Reflect.has(page, 'onShareAppMessage')) {
            page.onShareAppMessage = function (options) {
                let result = onShareAppMessage.call(this, options);
                // 可以修改分享的标题或者给分享的path附加额外参数
                return result;
            }
        }

        return Page(page);
    });
}
```

上面的代码还有一点点小瑕疵，就是当前页面不能在登录状态发生改变时及时作出反应；当然也不是不能解决，可以参考这篇文章 [为微信小程序增加通知机制](/posts/2c6d003/index.html) 监听登录状态的变化情况。

注：由于 wx.showShareMenu 和 wx.hideShareMenu 控制的是当前页面的转发按钮，假设当前页面栈中有 A、B、C 三个页面，不管你在 A 或 B 中调用这对 API，控制的都是 C 页面的转发按钮，因此需要在每个页面的 onShow 方法中进行判断处理。
