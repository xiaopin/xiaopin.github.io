---
title: macOS读写NTFS格式的磁盘
abbrlink: 765cd42b
date: 2023-08-16 17:20:09
tags:
    - macOS
---

当我们在 macOS 系统外接 NTFS 格式的磁盘(U 盘/移动硬盘等)时，正常情况下，只能读取，不能写入。我们可以借助一些第三方软件，实现 NTFS 的读写操作，但是其实 macOS 已经内置了对 NTFS 的读写支持，只是由于种种原因，Apple 屏蔽了对 NTFS 的“写入”操作。

接下来我们将通过命令的方式实现对 NTFS 格式的读写操作。

1、执行 `diskutil list` 命令查看磁盘信息

```shell
$ diskutil list
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *500.3 GB   disk0
   1:                        EFI  EFI                    209.7 MB   disk0s1
   2:                 Apple_APFS  Container disk1        500.1 GB   disk0s2

/dev/disk1 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +500.1 GB   disk1
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD - 数据     341.9 GB   disk1s1
   2:                APFS Volume Preboot                 547.7 MB   disk1s2
   3:                APFS Volume Recovery                620.4 MB   disk1s3
   4:                APFS Volume VM                      9.7 GB     disk1s4
   5:                APFS Volume Macintosh HD            15.3 GB    disk1s5
   6:              APFS Snapshot com.apple.os.update-... 15.3 GB    disk1s5s1

/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *30.9 GB    disk2
   1:               Windows_NTFS KINGSTON                29.6 GB    disk2s1
   2:                       0x1B                         469.2 MB   disk2s2
```

> 划重点： `NAME` 列中的 `KINGSTON` 和 `IDENTIFIER` 列中的 `disk2s1` 这两个信息要记住，我们后面要用到

2、卸载磁盘

```shell
$ sudo umount /Volumes/KINGSTON
```

其中 `KINGSTON` 是 U 盘挂载之后的名称，参考上面列出的 `NAME` 列信息

3、创建一个临时目录，用于挂载磁盘

```shell
$ sudo mkdir /Volumes/USB
```

4、将磁盘以 NTFS 格式重新挂载

```shell
$ sudo mount -t ntfs -o rw,auto,nobrowse /dev/disk2s1 /Volumes/USB
```

其中 `/dev/disk2s1` 为需要挂载的磁盘，可以从 `diskutil list` 命令所列出的结果中获取到，`/Volumes/USB` 则为前面刚创建的临时目录。

至此，我们就可以随意地读写 NTFS 格式的磁盘了，只不过当我们将磁盘卸载之后，重新插上，则需要根据上述步骤再操作一遍。

> 想省事又不想安装第三方 NTFS 软件的话，可以考虑将磁盘格式化为 `FAT32` 格式，macOS 默认支持对 FAT32 格式进行读写操作，但是该格式有个缺点就是单文件大小不能超过`4GB`。


