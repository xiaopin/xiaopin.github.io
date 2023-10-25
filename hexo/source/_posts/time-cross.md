---
title: JavaScript 判断时段是否有交集
abbrlink: e8154a51
date: 2022-10-02 23:02:34
tags:
    - JavaScript
---

在做排班功能时，有个需求就是对同一个人员在同一天时间内，所排班次的时间不允许存在有交集。

假设有三个班次：A(08:00-11:30)、B(09:00-12:00)、C(13:30-18:00)，对于员工张三来说，如果在同一天内排了A班次则不允许排B班次，因为这两个班次在时间上存在交集（09:00-11:30），一个人不可能同时上多个班次。

- util.js

```js
export default class {
    /**
     * 判断给定的时段是否有交集
     * 假定所有的时段均在同一天内, 如需判断跨天, 则先将多个时段按天划分后再调用该方法进行判断
     *
     * @param {Array} array 二维数组(每个数组由开始时间/结束时间两个元素组成, [['01:00', '02:00'], ['03:00', '04:00']])
     * @returns {Boolean} true:多个时段之间有交集 false:时段无交集
     */
    static isTimeCross(array) {
        if (Array.isArray(array) && array.length > 1) {
            const length = array.length
            array = array.map(arr => arr.map(time => parseInt(String(time).replace(':', '')) || 0))
            for (let i = 0; i < length; i++) {
                for (let j = 0; j < length; j++) {
                    if (i == j) continue
                    const [s1, e1] = array[i]
                    const [s2, e2] = array[j]
                    if (s1 < e2 && s2 < e1) {
                        return true
                    }
                }
            }
        }
        return false
    }
}

```

- 使用示例

```js
import Util from '@/util.js'

export default {
    mounted: function() {
        // 根据所选班次获取到所有班次的时间
        const times = [['08:00', '12:00'], ['13:30', '17:30']]
        // 判断所排班次是否有交集
        const flag = Util.isTimeCross(times)
        if (flag) {
            console.log('所排班次存在时间交集问题，请重新排班')
        } else {
            console.log('排班成功')
        }
    }
}
```
