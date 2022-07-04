---
title: macOS 通过 brew 安装 Tomcat
abbrlink: 378f3f3a
date: 2022-06-28 10:49:11
tags:
    - macOS
---

这里是通过 [Homebrew](https://brew.sh) 来安装 [Tomcat](https://tomcat.apache.org)，不涉及通过其他途径的安装。

## 安装

- 查找可用的 tomcat 版本

```shell
brew search tomcat
```

正常情况下会输出如下内容：

```
tomcat        tomcat-native        tomcat@7        tomcat@8        tomcat@9
```

目前 Tomcat 最新版本是10，这里列出了可用的版本有 7、8、9、10，可根据实际情况安装对应的版本。

- 安装 Tomcat

```shell
brew install tomcat
```

稍等片刻，等待命令执行完毕即可成功安装 Tomcat。

## 启动 Tomcat

```shell
brew services start tomcat
```

如果终端提示 `Successfully started 'tomcat' (label: homebrew.mxcl.tomcat)` 则说明启动成功，浏览器访问 `http://localhost:8080` 即可看到 Tomcat 的页面。

## Tomcat 目录

通过 brew 安装的 Tomcat，相关的目录如下：

- 安装目录 `/usr/local/Cellar/`
- 配置文件 `/usr/local/etc/tomcat/`
- 站点目录 `/usr/local/Cellar/tomcat/10.0.14/libexec/webapps/`

当我们在浏览器访问 `http://localhost:8080` 时，加载的是 `/usr/local/Cellar/tomcat/10.0.14/libexec/webapps/ROOT/index.jsp`，所以 Tomcat 应该默认加载的是 `ROOT` 目录。

## 部署 Vue 项目

如果 Vue 项目未修改 `publicPath` 参数，则将打包出来后的 `dist` 目录下的*所有文件*拷贝到 Tomcat 的 `webapps/ROOT/` 目录下，直接访问 `http://localhost:8080` 即可打开 Vue 项目。

如果想部署到子路径下，比如想通过 `http://localhost:8080/dist/` 目录来访问，则在打包时将 `publicPath` 参数修改为 `/dist/`，然后将打包后得到的 `dist` 目录拷贝到 `webapps/` 目录下即可。
