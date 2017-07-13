---
title: UIScrollView计算当前滚动页码
date: 2017-07-13 11:32:10
tags:
	- iOS
---

我们需要在UIScrollView滚动时根据滚动偏移量实时计算当前滚动到第几页，需要实现`scrollViewDidScroll(_:)`这个代理方法，记得开启分页功能`scrollView.isPagingEnabled = true`

```Swift
func scrollViewDidScroll(_ scrollView: UIScrollView) {
    let pageWidth = scrollView.bounds.width
    let offsetX = scrollView.contentOffset.x
    let currentPage = Int(floor((offsetX - pageWidth / 2) / pageWidth)) + 1
}
```