---
title: Homebrew
date: 2019-05-29 09:15:50
tags:
	- macOS
---

Homebrew 是 mac 平台的软件包管理器，它允许我们通过 `brew install xxx` 的方式安装软件。

#### 安装 Homebrew

[官网](https://brew.sh/index_zh-cn) 上有详细的说明，可以自行查阅。

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Homebrew 默认会安装在 /usr/local/Homebrew 目录。

#### 使用 Homebrew

这里只介绍常用的命令，更多用法或详细用法请使用 `brew --help` 查看。

通过 Homebrew 安装的软件包会默认安装在 `/usr/local/Cellar` 目录下，并会将其软连接到 `/usr/local/bin/` 目录。

- 搜索软件
```
brew search xxx
```

- 安装软件
```
brew install xxx
```

- 卸载软件
```
brew uninstall xxx
```

- 更新软件
```
brew upgrade xxx
```

- 查看软件包信息
```
brew info xxx
```

- 列出所有已安装的软件
```
brew list
```

- 更新 Homebrew
```
brew update
```

#### 更换源

当我们执行 `brew install xxx` 或者 `brew update` 命令时，终端会一直卡在 `Updating Homebrew` 的地方，这是因为 Homebrew 自带的镜像源被墙，访问不了，此时需要更换成国内的镜像源。

下面以 [清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/) 为例：

```
cd "$(brew --repo)"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

brew update
```

可以恢复原来的镜像：

```
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git

cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core

brew update
```