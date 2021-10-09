---
title: Vue 导出 Excel
abbrlink: 65afa4a2
date: 2021-10-09 10:30:04
tags:
    - Vue
    - JavaScript
---

最近做了一个和 Android 设备机进行交互的 Web 项目，基本原理是在 Android 上通过 [AndServer](https://github.com/yanzhenjie/AndServer) 搭建一个 HTTP 服务器，然后把 Vue 项目打包进去，这样就可以通过网页和 Android 设备进行交互，配置一些常用选项、查看数据、导出数据。

这里主要就是介绍一下导出 Excel 的功能。

## 安装xlsx

导出 Excel 功能，只需要安装 [xlsx](https://github.com/SheetJS/sheetjs) 这一个库即可。

```shell
npm i xlsx
```

## 导出数据源

这里额外提一下，由于是局域网内的场景（作为厂区门口门禁机的 Android 设备，以及在内网中访问的 Web 服务），所以网络的传输速率不在考虑范围内，有些数据量会比较大（比如刷卡数据，可能达到几十万笔的记录），所以采取的是前端分页加载的策略，每页加载少量数据，先把所有数据按照分页方式加载回来，然后再统一进行处理并导出。

- 导出前需要对数据先进行格式化，xlsx 导出的数据，需要的是一个二维数组的格式

```JavaScript
const exportData = [
    ['姓名', '年龄', '性别'],
    ['张三', 18, '男'],
    ['李四', 19, '女'],
    ['王五', 20, '男'],
]
```

- 导出数据 & 生成Excel

```JavaScript
import XLSX from 'xlsx'

const sheetSize = 10000 // 每一个 Sheet 的数据量
const sheetTitles = exportData.shift() // 数组中第一个元素为表头字段
const sheetCount = Math.ceil(exportData.length / sheetSize)
const book = XLSX.utils.book_new()
for (let i = 0; i < sheetCount; i++) {
    const begin = i * sheetSize
    const end = begin + sheetSize
    const sheetData = exportData.slice(begin, end)
    sheetData.unshift(sheetTitles) // 为每个 Sheet 添加表头
    const sheet = XLSX.utils.aoa_to_sheet(sheetData)
    XLSX.utils.book_append_sheet(book, sheet, `Sheet${i + 1}`)
}
XLSX.writeFile(book, 'export_data.xlsx')
```

至此，就能将数据导出为Excel！

## 导出DOM元素

有时候我们不想请求数据，而是只需要将页面上的 table 元素数据直接导出，能不能做到呢？

答案是：能！！！

```JavaScript
import XLSX from 'xlsx'

const dom = this.$refs.tableRef.$el // document.getElementById('#tableId')
const sheet = XLSX.utils.table_to_sheet(dom)
const book = XLSX.utils.book_new()
XLSX.utils.book_append_sheet(book, sheet, 'Sheet1')
XLSX.writeFile(book, 'export_dom.xlsx')
```

这里使用的是 element-ui 的 el-table