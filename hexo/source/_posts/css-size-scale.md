---
title: CSS设置宽高比
date: 2019-11-26 14:40:49
tags:
	- CSS
---

```html
<view class="scale-box">
	<view class="scale-box-content">
		<!-- content -->
	</view>
</view>
```

```css
.scale-box {
	width: 100%;
	height: 0;
	padding-bottom: 100%;
	position: relative;
}
.scale-box-content {
	width: 100%;
	height: 100%;
	position: absolute;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
}
```

通过设置 scale-box 的 `padding-bottom` 来控制宽高比例，width 和 padding-botton 都为 `100%` 则表示正方形，宽度 100%、padding-bottom 50% 则表示宽高比为 2:1
