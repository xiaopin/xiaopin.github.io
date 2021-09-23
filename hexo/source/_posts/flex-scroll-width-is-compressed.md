---
title: Flex 设置横向滚动后元素宽度被压缩
abbrlink: 971f15fe
date: 2021-09-23 15:52:07
tags:
 - CSS
---

当给一个容器采用 flex 布局并且设置横向滚动的时候，当所有子元素的内容超出父元素的宽度时，子元素的宽度被压缩了，从而导致变形。

需要给父元素设置 `overflow-x: auto;` 属性，然后子元素设置 `flex-shrink: 0;` 属性。

```HTML
<div class="parent">
    <div class="children">xxx</div>
    <div class="children">yyyy</div>
    <div class="children">wwwww</div>
    <div class="children">zzzzzz</div>
</div>
```

```CSS
.parent {
    width: 500px;
    height: 40px
    display: flex;
    align-items: center;
    flex-wrap: nowrap; // 不换行
    overflow-x: auto; // 横向滚动
}
.children {
    width: auto; // 这里假设每个子元素的宽度跟随内容进行变化, 即每个子元素宽度不相同
    height: 100%;
    flex-shrink: 0; // 禁止宽度被压缩
}
```
