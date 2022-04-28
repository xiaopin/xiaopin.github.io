---
title: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
abbrlink: f5176a02
date: 2022-04-28 10:14:55
tags:
    - Other
---

之前在 GitHub 上进行 clone、push 等操作，都是通过 SSH keys 私钥的方式来进行操作，这过程中不需要用到 GitHub 的账号和密码，今天换了台电脑，在执行 push 操作时并在输入账号和密码后却报错了：

```
remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
remote: Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ for more information.
fatal: Authentication failed for 'https://github.com/xiaopin/xiaopin.github.io.git/'
```

原来是 GitHub 已经禁止通过密码的认证方式来操作代码仓库，需要改用个人访问令牌的方式来认证。

那么就去 GitHub 后台创建一个 token 呗，登陆后找到 `Settings` -- `Developer settings` -- `Personal access tokens`，点击 `Generate new token` 按钮新建一个 token

![](/images/2022/1651114231849.png)

按要求填写一个名字，选择有效期并勾选相应的权限，点击保存即可

![](/images/2022/1651113639574.jpg)

最后就会看到新生成的 token 如下图所示：

![](/images/2022/1651114231828.jpg)

将 token 复制下载，当我们执行 push 等操作时，通过将该 token 作为密码填写即可正常操作仓库。
