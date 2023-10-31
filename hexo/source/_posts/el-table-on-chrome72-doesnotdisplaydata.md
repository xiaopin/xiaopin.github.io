---
title: Element-Plus 的表格组件(el-table)在 Chrome 72/73 的版本下不显示数据
abbrlink: 391caf7c
date: 2023-10-30 11:08:53
tags:
    - Vue
---

## 前言

刚接手的项目在上周遇到个问题，同事反馈说在 Chrome 浏览器下，`el-table` 表格部分内容不显示，显示为空白。

第一时间就在我电脑上进行测试，发现一切没问题啊，显示正常，但是在同事电脑上确实未显示，控制台也无任何报错信息。

那就只能先排查浏览器兼容性问题了，先看下同事电脑上 Chrome 的版本，好家伙，`73.0.3642.0`的版本，这么老（我电脑用的是 107 的版本，公司内部网络环境，与互联网隔绝，我猜同事的电脑在第一次安装浏览器后应该也没再更新过了），看了 [ElementPlus](https://element-plus.org/zh-CN/guide/installation.html#%E7%8E%AF%E5%A2%83%E6%94%AF%E6%8C%81) 的环境支持，要求 Chrome 版本必须 `≥64`，那 73 的版本也符合要求啊。

但是并不是整个表格都显示为空白，因为部分 `el-table-column` 我使用了插槽，这部分表格列是能够正常显示的，那是不是通过插槽就能避免这个问题呢，从测试结果来看，插槽确实能够有效避免该问题的发生，但是为了兼容可能不会有人在用的 `Chrome 7x` 版本就要把所有表格的使用都改一遍，那这种修复方案是不能接受的，并且还会有后续的隐患。

## 解决方案

虽然通过插槽的方式能够避免该问题的发生，但是这是治标不治本的方案，不推荐。

然后在 [`issues`](https://github.com/element-plus/element-plus/issues) 中搜索相关问题，发现很早就有人给官方提了 issues，并且官方也已经在 [`2.3.10`](https://github.com/element-plus/element-plus/releases/tag/2.3.10) 版本中修复了该问题。

回看我们项目中引用的版本为 `2.3.6`，哈哈哈，还有什么好说的呢，更新版本就完事了。

已更新至目前最新版 `2.4.1`，问题解决。

> 终极方案：更新 element-plus 版本至 ≥2.3.10 即可！！！

## 参考

- [[Component] [table-column] el-table-column组件使用prop属性绑定值无效，使用默认插槽则可以](https://github.com/element-plus/element-plus/issues/9071)
- [fix(components): fix the problem that chrome 72 table doesnotdisplaydata](https://github.com/element-plus/element-plus/pull/10640)
