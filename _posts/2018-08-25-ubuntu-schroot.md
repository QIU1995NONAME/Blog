---
layout: docs
title:  "Linux Debian 安装 Ubuntu schroot 环境"
date:   2018-08-25
categories: Linux Debian schroot
enable_mathjax: false
enable_mermaid: false
enable_comments: true
---

### 工具 & 原料 (安装过程略)
+ Debian 9 x64
+ debootstrap
+ schroot

### 开始安装

首先，创建一个文件夹，放置新安装的 Ubuntu 14.04

```bash
$ export CHROOTS=...                            # 设置一个环境变量，放置所有的chroot环境
$ sudo mkdir -p ${CHROOTS}/ubuntu14             # 创建一个新文件夹
$ sudo debootstrap --arch amd64 trusty ${CHROOTS}/ubuntu14 \
> https://mirrors.tuna.tsinghua.edu.cn/ubuntu/  # 安装 Ubuntu 14 环境
```

然后，配置schroot
```bash
$ sudo cp -a /etc/schroot/default /etc/schroot/ubuntu14
$ cat << EOF | sudo tee /etc/schroot/chroot.d/ubuntu14.conf
[ubuntu14]
description=Ubuntu 14.04
type=directory
directory=${CHROOTS}/ubuntu14
users=qmh
groups=qmh
root-groups=root
setup.config=ubuntu14/config
setup.copyfiles=ubuntu14/copyfiles
setup.fstab=ubuntu14/fstab
setup.nssdatabases=ubuntu14/nssdatabases

EOF
```

进入 schroot 进行下一步配置
```bash
$ sudo apt-get install language-pack-en language-pack-zh-hans  # 安装字符集
$ sudo apt-get install fonts-wqy-microhei                      # 安装中文字体
```

在 /etc/schroot/ubuntu14 目录中即可对 ubuntu14 进行更加详细的配置 (略)


