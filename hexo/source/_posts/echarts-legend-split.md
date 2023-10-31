---
title: echarts 将图例(legend)拆分成左右两栏
abbrlink: d539db81
date: 2023-10-31 15:16:45
tags:
    - echarts
    - Vue
---

![](/images/2023/echarts-legend.png)

在多数情况下，我们的图形报表中要么不显示图例组件，要么只是将图例组件固定在上/下/左/右的某一个方向上，但是现在有个需求，是需要将图例组件进行拆分成两栏，分别在左右两侧进行展示。

## 测试环境

- Vue 3.x
- echarts 5.x
- TypeScript

## 准备测试数据

```TypeScript
const data: Array<{ name: string; value: number }> = new Array(7)
    .fill(0)
    .map((_, i) => {
        return { name: `随机数${i + 1}`, value: Math.ceil(Math.random() * 1000) + 100 }
    })
    .sort((a, b) => b.value - a.value)
const total = data.map((element) => element.value).reduce((a, b) => a + b, 0)
```

`data` 就是我们的图表数据，是一个一维对象数组，每个对象包含 `name` 和 `value` 两个属性，value 是一个随机产生的数值，`total` 是所有 value 的总和，用于后续计算百分比。

我们需要展示的是一个`南丁格尔玫瑰图`，递减或递增的数据展示出来的图表才好看，所以对 data 进行了排序处理。

## 图例数据

echarts 的 `legend` 配置项可以是单个对象，也可以是一个对象数组，既然我们想要将图例拆分成左右两栏，那就必须是一个数组了，数组的第一个元素是左侧的图例，第二个元素是右侧图例。

```TypeScript
[0, 1].map((i) => {
    const mid = Math.ceil(data.length / 2)
    const legendItem: echarts.LegendComponentOption = {
        orient: 'vertical',
        data: (i === 0 ? data.slice(0, mid) : data.slice(mid)).map((element) => element.name),
        icon: 'rect',
        itemWidth: 10,
        itemHeight: 10,
        itemGap: 24,
        top: 'center',
        formatter: (name) => {
            const item = data.find((element) => element.name === name)
            if (item) {
                const percentage = ((item.value / total) * 100).toFixed(2).replace(/\.0+$/, '')
                return `{name|${item.name}}{value|${percentage}%}`
            }
            return name
        },
        textStyle: {
            color: 'inherit',
            rich: {
                name: {
                    align: 'left',
                    width: 70,
                    fontSize: 14
                },
                value: {
                    align: 'right',
                    width: 50,
                    fontSize: 14
                }
            }
        }
    }
    if (i === 0) {
        legendItem.left = 'left' // 居左显示
    } else {
        legendItem.right = 'right' // 居右显示
    }
    return legendItem
})
```

通过 `mid` 将数据拆分成两份，然后再分别设置 `left` 和 `right` 属性，使其居左/居右显示。最后通过 `formatter` 格式器和 `textStyle` 的富文本样式来自定义图例上的文本内容，通过 `formatter` 将百分比显示出来，然后通过 `textStyle.rich` 来设置文本的对齐方式。

## series

最后通过 `series` 属性来设置我们的图表样式。

```TypeScript
[
    {
        type: 'pie',
        radius: ['30%', '50%'],
        center: ['50%', '50%'],
        roseType: 'area',
        label: {
            show: true,
            color: 'inherit',
            fontSize: 16,
            fontWeight: 'bold',
            formatter: '{c}'
        },
        data: data
    }
]
```

## 完整示例

```html
<script setup lang="ts">
import { onMounted } from 'vue'
import * as echarts from 'echarts'

onMounted(() => {
    const data: Array<{ name: string; value: number }> = new Array(7)
        .fill(0)
        .map((_, i) => {
            return { name: `随机数${i + 1}`, value: Math.ceil(Math.random() * 1000) + 100 }
        })
        .sort((a, b) => b.value - a.value)
    const total = data.map((element) => element.value).reduce((a, b) => a + b, 0)

    const options: echarts.EChartsOption = {
        legend: [0, 1].map((i) => {
            const mid = Math.ceil(data.length / 2)
            const legendItem: echarts.LegendComponentOption = {
                orient: 'vertical',
                data: (i === 0 ? data.slice(0, mid) : data.slice(mid)).map((element) => element.name),
                icon: 'rect',
                itemWidth: 10,
                itemHeight: 10,
                itemGap: 24,
                top: 'center',
                formatter: (name) => {
                    const item = data.find((element) => element.name === name)
                    if (item) {
                        const percentage = ((item.value / total) * 100).toFixed(2).replace(/\.0+$/, '')
                        return `{name|${item.name}}{value|${percentage}%}`
                    }
                    return name
                },
                textStyle: {
                    color: 'inherit',
                    rich: {
                        name: {
                            align: 'left',
                            width: 70,
                            fontSize: 14
                        },
                        value: {
                            align: 'right',
                            width: 50,
                            fontSize: 14
                        }
                    }
                }
            }
            if (i === 0) {
                legendItem.left = 'left'
            } else {
                legendItem.right = 'right'
            }
            // legendItem.left = i === 0 ? 'left' : 'right'
            return legendItem
        }),
        tooltip: {
            trigger: 'item',
            formatter: '{b}: {c}({d}%)'
        },
        series: [
            {
                type: 'pie',
                radius: ['30%', '50%'],
                center: ['50%', '50%'],
                roseType: 'area',
                label: {
                    show: true,
                    // 5.x以上版本需要显式设置为 inherit，否则标签颜色将显示为黑色
                    color: 'inherit',
                    fontSize: 16,
                    fontWeight: 'bold',
                    formatter: '{c}'
                },
                data: data
            }
        ]
    }
    const chartInstance = echarts.init(document.getElementById('chart')!)
    chartInstance.setOption(options)
})
</script>

<template>
    <div id="chart" style="width: 650px; height: 300px"></div>
</template>

<style lang="scss"></style>
```

## left/right属性的差异

在本例中，图例的居左/居右显示是通过设置 `left: 'left'`/`right: 'right'` 来实现的，其实仅靠设置 `left` 属性也是可以实现的，但是如果按照 `left: 'right'` 这样将图例进行居右显示，不仅是图例整体居右，还会影响到图例的图标位置，但是通过分别设置 `left` 和 `right` 属性的话，则图例的图标一直都是显示在左侧。

![](/images/2023/echarts-legend-left.png)

![](/images/2023/echarts-legend-left-right.png)