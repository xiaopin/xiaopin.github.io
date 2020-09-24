---
title: macOS/Linux 文本替换
tags:
  - Other
abbrlink: e884faf9
date: 2019-03-25 14:42:15
---

有时候我们需要将项目下的某个字符串全局替换成另一个字符串，可以借助 IDE 工具，当然也可以直接通过 `grep` + `sed` 命令完成。

## 单文件文本内容替换

```bash
sed -i "" "s/search/replace/g" filename
```

参数解析：
- search: 需要被替换的文本
- replace: 替换后的文本 
- filename: 需要进行文本替换的文件

示例：将当前目录下的 `a.txt` 文件中的 `Apple` 替换成 `Pear`

```bash
sed -i "" "s/Apple/Pear/g" ./a.txt
```

## 多文件文本内容替换

```bash
grep -rl "search" directory | xargs sed -i "" "s/search/replace/g"
```

参数解析:
- search: 需要被替换的文本
- directory: 需要进行文本替换操作的目录
- replace: 替换后的文本

示例：将 `Documents` 目录下的所有文件的 `Apple` 替换成 `Pear`

```bash
grep -rl "Apple" ~/Documents | xargs sed -i "" "s/Apple/Pear/g"
```

## 转义字符

示例1：将 `//` 替换成 `\\`
```bash
grep -rl "//" ./ | xargs sed -i "" "s/\/\//\\\\\\\\/g"
```


## Note

你可能已经注意到 `-i` 参数后面加了个空字符串（`""`），这是 macOS 平台下必须添加的，-i 参数后面必须跟一个字符串用来备份，空字符串表示不备份，如果省略该字符串则会报如下错误：
```bash
sed: 1: ".//a.txt": invalid command code .
```
Linux 平台请自行忽略 `""`。


当然，grep、sed 命令的用法远不止这些，更多用法请自行研究，这里给出菜鸟教程中对这两个命令的分别介绍：
- [grep](http://www.runoob.com/linux/linux-comm-grep.html)
- [sed](http://www.runoob.com/linux/linux-comm-sed.html)
