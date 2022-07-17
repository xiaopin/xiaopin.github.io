---
title: 关于 react-native-snap-carousel 报 ViewPropTypes 的错误
abbrlink: 537cc963
date: 2022-07-11 22:37:14
tags:
    - React Native
    - TypeScript
    - JavaScript
---

## 环境

先说一下我当前的环境

- react 18.0.0
- react-native 0.69.1
- react-native-snap-carousel 3.9.1

## 错误信息

通过 `yarn add react-native-snap-carousel` 安装后，先执行 `pod install` 安装 iOS 依赖，然后通过 `yarn start` 和 `yarn run ios` 运行项目，最后在控制台中会输出以下错误信息：

```
 ERROR  Invariant Violation: ViewPropTypes has been removed from React Native. Migrate to ViewPropTypes exported from 'deprecated-react-native-prop-types'.
 ERROR  Invariant Violation: Module AppRegistry is not a registered callable module (calling runApplication). A frequent cause of the error is that the application entry file path is incorrect.
      This can also happen when the JS bundle is corrupt or there is an early initialization error when loading React Native.
 ERROR  Invariant Violation: Module AppRegistry is not a registered callable module (calling runApplication). A frequent cause of the error is that the application entry file path is incorrect.
      This can also happen when the JS bundle is corrupt or there is an early initialization error when loading React Native.
```

## 解决方案

试了下网上的一些方案，然并卵，所以这里采用修改源码的方案。

- 安装 `deprecated-react-native-prop-types` 库
```
yarn add deprecated-react-native-prop-types -D
```

- 修改 `react-native-snap-carousel` 源代码

找到 `node_modules/react-native-snap-carousel/src/` 目录下的文件，逐一将 `ViewPropTypes` 的导入从 `react-native` 改为 `deprecated-react-native-prop-types`。

例如 `node_modules/react-native-snap-carousel/src/carousel/Carousel.js`  中，将

```JavaScript
import { Animated, Easing, FlatList, I18nManager, Platform, ScrollView, View, ViewPropTypes } from 'react-native';
```

替换为：

```JavaScript
import { Animated, Easing, FlatList, I18nManager, Platform, ScrollView, View } from 'react-native';
import { ViewPropTypes } from 'deprecated-react-native-prop-types';
```

将所有文件替换完成后，重新编译运行即可。

> 该方案有个不好的地方，每次执行 `yarn add` / `yarn remove` 操作后，都需要重新手动修改一次。即便如此，也总比运行失败要好，只能期待后续更新修复了。
