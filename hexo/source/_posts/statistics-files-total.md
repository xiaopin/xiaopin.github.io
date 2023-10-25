---
title: macOS/Liunx 统计指定目录下的文件数量
abbrlink: a260161
date: 2023-08-17 17:45:29
tags:
    - masOS
---

## 仅限当前目录（不包含子目录以及隐藏文件）

-   统计文档当前目录下的文件总数

```shell
$ ls -l ~/Documents | grep '^-' | wc -l
```

-   统计文档当前目录下的文件夹总数

```shell
$ ls -l ~/Documents | grep '^d' | wc -l
```

-   统计文档当前目录下的文件和文件夹的总数

```shell
$ ls -l ~/Documents | grep '^[-d]' | wc -l
```

## 通过`R`参数递归统计（包含当前目录及所有子目录，但是不包含隐藏文件）

-   统计文档目录及其所有子目录下的文件总数

```shell
$ ls -lR ~/Documents | grep '^-' | wc -l
```

-   统计文档目录及其所有子目录下的文件夹总数

```shell
$ ls -lR ~/Documents | grep '^d' | wc -l
```

-   统计文档目录及其所有子目录下的文件和文件夹的总数

```shell
$ ls -lR ~/Documents | grep -E '^(-|d)' | wc -l
```

## 通过`A`参数列出隐藏文件(已排除`.`和`..`目录，包含当前目录及其所有子目录，同时包括了隐藏文件)

-   统计文档目录及其所有子目录下的文件总数

```shell
$ ls -lAR ~/Documents | grep '^-' | wc -l
```

-   统计文档目录及其所有子目录下的文件夹总数

```shell
$ ls -lAR ~/Documents | grep '^d' | wc -l
```

-   统计文档目录及其所有子目录下的文件和文件夹的总数

```shell
$ ls -lAR ~/Documents | grep -E '^(-|d)' | wc -l
```
