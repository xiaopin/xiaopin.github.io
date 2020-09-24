---
title: 用一个仓库管理hexo博客源码与站点文件
tags:
  - 随记
abbrlink: b0d2b79f
date: 2016-11-23 21:43:41
---

在GitHub上创建一个新仓库，注意命名为`username.github.io`即可。

### 将仓库克隆到本地

``` bash
$ cd ~/Documents
$ git clone https://github.com/xiaopin/xiaopin.github.io.git
```

### 创建hexo分支用于管理博客源码

``` bash
$ git clone https://github.com/xiaopin/xiaopin.github.io.git
$ cd xiaopin.github.io/
$ git checkout -b hexo
$ touch .gitignore
$ git add .gitignore
$ git commit -m “add .gitignore“
$ git push origin hexo
```

### 在hexo分支下创建站点
``` bash
$ mkdir hexo
$ cd hexo/
$ hexo init
$ npm install
```

### 安装主题并通过git子模块来管理主题
``` bash
$ cd ../
$ git submodule add https://github.com/litten/hexo-theme-yilia.git hexo/themes/yilia
```

### 将hexo源码与主题提交到GitHub仓库

``` bash
$ git add .
$ git commit -m "add hexo source and yilia theme"
$ git push origin hexo
```

### 部署

安装hexo-deployer-git
``` bash
npm install hexo-deployer-git --save
```

修改配置_config.yml
```
deploy:
  type: git
  repo: https://github.com/xiaopin/xiaopin.github.io.git
  branch: master
```

配置好后只需执行`hexo g -d`即可生成静态文件并上传到GitHub上。

### 添加README.md

将README.md文件放到`hexo/source/`目录下，此时部署的话，你会发现hexo把README.md解析成了HTML了。这里采取修改后缀名的方式解决，将`README.md`重命名为`README.mdown`。

如果需要自定义域名，在`hexo/source/`目录下，新建一个`CNAME`文件，文件内容为你的域名（如：0daybug.com）。


#### 附上.gitignore的内容

```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

