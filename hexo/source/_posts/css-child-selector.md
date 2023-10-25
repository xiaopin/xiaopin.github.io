---
title: CSS 子元素选择器（first-child/last-child/nth-child/nth-last-child）
abbrlink: c1279cc7
date: 2023-10-25 13:41:40
tags:
    - CSS
---
虽然从事前端这么久了，但是也一直没怎么写过关于 CSS 的文章，今天我们就聊聊 CSS 子元素选择器。

开始之前，我们先创建一个初始化场景页面，以下是我们的示例开始的代码：

```html
<script setup lang="ts">
const items = new Array(15).fill(0).map((_, idx) => idx)
</script>

<template>
  <div class="list-container">
    <div class="list-item" v-for="item in items" :key="item">{{ item + 1 }}</div>
  </div>
</template>

<style>
.list-container {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  background-color: #E6E6FA;
}
.list-item {
  width: calc(20% - 24px);
  height: 44px;
  margin: 10px 0 10px 20px;
  font-size: 14px;
  color: white;
  background-color: #D3D3D3;
  display: flex;
  align-items: center;
  justify-content: center;
  box-sizing: border-box;
}
</style>
```

初始效果如下图所示：

![](/images/2023/ccs.png)

> 注意：CSS 的索引是从 `1` 开始的

## first-child

`:first-child` 表示选择列表中的第一个元素，比如我们需要将列表中第一个 `.list-item` 背景改为红色，文字颜色改为黑色，则代码如下：

```css
.list-item:first-child {
  background-color: red;
  color: black;
}
```

![](/images/2023/ccs1.png)

## last-child

`:last-child` 表示选择列表中的最后一个元素，比如我们需要将列表中最后一个 `.list-item` 背景改为绿色，文字改为红色，则代码如下：

```css
.list-item:last-child {
  background-color: green;
  color: red;
}
```

![](/images/2023/ccs2.png)

## nth-child()

1、括号内填写具体数字，表示选择列表中第 N 个

比如我想要列表第三个元素的字体大小为28px、粗体，背景为红色，则代码如下：

```css
.list-item:nth-child(3) {
  background-color: red;
  font-size: 28px;
  font-weight: bold;
}
```

![](/images/2023/ccs3.png)

> 如果我们填写了 1，不就代表选择列表中的第一个元素了吗，所以 `:nth-child(1)` 等价于 `:first-child`

2、nth-child(2n)

`2n` 表示选择列表中的偶数元素，即选择第 2、4、6、8、10... 个元素，比如我们需要给偶数列添加一个蓝色边框，代码如下：

```css
.list-item:nth-child(2n) {
  border: 4px solid #1E90FF;
}
```

![](/images/2023/ccs4.png)

3、nth-child(2n-1)

`2n-1` 表示选择列表中的奇数元素，即选择第 1、3、5、7、9... 个元素，比如我们需要给偶数列添加一个紫色背景，代码如下：

```css
.list-item:nth-child(2n-1) {
  background-color: #8B008B;
}
```

![](/images/2023/ccs5.png)

4、nth-child(n+4)

`n+4` 表示选择列表中元素从第 4 个开始到最后一个(包含开始索引位置，即 `≥4`)

```css
.list-item:nth-child(n+4) {
  background-color: red;
}
```

![](/images/2023/ccs6.png)

5、nth-child(-n+4)

`-n+4` 表示选择列表中的从第 1 个到第 4 个，即选中前 4 个元素(包含结束的索引位置，即 `≤4`)，根据实际情况你需要选择前几个就把数字改成几即可。

```css
.list-item:nth-child(-n+4) {
  background-color: red;
}
```

![](/images/2023/ccs7.png)

## nth-last-child()

1、括号内填写具体数字，表示选择列表中倒数第 N 个

比如我想将列表中倒数第 2 个的背景色改成红色，则代码如下：

```css
.list-item:nth-last-child(2) {
  background-color: red;
}
```

![](/images/2023/ccs8.png)

> 如果我们填写了 1，不就代表选择列表中的最后一个元素了吗，所以 `:nth-last-child(1)` 等价于 `:last-child`

2、2n/2n-1 奇偶数元素

`nth-last-child(2n)` 等价于 `nth-child(2n)`，`nth-last-child(2n-1)` 等价于 `nth-child(2n-1)`。

3、nth-last-child(n+4)

`n+4` 表示选择列表中顺数第 1 个到**倒数**第 4 个（含）元素

```css
.list-item:nth-last-child(n+4) {
  background-color: red;
}
```

![](/images/2023/ccs9.png)

4、nth-last-child(-n+4)

`-n+4` 表示选择列表中的从**倒数**第 1 个到**倒数**第 4 个，即选中最后 4 个元素，根据实际情况你需要选择最后几个就把数字改成几即可。

```css
.list-item:nth-last-child(-n+4) {
  background-color: red;
}
```

![](/images/2023/ccs10.png)


