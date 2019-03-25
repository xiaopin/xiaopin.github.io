---
title: setTimeout 的时间限制
date: 2019-03-25 09:40:48
tags:
	- JavaScript
---

函数定义：
```JavaScript
setTimeout(function, millisec);
```

对于 setTimeout 函数的说明，不管是 [w3cschool](http://www.w3school.com.cn/htmldom/met_win_settimeout.asp) 还是 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/setTimeout) 文档都未提及第二个参数 `millisec` 的取值范围，只是简单地说明该参数是一个需要延时执行的毫秒数。

最近就遇到了个问题，有个活动需要在活动结束后告诉用户当前活动已结束(延时执行)，当代码都写好之后，在Android端可以正常跑起来，但是到了iOS端之后一跑起来就立马提示活动已结束（也就是执行了 setTimeout 的函数代码块）。
当时真是一头雾水，后来才发现是 setTimeout 的 millisec 参数有问题，millisec 参数是一个 Int32 的类型，最大值为 `2^32 - 1`，即 `2147483647`。一旦超过这个限制，那么 millisec 参数将被视为 `0` 也就意味着代码会被立马执行。

结论：

setTimeout 的 millisec 参数的取值范围：0 ～ 2147483647。