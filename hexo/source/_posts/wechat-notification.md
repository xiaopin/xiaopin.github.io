---
title: 为微信小程序增加通知机制
date: 2018-12-11 12:25:16
tags:
	- 微信小程序
---

当有多个页面需要监听某个状态（如：登录状态），当状态发生改变时，需要及时通知到每个监听者（页面），这就是通知机制。

## 源码

新建一个 `NotificationCenter.js` 文件，内容如下：

```JavaScript
/**
 * @variable 记录所有注册的通知
 * 
 * key为通知的名称，value为监听该通知的对象数组
 */
let _observerTables = {};

const NC_NAME_KEY       = 'NC_NAME_KEY';
const NC_OBSERVER_KEY   = 'NC_OBSERVER_KEY';
const NS_CALLBACK_KEY   = 'NS_CALLBACK_KEY';

/**
 * @class 通知中心管理类
 * 
 * 添加/移除是成对出现的，需要自行移除观察者对象
 * 否则将会导致观察者对象不被释放，造成内存泄漏
 * 
 * 如果页面只需要在当前展示时进行监听，则在`onShow()`中添加，在`onHide()`中移除
 * 如果页面需要在整个生命周期内进行监听，则在`onLoad()`中添加，在`onUnload()`中移除
 */
class NotificationCenter {

    /**
     * @method 添加一个观察者对象
     * 
     * @param observer  {Object}            观察者(Page对象)
     * @param name      {String}            通知的名称
     * @param callback  {String/Function}   回调(接收一个Object类型的参数,如果为字符串则确保observer对象实现了该方法)
     */
    static addObserver(observer, name, callback) {
        if (typeof observer !== 'object' || observer === null) {
            throw new Error(`${observer} is not a Page object.`);
        }
        if (typeof name !== 'string' || !name.length) {
            throw new Error(`${name} is not a string or is empty.`);
        }
        if (typeof callback === 'string') {
            if (typeof Reflect.get(observer, callback) !== 'function') {
                throw new Error(`${observer} not implemented ${callback} method.`);
            }
        } else if (typeof callback !== 'function') {
            throw new Error(`${callback} is not a function.`);
        }

        let observers = Reflect.get(_observerTables, name) || [];
        observers.push({
            NC_NAME_KEY: name,
            NC_OBSERVER_KEY: observer,
            NS_CALLBACK_KEY: callback
        });
        Reflect.set(_observerTables, name, observers);
    }

    /**
     * @method 移除观察者对象
     * 
     * @param observer  {Object}    观察者(Page对象)
     * @param name      {String}    通知的名称, 如果未指定则会移除`observer`注册的所有通知
     */
    static removeObserver(observer, name = null) {

        const removeObserverBlock = (o, k) => {
            let obs = Reflect.get(_observerTables, k).filter(element => {
                return (Reflect.get(element, NC_OBSERVER_KEY) != o);
            });
            if (obs.length) {
                Reflect.set(_observerTables, k, obs);
            } else {
                Reflect.deleteProperty(_observerTables, k);
            }
        };

        if (typeof name === 'string' && name.length) {
            // 移除指定通知名称的观察者
            if (!Reflect.has(_observerTables, name)) {
                return;
            }
            removeObserverBlock(observer, name);
        } else {
            // 未指定通知名称, 移除 observer 监听的所有通知事件
            for (let key in _observerTables) {
                removeObserverBlock(observer, key);
            }
        }
    }

    /**
     * @method 发送一个通知
     * 
     * @param name      {String}    通知的名称
     * @param userInfo  {Object}    需要传递给观察者的数据,默认为`null`
     */
    static postNotificationName(name, userInfo = null) { 
        if (typeof name !== 'string' || !name.length) {
            throw new Error('`name` is illegal parameter.');
        }
        if (!Reflect.has(_observerTables, name)) {
            return;
        }

        const notification = { name: name, userInfo: userInfo };
        const obs = Reflect.get(_observerTables, name);
        obs.forEach((o) => {
            let observer = Reflect.get(o, NC_OBSERVER_KEY);
            let callback = Reflect.get(o, NS_CALLBACK_KEY);
            let func = (typeof callback === 'function') ? callback : Reflect.get(observer, callback);
            Reflect.apply(func, observer, [notification]);
        });
    }

}

module.exports = NotificationCenter;
```

## 使用示例

- 页面A监听某个通知

    1、导入文件
    ```JavaScript
    const NotificationCenter = require('NotificationCenter.js');
    ```

    2、当页面加载完毕时注册通知(onLoad方法中调用)

    - page 对象必须实现 notificationCallback 方法
    ```JavaScript
    NotificationCenter.addObserver(this, 'NOTIFICATION_NAME', 'notificationCallback');
    ```

    - 匿名函数方式回调
    ```JavaScript
    NotificationCenter.addObserver(this, 'NOTIFICATION_NAME', (res) => {
        console.log(res);
    });
    ```

    两种注册方式，选择你喜欢的一种即可。

    3、当页面卸载时移除通知(onUnload方法中调用)
    ```JavaScript
    NotificationCenter.removeObserver(this, 'NOTIFICATION_NAME');
    ```

- 页面B中发出通知

    4、发出通知
    ```JavaScript
    NotificationCenter.postNotificationName('NOTIFICATION_NAME', {'userInfo' : null});
    ```

注：

1、2和3是成对出现的，添加了注册就要记得移除

2、可以根据实际需求在 onLoad/onUnload、onShow/onHide 中进行添加/移除操作