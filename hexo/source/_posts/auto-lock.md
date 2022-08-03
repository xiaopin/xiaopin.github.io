---
title: Web自动锁屏功能
abbrlink: 37bef8ad
date: 2022-07-22 11:10:43
tags:
  - TypeScript
  - JavaScript
---

Web项目中有一个自动锁屏的功能，当用户停止操作页面超过一定时间后，自动锁屏。

## 场景分析

首先我们确定下用户的操作有哪些：
- 鼠标点击
- 键盘按键
- 鼠标滚轮滚动
- 鼠标在页面中滑动
- 浏览器窗口之间切换(`visibilitychange`)

其中
- 鼠标点击时会触发`mousedown`、`mouseup`、`click`事件
- 键盘按键输入会触发`keydown`、`keyup`、`keypress`事件
- 滚轮滚动会触发`wheel`事件
- 鼠标移动会触发`mousemove`事件

我们也不需要每个事件都监听，一种操作监听一个事件即可，比如鼠标点击事件，我们监听鼠标按下(mousedown)事件即可，当鼠标按下时我们认为用户操作了页面，其他事件(mouseup、click等)我们则不处理。综上所述，我们只需要监听`mousedown`、`keydown`、`wheel`、`mousemove`这四个事件即可，当用户有相关操作后，我们及时更新保存用户的最后操作时间。

~~别问我为什么是 mousedown 而不是 mouseup，你可以按自己喜好来，甚至你两者都监听也没问题。~~

## 源码

将下面代码保存为 `AutoLock.ts`

```ts
/// 需要监听的事件
const EVENT_NAMES: (keyof WindowEventMap)[] = ['mousedown', 'mousemove', 'keydown', 'wheel'];

export type AutoLockOption = {
    /** 自动锁定时间(毫秒, 默认5分钟) */
    time: number;
};

export default class AutoLock {
    private isLocked: boolean = false;
    private timer: number | undefined = undefined;
    private latestTime: number | undefined = undefined;
    private option: AutoLockOption;

    constructor(option?: AutoLockOption) {
        /**
         * 为了在回调中能够正确使用`this`, 这里需要进行绑定, 否则回调方法中的`this`将会指向`window`对象
         * 如果不明白, 可以参考React的事件处理章节中的相关说明(https://zh-hans.reactjs.org/docs/handling-events.html)
         */
        this.resetTimer = this.resetTimer.bind(this);
        this.handleTimerCallback = this.handleTimerCallback.bind(this);
        this.handleVisibilityChange = this.handleVisibilityChange.bind(this);

        const defaultOption: AutoLockOption = { time: 5 * 60 * 1000 };
        this.option = Object.assign(defaultOption, option);
        this.addListener();
        document.addEventListener('visibilitychange', this.handleVisibilityChange);
    }

    destroy(): void {
        document.removeEventListener('visibilitychange', this.handleVisibilityChange);
        this.removeListenner();
    }

    /** 锁定(用于用户手动锁定页面时调用该方法停止计时) */
    lock(): void {
        this.isLocked = true;
        this.removeListenner();
    }

    /** 解锁(当用户解锁页面时调用该方法重新进行计时) */
    unlock(): void {
        this.isLocked = false;
        this.addListener();
    }

    /** 添加事件监听器并启动计时器 */
    private addListener(): void {
        EVENT_NAMES.forEach(element => window.addEventListener(element, this.resetTimer));
        this.startTimer();
        this.latestTime = new Date().getTime();
    }

    /** 移除事件监听器并停止计时器 */
    private removeListenner(): void {
        EVENT_NAMES.forEach(element => window.removeEventListener(element, this.resetTimer));
        this.clearTimer();
        this.latestTime = undefined;
    }

    /** 页面显示/隐藏的回调 */
    private handleVisibilityChange(): void {
        if (this.isLocked) return;
        if (document.visibilityState === 'visible') {
            this.startTimer();
            this.handleTimerCallback();
        } else {
            this.clearTimer();
        }
    }

    /** 事件回调, 更新最后操作时间 */
    private resetTimer(event: Event): void {
        if (this.isLocked) return;
        this.latestTime = new Date().getTime();
    }

    /** 发送`autolock`事件通知外界进入自动锁定 */
    private dispatchLockEvent(): void {
        const event = new CustomEvent('autolock', { detail: this });
        window.dispatchEvent(event);
    }

    /** 计时器的回调事件 */
    private handleTimerCallback(): void {
        if (this.isLocked || !this.latestTime) return;
        const isOvertime = new Date().getTime() - this.latestTime >= this.option.time;
        if (isOvertime) {
            this.lock();
            this.dispatchLockEvent();
        }
    }

    /** 开启计时器 */
    private startTimer(): void {
        this.clearTimer();
        this.timer = setInterval(this.handleTimerCallback, 1000);
    }

    /** 清除计时器 */
    private clearTimer(): void {
        if (this.timer) {
            clearInterval(this.timer);
            this.timer = undefined;
        }
    }
}
```

## 使用示例

在 `App.vue` 中进行使用

- 导入模块
```ts
import AutoLock from './AutoLock.ts'
```

- 实例化AutoLock
```ts
this.autoLock = new AutoLock({ time: 3 * 60 * 1000 })
```

- 监听 `autolock` 进入锁定事件
```ts
// JavaScript
window.addEventListener('autolock', (e) => {})

// TypeScript可以使用该方法进行监听, 避免语法报错
window.addEventListener('autolock', {
    handleEvent: (e: CustomEvent) => {
        // TODO: 进入锁屏, 你可以自行处理UI上的逻辑, 在body添加一层遮罩或者跳转到对应的锁屏页面

        // 当用户解锁页面后, 记得调用`unlock`方法重新开启计时即可
        setTimeout(() => {
            this.autoLock.unlock()
        }, 5000)
    }
})
```

- 在 App 卸载时销毁 AutoLock 对象
```ts
this.autoLock.destroy()
```

> 当用户成功解锁页面后，记得调用`unlock()`方法重新开启计时，如果页面提供了手动锁定的按钮，当用户点击锁定按钮成功锁屏后，记得调用`lock()`方法停止计时器。

## 兼容性

主要使用了 [addEventListener](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener)、[removeEventListener](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/removeEventListener) 和 [dispatchEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/dispatchEvent) 这三个接口，兼容IE9+以及其他浏览器，理论上不存在兼容性问题。