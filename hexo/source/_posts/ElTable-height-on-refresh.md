---
title: el-table 在刷新页面后高度显示异常的问题
abbrlink: '3101281'
date: 2022-05-13 17:16:39
tags:
    - Vue
---

最近在维护一个老项目的时候，发现这么一个有趣的问题，页面中使用了 `el-table` 这个组件来展示数据并且通过 `height` 属性设置了固定高度，如果是正常的点击跳转，表格高度也能正确地显示，但是只要你一刷新页面后，表格高度就会出现异常了，只能显示一条数据的高度。

项目中的使用代码如下：
```Vue
<template>
    <el-table :height="tableHeight">
        ...
    </el-table>
</template>

<script>
export default {
    data() {
        return {
            tableHeight: document.body.offsetHeight - 100 + 'px' // 错误的使用方式
        }
    }
}
</script>
```

可以看到我们在 `data` 方法中直接计算了表格高度，如果是正常的点击跳转打开的组件，是能够正确获取到页面高度的，但是刷新的话，`document.body.offsetHeight` 的高度就会为 `0`，所以解决的办法就是我们把高度的计算放到 `mounted` 生命周期中，确保组件挂载了再来计算高度，这样就能正确获取到页面高度了。

所以修复后的代码如下：
```Vue
<template>
    <el-table :height="tableHeight">
        ...
    </el-table>
</template>

<script>
export default {
    data() {
        return {
            tableHeight: '0px'
        }
    },

    mounted() {
        // 延迟获取页面高度, 确保能正确返回
        this.tableHeight = document.body.offsetHeight - 100 + 'px'
    }
}
</script>
```

以上就是针对这个小问题的修复方案，当然如果想要完美的话，还是要记得监听窗口的大小变化，实时计算高度。
