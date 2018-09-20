---
layout: docs
title:  "虚拟机安装 Gentoo"
date:   2018-02-04
categories: Linux Gentoo
enable_mathjax: false
enable_mermaid: false
enable_comments: true
---
## 说在前面
+ 配置相关 主要按照<https://wiki.gentoo.org>上面的指示进行操作。
+ 有一些操作比较谨慎，为了速度可以跳过某些可能无用的步骤。但是个人认为，欲速则不达。

<hr>
## 准备安装 Gentoo

### 主机环境
+ Linux Kernel 4.18 (Fedora 28)
+ VirtualBox 5.2.18

### 虚拟机基本配置
+ CPU 核心数: 4
+ EFI:  启用
+ 内存： 6G
+ (SATA 0) 硬盘: 12 GiB (普通 完全分配 SSD)
    - 存放 引导程序 及 内核
    - 存放 系统本身
+ (SATA 1) 硬盘: 16 GiB (普通 动态分配 HDD)
    - 存放 portage 缓存
    - 存放 内核源码
    - 存放 home
    - 存放 var
+ (SATA 2) 硬盘: 18 GiB (完全写入 动态分配 HDD)
    - 存放 swap
    - 存放 编译中间文件
    - 一定要给多一点 否则编译到一半因为空间不足而失败，是一件令人郁闷的事情。
+ (SATA 4) 光盘: Fedora 28 Live CD (仅仅用作启动盘)
+ 虚拟网卡：2个
    - Host-Only
    - NAT

### 关机快照：0
（略）
### 启动虚拟机

使用 Fedora 的 LiveCD 启动到界面，使得虚拟机可以连接到外部的网络。

<hr>
## 安装 Gentoo 之前的准备

### 使用 gdisk 为硬盘进行分区

由于是在新创建的虚拟机中，所以直接使用下述命令即可进行分区。

``` bash
(Fedora Live) $ gdisk /dev/sda
(Fedora Live) $ gdisk /dev/sdb
(Fedora Live) $ gdisk /dev/sdc
```

否则，建议使用 lsblk 等命令进行确认。以下是本人分区完成后使用 gdisk -l 的结果:

```
(Fedora Live) $ gdisk /dev/sda -l
Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present
...

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          133119   64.0 MiB    EF00  EFI System
   2          133120         2099199   960.0 MiB   8300  Linux filesystem
   3         2099200        25165790   11.0 GiB    8300  Linux filesystem

(Fedora Live) $ gdisk /dev/sdb -l
...

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048        33554398   16.0 GiB    8300  Linux filesystem

(Fedora Live) $ gdisk /dev/sdc -l
...

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         4196351   2.0 GiB     8200  Linux swap
   2         4196352        37748702   16.0 GiB    8300  Linux filesystem
...

```

### 格式化各个分区并挂载

``` bash
###### format devices
(Fedora Live) $ sudo mkfs.fat -n "EFIBOOT" /dev/sda1
(Fedora Live) $ sudo mkfs.xfs -L "BOOT"    /dev/sda2
(Fedora Live) $ sudo mkfs.xfs -L "ROOT"    /dev/sda3
(Fedora Live) $ sudo mkfs.xfs -L "A_DATA"  /dev/sdb1
(Fedora Live) $ sudo mkswap                /dev/sdc1
(Fedora Live) $ sudo mkfs.xfs -L "A_TEMP"  /dev/sdc2

###### mount devices
(Fedora Live) $ sudo mkdir /mnt/gentoo
(Fedora Live) $ sudo mount /dev/sda3 /mnt/gentoo
(Fedora Live) $ sudo mkdir -p /mnt/gentoo/{A/{DATA,TEMP},boot,home,var}
(Fedora Live) $ sudo mount /dev/sda2 /mnt/gentoo/boot
(Fedora Live) $ sudo mount /dev/sdb1 /mnt/gentoo/A/DATA
(Fedora Live) $ sudo mount /dev/sdc2 /mnt/gentoo/A/TEMP

(Fedora Live) $ sudo mkdir -p /mnt/gentoo/boot/efi
(Fedora Live) $ sudo mount /dev/sda1 /mnt/gentoo/boot/efi

###### bind directories
(Fedora Live) $ sudo mkdir -p /mnt/gentoo/A/DATA/__ROOT_{var,home}
(Fedora Live) $ sudo mount --bind /mnt/gentoo/A/DATA/__ROOT_home /mnt/gentoo/home
(Fedora Live) $ sudo mount --bind /mnt/gentoo/A/DATA/__ROOT_var  /mnt/gentoo/var

(Fedora Live) $ sudo mkdir -p /mnt/gentoo/A/DATA/__ROOT_usr_{src,portage_{distfiles,packages}}
(Fedora Live) $ sudo mkdir -p /mnt/gentoo/usr/{src,portage/{distfiles,packages}}
(Fedora Live) $ sudo mount --bind /mnt/gentoo/A/DATA/__ROOT_usr_src /mnt/gentoo/usr/src
(Fedora Live) $ sudo mount --bind /mnt/gentoo/A/DATA/__ROOT_usr_portage_distfiles /mnt/gentoo/usr/portage/distfiles
(Fedora Live) $ sudo mount --bind /mnt/gentoo/A/DATA/__ROOT_usr_portage_packages /mnt/gentoo/usr/portage/packages

(Fedora Live) $ sudo mkdir -p /mnt/gentoo/A/TEMP/__ROOT_var_tmp
(Fedora Live) $ sudo mkdir -p /mnt/gentoo/var/tmp
(Fedora Live) $ sudo mount --bind /mnt/gentoo/A/TEMP/__ROOT_var_tmp /mnt/gentoo/var/tmp

```

使用 lsblk 和 mount 命令查看挂载情况 ( 截取部分有用的内容 )

```
(Fedora Live) $ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0   12G  0 disk
├─sda1        8:1    0   64M  0 part /mnt/gentoo/boot/efi
├─sda2        8:2    0  960M  0 part /mnt/gentoo/boot
└─sda3        8:3    0   11G  0 part /mnt/gentoo
sdb           8:16   0   16G  0 disk
└─sdb1        8:17   0   16G  0 part /mnt/gentoo/A/DATA
sdc           8:32   0   18G  0 disk
├─sdc1        8:33   0    2G  0 part
└─sdc2        8:34   0   16G  0 part /mnt/gentoo/A/TEMP

(Fedora Live) $ mount | gentoo
/dev/sda3 on /mnt/gentoo type xfs ...
/dev/sda2 on /mnt/gentoo/boot type xfs ...
/dev/sda1 on /mnt/gentoo/boot/efi type vfat ...
/dev/sdb1 on /mnt/gentoo/A/DATA type xfs ...
/dev/sdc2 on /mnt/gentoo/A/TEMP type xfs ...
/dev/sdb1 on /mnt/gentoo/home type xfs ...
/dev/sdb1 on /mnt/gentoo/var type xfs ...
/dev/sdb1 on /mnt/gentoo/usr/portage/distfiles type xfs ...
/dev/sdb1 on /mnt/gentoo/usr/portage/packages type xfs ...
/dev/sdc2 on /mnt/gentoo/var/tmp type xfs ...
/dev/sdc2 on /mnt/gentoo/A/DATA/__ROOT_var/tmp type xfs ...

```

### 下载并解压 stage3
使用 Fedora Live CD 中自带的 Firefox 即可完成下载。下载完成后将 stage4 拷贝到 /tmp。解压缩时按照官网提供的解压缩命令进行解压缩。

``` bash
(Fedora Live) $ cd /mnt/gentoo
(Fedora Live) $ sudo tar -xvjpf /tmp/stage3-*.tar.bz2 --xattrs --numeric-owner
(Fedora Live) $ sudo mkdir /mnt/gentoo/etc/portage/repos.conf
(Fedora Live) $ sudo cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```

### 配置 /etc/fstab
``` bash
(Fedora Live) $ sudo su
(Fedora Live ROOT) cat > /mnt/gentoo/etc/fstab << EOF
/dev/sda1   /boot/efi       vfat      nouser,nosuid,noexec,sync,umask=0077  0 2
/dev/sda2   /boot           xfs       defaults,noatime,nodiratime           0 0
/dev/sda3   /               xfs       defaults,noatime,nodiratime           0 0
/dev/sdb1   /A/DATA         xfs       defaults,noatime,nodiratime           0 0
/dev/sdc2   /A/TEMP         xfs       defaults,noatime,nodiratime           0 0

/dev/sdc1   swap            swap      defaults                              0 0

/A/DATA/__ROOT_home                   /home                   none  bind    0 0
/A/DATA/__ROOT_var                    /var                    none  bind    0 0
/A/DATA/__ROOT_usr_src                /usr/src                none  bind    0 0
/A/DATA/__ROOT_usr_portage_distfiles  /usr/portage/distfiles  none  bind    0 0
/A/DATA/__ROOT_usr_portage_packages   /usr/portage/packages   none  bind    0 0

/A/TEMP/__ROOT_var_tmp                /var/tmp                none  bind    0 0

EOF

(Fedora Live ROOT) exit
```

### 关机快照：1
（略）

### 重新挂在分区，并进入 Gentoo
``` bash
(Fedora Live) $ sudo mkdir /mnt/gentoo
(Fedora Live) $ sudo mount /dev/sda3 /mnt/gentoo
(Fedora Live) $ sudo mount -t proc /proc /mnt/gentoo/proc
(Fedora Live) $ sudo mount --rbind /sys /mnt/gentoo/sys
(Fedora Live) $ sudo mount --make-rslave /mnt/gentoo/sys
(Fedora Live) $ sudo mount --rbind /dev /mnt/gentoo/dev
(Fedora Live) $ sudo mount --make-rslave /mnt/gentoo/dev
(Fedora Live) $ sudo cp -L /etc/resolv.conf /mnt/gentoo/etc/

(Fedora Live) $ sudo chroot /mnt/gentoo /bin/bash
(chroot gentoo) . /etc/profile
(chroot gentoo) mount -a
(chroot gentoo) mount -a       # /boot/efi会挂载失败，需要再挂一次

```

<hr>
## Chroot 环境下对 Gentoo 进行必要配置

### 配置 /etc/portage/make.conf
``` bash
(chroot gentoo) cat > /etc/portage/make.conf << EOF
GENTOO_MIRRORS="http://mirrors.163.com/gentoo/"
PORTDIR="/usr/portage"
PKGDIR="/usr/portage/packages"
DISTDIR="/usr/portage/distfiles"

CFLAGS="-O2 -pipe"
LC_MESSAGES=C

CPU_FLAGS_X86="sse4_1 sse4_2 sse3 ssse3 sse2 sse mmx mmxext"
PYTHON_TARGETS="python3_6 python3_5 python3_4 python2_7"
USE="bzip2 ipv6 lzma threads urandom"

EOF
```

### 安装 ebuild 数据库快照
``` bash
(chroot gentoo) emerge-webrsync
```

### 选择一个 profile (default/linux/amd64/17.0/systemd)
``` bash
(chroot gentoo) eselect profile list
(chroot gentoo) eselect profile set <no.>
```

### 配置地区
``` bash
(chroot gentoo) echo -e "en_US.UTF-8 UTF-8\nzh_CN.UTF-8 UTF-8" > /etc/locale.gen
(chroot gentoo) locale-gen
(chroot gentoo) eselect locale list # 找到 en_US.UTF8，将序号补在下面的命令中
(chroot gentoo) eselect locale set <no.>
(chroot gentoo) env-update
```

### 配置时区
``` bash
(chroot gentoo) echo "Asia/Shanghai" > /etc/timezone
(chroot gentoo) emerge --config sys-libs/timezone-data
```

### 配置 /var/lib/portage/world
``` bash
(chroot gentoo) cat > /var/lib/portage/world << EOF
app-admin/sudo
app-editors/vim
app-portage/gentoolkit
net-misc/networkmanager
sys-apps/gptfdisk
sys-apps/systemd
sys-boot/grub
sys-devel/gcc
sys-fs/xfsprogs
sys-kernel/dracut
sys-kernel/gentoo-sources:4.14.65

EOF
```

### 配置 /etc/portage/package.use
``` bash
(chroot gentoo) cd /etc/portage/package.use
(chroot gentoo) rm *

(chroot gentoo) cat > 00_sys-boot_grub << EOF
sys-boot/grub grub_platforms_efi-64 grub_platforms_pc device-mapper
EOF

(chroot gentoo) cat > 01_sys-apps_systemd << EOF
sys-apps/systemd elfutils sysv-utils
EOF

(chroot gentoo) cat > 20_net-misc_networkmanager << EOF
net-misc/networkmanager dhclient -dhcpcd -modemmanager -ppp -wext -wifi
EOF

```
### 配置 ROOT 密码
``` bash
(chroot gentoo) passwd
```

### 准备重构整个系统
``` bash
(chroot gentoo) emerge -aef @world
(chroot gentoo) emerge -a sys-kernel/gentoo-sources:4.14.65
```

### 配置内核
``` bash
(chroot gentoo) cd /usr/src/linux
(chroot gentoo) make defconfig                # 先进行一次默认值配置
(chroot gentoo) make menuconfig               # 根据 WIKI 进行自定义配置
```

### 关机快照：2
（略）

### 重构整个系统并删除无用的软件包
``` bash
(chroot gentoo) emerge -auDN @world --keep-going
(chroot gentoo) emerge -a --depclean
```

#### 安装内核
``` bash
(chroot gentoo) cd /usr/src/linux
(chroot gentoo) make -j8                      # 编译
(chroot gentoo) make modules_install          # 安装
(chroot gentoo) make install                  # 安装
(chroot gentoo) dracut --kver ${KVER}         # 生成initramfs
```

#### 配置 GRUB2
``` bash
(chroot gentoo) grub-install --efi-directory=/boot/efi
(chroot gentoo) cd /boot/efi/EFI
(chroot gentoo) cp gentoo/grubx64.efi BOOT/BOOTX64.EFI
(chroot gentoo) grub-mkconfig -o /boot/grub/grub.cfg
```

### 关机快照：3
（略）

<hr>
## 调整 Gentoo 系统

### 第一次配置  再重启之后就可以通过 ssh 进行配置
``` bash
###### 默认启动到命令行
(gentoo root) systemctl set-default multi-user.target
###### 启动两个必要的网络服务
(gentoo root) systemctl enable sshd
(gentoo root) systemctl enable NetworkManager
###### 配置用户和权限 使组 wheel 组 拥有 sudo 权限
(gentoo root) visudo
(gentoo root) useradd admin
(gentoo root) usermod -a -G wheel admin
(gentoo root) passwd admin
###### 配置主机名称
(gentoo root) hostnamectl set-hostname '....'
```

### 关机快照：4
（略）

### 重新设置网络
``` bash
### 配置 host-only 网卡
(gentoo root) nmcli con modify <......> \
  connection.id enp0s3 \
  ipv4.method manual \
  ipv4.addresses <ip> \
```

### 重新选择一个 profile (default/linux/amd64/17.0/desktop/plasma/systemd)
``` bash
(gentoo root) eselect profile list
(gentoo root) eselect profile set <no.>
```

### 重新配置 /var/lib/portage/world 并做好更新整个系统的准备
``` bash
(gentoo root) cat > /var/lib/portage/world << EOF
app-admin/sudo
app-editors/vim
app-portage/gentoolkit
app-shells/zsh
app-text/tree
dev-db/mariadb
dev-lang/python:2.7
dev-lang/python:3.6
dev-lang/ruby
dev-vcs/git
kde-plasma/plasma-desktop
media-fonts/wqy-bitmapfont
media-fonts/wqy-microhei
net-misc/networkmanager
sys-apps/gptfdisk
sys-apps/systemd
sys-boot/grub
sys-devel/gcc
sys-fs/xfsprogs
sys-kernel/dracut
sys-kernel/gentoo-sources:4.14.65

EOF

(gentoo root) emerge -aef @world
```

### 关机快照：5
（略）

### 更新整个系统

``` bash
(gentoo) $ sudo emerge -e @world --keep-going
(gentoo) $ sudo emerge -a --depclean
(gentoo) $ sudo revdep-rebuild
```

### 清理多余的快照，并打下当前状态的快照(略)
