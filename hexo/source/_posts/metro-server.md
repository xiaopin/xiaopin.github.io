---
title: Unable to load script. Make sure you're either running a Metro (run 'npx react-native start') or that your bundle "index.android.bundle' is packaged correctly for release.
abbrlink: e085a09c
date: 2022-07-15 09:57:42
tags:
    - React Native
---

当通过 `yarn run android` 将 React Native 项目跑在 Android 模拟器上时，模拟器上报了如下错误信息：

```
Unable to load script. Make sure you're either running a Metro (run 'npx react-native start') or that your bundle "index.android.bundle' is packaged correctly for release.
```

如果是没运行 metro 服务，那通过 `yarn start` 开启服务即可，可我明明已经将 metro 服务跑起来了，Android 模拟器还是报这个错。

最后排查原因是我之前在 `8081` 端口上运行了一个 vue 项目，把端口占用了；把 vue 项目停掉，释放端口，重新运行 `yarn start` 和 `yarn run android` 就解决了。

> 但是这个端口问题不影响 iOS 模拟器，iOS 模拟器能够正常跑起来，只有 Android 模拟器才会报错，不清楚 metro 在两者间有何区别。