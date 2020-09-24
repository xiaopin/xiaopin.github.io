---
title: 通过 DOMSubtreeModified 事件接收 DOM 节点变化的回调
tags:
  - JavaScript
abbrlink: f668727b
date: 2019-11-12 17:23:19
---

需求场景：通过 WKWebView 加载第三方网页，其中有一个录制人脸视频的功能，为了用户体验需要默认调用前摄像头，然而第三方的网页我们无法调整，只能通过注入 JS 来修改 input 标签，将 capture 属性改成 user 即可调用前摄像头。

网页内容是动态生成的，并不是写死的模板，所以需要监听 body 子节点发生改变的事件，每当 body 内容发生变更时，就去找出调用摄像头的 input 标签，找到之后动态修改它的属性。

```JavaScript
function findVideoInput() {
    var inputs = document.getElementsByTagName('input');
    for (var i = 0; i < inputs.length; i++) {
        var input = inputs[i];
        var type = input.getAttribute('type').toString().toLowerCase();
        var accept = input.getAttribute('accept');
        if (type == 'file' && /^video/i.test(accept)) {
            document.querySelector('body').removeEventListener('DOMSubtreeModified', findVideoInput, false);
            input.setAttribute('capture', 'user');
        }
    }
}
document.querySelector('body').addEventListener('DOMSubtreeModified', findVideoInput, false);
setTimeout(findVideoInput, 0);
```

虽然 [DOMSubtreeModified](https://www.w3.org/TR/DOM-Level-3-Events/#event-type-DOMSubtreeModified) 已经声明为 `Deprecated` 了，但是目前来说还是可以用的；建议使用 [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) 来代替 DOMSubtreeModified。