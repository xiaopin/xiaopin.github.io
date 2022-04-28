---
title: iOS新项目企业签名打包报错
abbrlink: c38eb6bd
date: 2022-04-28 14:33:27
tags:
    - iOS
---

工作了这么多年，还是今年第一次接触到苹果的企业账号打包 App，由于 app 不上架，都是通过企业签名方式打包，让客户通过扫码方式下载安装。

## 错误提示

由于是新项目并且是 `Automatically manage signing` 自动签名，当执行 Archive 打包时，给出了如下错误信息：

```
You currently don’t have access to this membership resource. To resolve this issue, agree to the latest Program License Agreement in your developer account.
```

## 解决方案

使用企业账号登陆[苹果开发者后台](https://developer.apple.com)，登陆进去后苹果就会给你弹出一个红色的提示内容，点击 `Review Agreement` 按钮：

![Apple Developer Enterprise Program Update](/images/2022/AppleDeveloperEnterpriseProgramUpdate.png)

弹出新的协议，只需要同意即可。

![Apple Developer Enterprise Program](/images/2022/AppleDeveloperEnterpriseProgram.png)

同意协议后重新打包即可正常签名。