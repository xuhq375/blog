---
title: "文件备份"
date: 2020-03-15
lastmod: 2020-03-15
author: core-man
draft: false
tags: ["文件备份"]
categories: ["计算机"]
slug: backup
---

在写程序或者处理大量数据时，不进行必要的备份是十分危险的。一旦电脑硬盘出问题，几个月的辛苦就付诸东流。要备份程序和数据，首先就需要有一块备份硬盘，然而不同操作系统（Linux、Windowns和Mac）使用的文件系统 (filesystem) 不尽相同，导致不同文件系统的硬盘往往不能跨平台读取和存储。

- Linux的`ext4`文件系统在Windows和Mac下不可读。
- Windows的`NTFS`文件系统在Mac下只读不能写，`exFAT`文件系统虽在Mac下可读可写，但文件权限会出错；`NTFS`和`exFAT`在Linux下虽可读可写，但是文件权限会出错。
- Mac的`HFS+`文件系统在Linux和Windows下不可读。

首先介绍各操作系统的主要文件系统，然后介绍如何在Linux，Windows和Mac下进行硬盘备份，最后简要介绍网盘备份。


## 文件系统

### Linux文件系统

Linux有几十个文件系统类型，可以利用以下命令查看全部文件系统的介绍：
``` bash
$ man 5 fs
# 或
$ man filesystems
```

Linux主要文件系统:

- `ext2`: Linux second extended file system (ext2fs)，Linux最传统的磁盘文件系统。
- `ext3`: 增加日志功能，可回溯追踪。
- `ext4`: `RHEL6`、`CentOS 6.x`、`Ubuntu`等大部分Linux发行版本的默认文件系统。
- `xfs`: `RHEL7`、`CentOS 7.x`等Linux发行版本的默认文件系统。


### Windows文件系统

Windows主要文件系统:

- `FAT`：也称`FAT16`，MS-DOS和最早期的WIN95操作系统中最常见的硬盘分区格式，能支持最大为2GB的硬盘分区，目前基本上已经不再使用了。
- `FAT32`：Win98开始，`FAT32`得到了广泛的应用，可以支持大到2TB的分区，但是单个文件最大只能支持4GB。
- `NTFS`: 从Windows XP系统开始逐渐成为主流的磁盘格式，目前仍是主流的磁盘格式。针对传统机械硬盘而设计的，对于新兴的Flash闪存材料不一定适用。1993年微软推出了全新的操作系统Windows New Technology (Windows NT)，后来的Windows 2000、XP、Windows 7、Windows 8、Windows 10都属于WindowsNT家族，这个家族的文件系统就叫New Technology Filesystem （`NTFS`）。
- `exFAT`: 也被称为FAT64，近年出现的格式，主要针对移动存储设备，如闪存、U盘等。因为`FAT32`单个文件不能超过4GB，使用`NTFS`又容易损坏闪存芯片，所以开发`exFAT`格式来解决这些问题。

Windows系统的文件格式具有一定的跨平台性

- `FAT32`，`NTFS`和`exFAT`在Linux下可读、可写，但文件权限会出错。
- `FAT32`和`exFAT`在Mac上的可读、可写，但文件权限会出错，而`NTFS`文件系统在Mac下只读不能写。


### Mac文件系统

Mac主要文件系统:

- `UFS`: 全称Unix File System，也称Berkeley Fast File System。Mac OS X Lion版本后才取消对`UFS`的支持。
- `HFS+`: 也称Mac OS Extended或HFS plus。苹果收购NeXT后，`UFS`在MacOS中作为默认文件系统的时间并不长。1998年1月，苹果收购NeXT一年后，发布了Mac OS 8.1，并搭载HFS Plus (`HFS+`)文件系统，用于取代`UFS`。可以用Time Machine进行有效的备份。
- `APFS`: 2017年，伴随着Mac OS High Sierra，苹果正式发布了Apple File System。为了和`AFS (Apple File Service)`区分，采用了`APFS`作为缩写，目前与Time Machine还不兼容。

为了与Windows相兼容，Mac还有`MS-DOS (FAT)`、`Windows NT Filesystem`和`ExFAT`等文件系统。


## 硬盘备份

我买了几块4T的Seagate Backup Plus移动硬盘，大约800元/块，默认文件格式是`NTFS`或`exFAT`，可以根据自己的实际需求对硬盘进行格式化。

我个人的硬盘主要用于：
- 存储数据
    - 为了能在Windows，Mac和Linux跨平台使用，可以选择`exFAT`。其在三个系统里均可读可写，只是在Linux和Mac中文件权限会出错，但是数据文件的权限出错对日常科研并没有影响。
    - 如果只想在Windows和Linux之间使用，也可以采用`NTFS`，其在这两个系统里也是可读可写，但文件权限会出错。
- 增量备份脚本、文件
    - 对于不同的操作系统，需要选择不同的文件系统进行格式化，保证文件的权限不变。
    - 根据自身需求，每隔一段时间(每周或者每个月)进行备份，在出差前最好也对电脑做个备份。

以下的讨论都是基于第二个目的，第一个目的只需格式化操作即可。文件管理可以参考[SeisMan's blog](https://blog.seisman.info/file-organization)。


### Linux

目前我使用CentOS 7，文件系统为`xfs`。为了与大部分Linux版本兼容，我采用更普遍的`ext4`作为移动硬盘的文件系统。

在Linux上插入移动硬盘，查看移动硬盘挂在的目录：
``` bash
$ df -T
Filesystem                         Type      1K-blocks       Used  Available Use% Mounted on
/dev/mapper/cl_ivantjuawinata-root xfs       104802308    8157576   96644732   8% /
devtmpfs                           devtmpfs    8136724          0    8136724   0% /dev
tmpfs                              tmpfs       8160268          0    8160268   0% /dev/shm
tmpfs                              tmpfs       8160268      26636    8133632   1% /run
tmpfs                              tmpfs       8160268          0    8160268   0% /sys/fs/cgroup
tmpfs                              tmpfs       8160268        416    8159852   1% /tmp
/dev/sda2                          xfs          505580     237656     267924  48% /boot
/dev/sda1                          vfat         204580       9960     194620   5% /boot/efi
/dev/mapper/cl_ivantjuawinata-home xfs      1102878568  879866152  223012416  80% /home
tmpfs                              tmpfs       1632056         28    1632028   1% /run/user/1000
/dev/sdc2                          ext4     3845428216 1781612652 1868454900  49% /run/media/core-man/4T
```

可以看到移动硬盘挂载在`/dev/sdc2`，千万不要搞错挂载目录。

``` bash
# 卸载`/dev/sdc2`
$ sudo umount /dev/sdc2

# 将移动硬盘格式化成`ext4`文件系统
$ sudo mkfs.ext4 /dev/sdc2
```

最后利用`rsync`命令对重要的文件、脚本、数据等进行备份了。第一次备份会比较慢，但是第二次由于只更新电脑中改动过的文件，会比较快。可以写一个备份脚本，这里有一个perl脚本[backup.pl](backup.pl)可以参考。


### Windows

Windows下进行备份，移动硬盘的文件系统格式可以是`NTFS`或`exFAT`。备份跟Linux很类似，使用DOS的`robocopy`命令进行同步。这里有一个[backup.txt](backup.txt)文件可以参考，将后缀txt给成bat，就变成了bat脚本，双击bat脚本就直接运行了。需要查看txt原文件，将参数按照自己的实际情况进行修改。

如果备份脚本里有中文，则文件需要另存为成ANSI编码格式，而不是UTF-8格式：

![save as](ANSI-1.png)

![ANSI](ANSI-2.png)


### Mac

我使用Mac强大的Time Machine进行备份，`Time Machine`可以回到之前任意时间点备份的文件状态。

首先将移动硬盘格格式化成`HFS+`。如果有人为了能够与Windows和Linux兼容，可以只将部分空间格式化成`HFS+`，保留部分`NTFS`或者`exFAT`空间，不过有几点注意事项：

- Mac自身的`Disk Utility`不能将硬盘格式化为`NTFS`，只能格式化成`MS-DOS(FAT)`和`exFAT`。如果移动硬盘原始格式不是`NTFS`，而我们想使用`NTFS`，可以用Windows或者Linux下的工具先将其格式化成`NTFS`。
- `NTFS`文件在Mac上只读，`MS-DOS(FAT)`和`exFAT`在Mac上可读、可写，但是文件权限会出错；`MS-DOS(FAT)`单个文件不能大于4GB；`exFAT`作为`FAT32`的升级版，**据说**目前还不太稳定，如果文件系统不够稳定，就存在着分区表丢失、数据丢失等隐患，建议谨慎使用，但是目前`exFAT`又确实是跨三个平台的最好的文件格式。


插上移动硬盘，然后打开Mac的`Disk Utility`:

![Disk Utility 1](MAC-partition-1.png)

将3.8T的空间格式化 (`Partition`) 成`Mac OS Extended (Journaled)`格式：

![Disk Utility 2](MAC-partition-2.png)

![Disk Utility 3](MAC-partition-3.png)

修改分区Name，选择一个自己喜欢名字。同时也可以增加其他格式的分区。


接下来就可以利用`Time Machine`进行增量备份了，第一次会比较久，后面每次更新新文件：
- 系统偏好设置 -> Time Machine -> 选择备份磁盘 -> 选择分好的磁盘专区，将当期Mac电脑文件备份到移动硬盘中


## 网盘备份

现在有很多网盘服务，可以用来在不同设备之间进行同步，也可以用来备份。目前可以在Linux下使用客户端的同步网盘有`坚果云`，`Dropbox`, `MEGA`和`百度网盘`等。其他还有`OneDrive`，`Google Drive`，`七牛云存储`等网盘。

- `坚果云`：国内目前最好的全平台同步网盘，不限空间，但限制每月上传流量1G，下载流量3G；有需要的可以考虑购买高级版。
- `Dropbox`： 国外最好用的全平台同步网盘，但被墙了，熟练掌握科学上网技巧的人可以使用， 一般人还是不要用了；免费用户只有 2GB 容量，最大可扩容到 18GB 以上；付费用户容量为 1T，对于国内用户稍稍有些贵。
- `MEGA`：免费容量50G，作为同步盘来说基本是够用了。
- `百度网盘`：可以有2T的免费空间。
- `OneDrive`: 微软的云服务，新版的 Word、Excel、PowerPoint 以及 Onenote 等等都可以直接保存到 OneDrive 中；初始免费容量 5G；NTU 员工有1T的空间，用来备份和同步日常文件足够了。
- `Google Drive`： 免费容量15G。
- `Google Photos`: 用于同步手机照片。

我个人的使用情况：

- 使用`坚果云`在Mac和Linux之间同步文件
- 使用`OneDrive`和`Google Drive`对Mac进行网盘备份
- 使用`Dropbox`，`OneDrive`和`Google Drive`进行文件共享
- 使用`Google Photos`同步手机照片
- 一些视频、照片、软件、书籍等文件放在`百度网盘`里进行备份


## 驱动程序

如果只用一块移动硬盘，但是想在Windows和Mac系统下使用，也可以直接花钱解决: [https://www.paragon-software.com](https://www.paragon-software.com)。这也是Mac的官方推荐。


## 参考

- Linux文件系统：[鸟哥的Linux私房菜：基础学习篇 第四版](https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content/59.html)、[深入理解ext4等Linux文件系统](https://zhuanlan.zhihu.com/p/44267768)、[Linux文件系统类型简介及支持的文件系统汇总--Linux入门到精通系列](https://blog.csdn.net/lf8289/article/details/2146917)、[daduryi博客](https://blog.csdn.net/daduryi/article/details/81299961)
- Windows文件系统：[Hansel博客](https://blog.csdn.net/hansel/article/details/46482383)、[关于硬盘/U盘/储存卡格式](https://zhuanlan.zhihu.com/p/33506458)
- Mac文件系统：[磁盘工具使用手册](https://support.apple.com/zh-cn/guide/disk-utility/dsku19ed921c/mac)、[谈谈Mac OS的文件系统](https://zhuanlan.zhihu.com/p/33656976)、[How-To Geek](https://www.howtogeek.com/331042/whats-the-difference-between-apfs-macos-extended-hfs-and-exfat)
- Mac磁盘管理：[星际穿越者博客](https://blog.csdn.net/xicikkk/article/details/46900267)、[磁盘工具使用手册](https://support.apple.com/zh-cn/guide/disk-utility/welcome/mac)（[在Mac上使用“磁盘工具”将物理磁盘分区](https://support.apple.com/zh-cn/guide/disk-utility/dskutl14027/mac)、[如何抹掉Mac磁盘](https://support.apple.com/zh-cn/HT208496)、[在Mac上使用“磁盘工具”来格式化磁盘以用于Windows](https://support.apple.com/zh-cn/guide/disk-utility/dskutl1010/mac)）
- 网盘基础：[SeisMan's blog:一些产品的推广链接](https://blog.seisman.info/my-referral-links)、[SeisMan's blog:CentOS 7 配置指南 — 日常软件篇](https://blog.seisman.info/centos7-setup-4)、[SeisMan's blog:我所使用的软件/服务列表](https://blog.seisman.info/personal-preferences)


## 编辑历史

- 2020-09-17: 增加Linux 百度网盘
- 2020-03-02: 增加exFAT格式
- 2019-07-06: 初稿

