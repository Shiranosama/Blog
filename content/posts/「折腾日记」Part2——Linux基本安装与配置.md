---
title: 「折腾日记」Linux基本安装与配置——Linux，计算机不得不品的一环Part2
tags:
  - Linux
categories:
  - 折腾日记
  - Linux传教
keywords:
  - Linux
  - Arch
  - Debian
  - RedHat
  - GNU/Linux
description: Linux，计算机不得不品的一环第二部分，介绍Linux基本安装与配置
summary: Linux，计算机不得不品的一环第二部分，介绍Linux基本安装与配置
date: 2025-08-10 11:32:51
draft: false
---

## 前言

### 岚的声明

<font size=4>本文为**岚**的**折腾日记**，它只是因为我**想**而写下的，**不具备**权威性，我也**无法**保证正确，**仅供参考**</font>

在[上一篇文章](https://shiranosama.github.io/posts/aaa88a44/index.html)中，我们介绍了Linux的起源、发展以及如何选择适合自己的发行版。本文将介绍Linux的基本安装与配置，帮助读者更好地了解和使用Linux系统
本文并不会去介绍某一发行版的安装，官方文档已经足够详细，我将会介绍安装通用步骤都做了什么，并引出一些基本概念

与前文相同，本文将**Linux**用于指代**基于Linux内核的操作系统**

### 参考资料
- [英文WikiPedia](https://en.wikipedia.org/)
- [ArchWiki](https://wiki.archlinux.org/)
- [GentooWiki](https://wiki.gentoo.org/)
- [个人经验]()

## 安装

### 准备

- 工具：U盘（虚拟机无需）、Linux发行版映像、[Rufus](https://github.com/pbatard/rufus)/[Etcher](https://etcher.balena.io/)、充足的磁盘空间、以及一双手

1. 刻录LiveCD：使用Rufus或Etcher将Linux发行版映像刻录到U盘上
2. 重启计算机，进入BIOS/UEFI设置，启动LiveCD
3. 使用手机或其他设备打开对应发行版的官方文档，通常会提供详细的安装指南和常见问题解答

> **LiveCD**是一个完整的可引导安装程序（类似WinPE），它包含一个完整的操作系统，允许用户从储存设备将操作系统载入到内存运行，而非从硬盘驱动器载入。**LiveCD**通常用于演示操作系统、系统安装、修复系统故障等场景

### 分区

- 分区前请备份重要数据，分区操作不可逆
- 分区工具：
  + 一般而言，Linux发行版提供的安装映像会自带分区工具，eg.**cfdisk(交互式)**、**fdisk(命令行)**、**parted(交互式)**、**gparted(图形化)** 等；有关工具如何使用，这里不做介绍
  + 替代方案：使用WinPE进行分区再后进入LiveCD进行安装
- 分区方案：
  + 通常，**UEFI设备**建议使用**GPT分区表**，建议分区方案为：
    - **根分区**：推荐格式：**ext4**；**根分区**，建议大小为16GB以上，用于存放`/`目录下的文件
    - **EFI分区**：推荐格式：**fat32**；EFI启动分区，建议大小为300MB，用于存放EFI启动文件，其会挂载到**根分区**的`/boot/efi`目录
    - `/home`：推荐格式：**ext4**；用户主目录，用于存放`/home`目录下的文件，需要将其挂载到**根分区**的`/home`目录
    - **交换分区**：推荐格式：**swap**；交换分区，用于在物理内存不足时提供额外的内存空间，建议大小为物理内存的1.5倍
  + **Legacy BIOS**设备建议使用**MBR分区表**，建议分区方案为：
    - **根分区**：推荐格式：**ext4**；**根分区**，建议大小为16GB以上，用于存放`/`目录下的文件
    - `/home`：推荐格式：**ext4**；用户主目录，用于存放`/home`目录下的文件，需要将其挂载到**根分区**的`/home`目录
    - **交换分区**：推荐格式：**swap**；交换分区，用于在物理内存不足时提供额外的内存空间，建议大小为物理内存的1.5倍
  - **Legacy BIOS**设备分区方案与**UEFI引导**设备分区方案类似，区别在于**分区表格式**和**Boot分区**（MBR分区表为引导程序预留了空间）不同
- Linux分区方案较为灵活，例如`/home`可以合并到**根分区**中，或者增加`/tmp`、`/var`等分区用于存放临时文件和变量文件，独立分区可以增强**安全性**，便于**备份**和**维护**；也可以不使用**交换分区**，在稍后配置**swapfile**或**ZRAM**；**EFI分区**使用**fat32**格式可以与Windows系统共用

- 挂载分区：
  + 使用对应命令将分区挂载到对应目录，例如：
    ```bash
    # 挂载根分区到/mnt
    mount /dev/sda1 /mnt
    # 挂载启动分区到/mnt/boot/efi for UEFI
    mount /dev/sda2 /mnt/boot/efi
    # 挂载用户主目录到/mnt/home
    mount /dev/sda3 /mnt/home
    # 激活交换 如果有
    swapon /dev/sda4
    ```
  + 使用`lsblk`命令查看分区情况，确保分区挂载正确

- 磁盘与分区：
  + **磁盘**通常指存储设备，分区操作是使用**分区表**将**磁盘**划分为多个**逻辑分区**，用户可以为每个**分区**指定**文件系统**和**挂载点**，以便在操作系统中进行访问和管理

> 其他文件系统eg.**btrfs**、**xfs**、**zfs**等也可以根据个人需求进行选择，不同文件系统具有不同的**特性**，例如**btrfs**具有快照、压缩、RAID、子卷等功能；特性不同的文件系统最优分区方案也**不同**，如**btrfs**通常是**根分区配合子卷**

### 网络

- 确保网络连接正常，可以使用`ping bing.com`命令测试网络连接
- 连接网络：
  + 多数安装映像会自动连接有线网络，无线网络需要手动连接；若未提供交互式网络连接工具，可以使用**iw**或**nmcli**等工具进行连接
- 更新时间以确保Live环境时间准确：
  + 使用`timedatectl`命令更新时间

  > 有关网卡驱动问题：如果遇到在LiveCD中无法连接网络，可以尝试使用官方提供的**离线安装映像**，或者使用**有线网络**进行安装

### 安装

- 配置镜像源：
  + 国内用户通常难以连接官方镜像，可以使用国内镜像源进行安装，eg. **[校园网联合镜像站](https://help.mirrors.cernet.edu.cn/)**，按照对应文档进行配置
  + 配置完成后，便可以开始安装；交互式安装工具会自动从配置的镜像源下载并安装软件包
- 更新缓存：
  + 使用对应包管理命令更新软件包缓存
- 安装软件包，除**必须**软件包外，其他可稍后安装：
  + **base**：基础软件包，包括系统核心、基本工具、库等 ***必须***
  + **base-devel**：开发软件包，包括编译器、构建工具、库等 ***必须***
  + **linux**：内核软件包，包括内核、模块等 ***必须***
  + **linux-firmware**：内核固件软件包，包括内核固件等 ***必须***
  + **grub**：引导程序软件包，包括引导程序等 ***必须***
  + **efibootmgr**：EFI引导管理器软件包，用于管理EFI引导项
  + **sudo**：权限管理软件包，用于管理用户权限
  + **networkmanager**：网络管理器软件包，用于管理网络连接
  + **ALSA**：音频系统软件包，用于管理音频设备
  + **pulseaudio**：音频服务器软件包，用于管理音频流
  + **vim**：文本编辑器软件包，用于编辑文本文件
  + 其他软件包，根据需要的软件包进行安装，eg.**gnome**、**plasma**、**firefox**等
- 包管理：
  + **包管理(Package Management)** 是用于统一管理软件包的工具，它旨在自动处理软件包的安装、更新、卸载等操作，并解决软件包之间的依赖关系；通常 **包管理** 会从 **储存库(Repository)** 中下载软件包及其元数据（版本号、校验信息、依赖关系等），然后自动处理软件包的安装、更新、卸载等操作

  > Linux软件包通常具有庞大的**依赖关系**，例如安装**A软件包**需要**B软件包**，安装**B软件包**需要**C软件包**，幸运的是，现在大多数**包管理**会自动解决依赖关系，无需手动安装依赖包

### Chrooting

- 安装了基本软件包后，此时Linux系统已经可以运行，但是由于还没有配置**引导程序**，所以无法从硬盘启动；此时通过LiveCD进入Chroot环境，进一步配置系统
- 记得先复制**DNS信息**：
  ```bash
  cp --dereference /etc/resolv.conf /mnt/etc/
  ```
- 进入Chroot环境的方法请参照官方文档，一些发行版提供快捷设置Chroot环境的脚本，如**arch-chroot**；这里着重介绍**手动设置Chroot环境**的方法：
  1. 挂载必须的文件系统：
    - **/proc/**，伪文件系统，由内核提供，用于提供进程信息、内核参数等
    - **/sys/**，伪文件系统，类似 **/proc/**，但更结构化
    - **/dev/**，设备文件系统，用于提供设备文件
    - **/run/**，临时文件系统，用于运行时数据，如PID文件、锁文件等
    ```bash
    mount -t proc none /mnt/proc
    mount --rbind /sys /mnt/sys
    mount --make-rslave /mnt/sys
    mount --rbind /dev /mnt/dev
    mount --make-rslave /mnt/dev
    mount --bind /run /mnt/run
    mount --make-slave /mnt/run
    ```
    > --make-rslave：将挂载点设置为从属挂载点，用于安装**systemd**时避免挂载点被破坏

  2. 修改根目录：
    ```bash
    chroot /mnt /bin/bash
    ```
  3. 设置环境变量：
    ```bash
    source /etc/profile
    export PS1="(chroot) ${PS1}"
    ```

### Fstab

- **fstab**文件通常列出所有可用的磁盘分区和其他类型的文件系统以及可能不是基于磁盘的数据源，并告诉它们如何初始化或以其他方式集成到更大的文件系统结构中；一般而言，`/etc/fstab`文件中每行包含6个字段，分别是：
  - **\<device\>**：用于指明要挂载的设备或远程文件系统，可以是设备路径、UUID、分区标签
  - **\<dir\>**：用于指明挂载点，即 **\<device\>** 挂载到何处
  - **\<type\>**：用于指明文件系统类型，eg. **ext4**、**vfat**、**swap**等
  - **\<options\>**：用于指明**mount命令**的参数，用半角逗号分隔，如 **defaults**，**noatime**，**nodiratime**，不同文件系统支持的参数不同，请参考对应文件系统文档
  - **\<dump\>**：用于指明是否使用**dump**备份文件系统，0表示不备份，1表示备份
  - **\<fsck\>**：用于指明文件系统检查的顺序，0表示不检查，1表示优先检查，2表示次优先检查
  - 示例：
    ```fstab
    # <device>                                <dir> <type>          <options>                                        <dump> <fsck>
    UUID=0a3407de-014b-458b-b5c1-848e92a327a3 /     ext4 defaults,noatime,nodiratime                       0      1
    UUID=CBB6-24F2                            /boot vfat defaults,nodev,nosuid,noexec,fmask=0177,dmask=0077 0      2
    UUID=f9fe0b69-a280-415d-a03a-a32752370dee none  swap defaults                                           0      0
    UUID=b411dc99-f0a0-4c87-9e05-184977be8539 /home ext4 defaults,noatime,nodiratime                       0      2
    ```
- 交互式安装的发行版会自动配置**fstab**文件；其余发行版请参考官方文档或手动编辑**fstab**文件

### 配置
- 这一步会配置系统的一些基本设置，如时区、主机名、用户名、密码等，许多系统会提供交互式工具进行配置
- 时间：
  ```bash
  # 设置时区
  ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
  # 同步时间
  hwclock --systohc
  ```
  + 这会将`/etc/localtime`链接到`/usr/share/zoneinfo/`下对应时区的文件，并根据此文件中的时区设置系统时间，再同步到硬件时钟
- 本地化：
  ```bash
  # 设置本地化
  vim /etc/locale.gen #把你需要的语言取消注释，eg. en_US.UTF-8 UTF-8、zh_CN.UTF-8 UTF-8
  locale-gen
  echo "LANG=en_US.UTF-8" > /etc/locale.conf
  ```
  + 这会根据`/etc/locale.gen`文件中的设置生成本地化文件，而后设置系统默认本地化为`en_US.UTF-8`

  > 不建议设置`/etc/lacale.conf`为任何**非英文**值，这会导致 **TTY(终端)** 显示乱码

- 键盘布局：
  ```bash
  echo "KEYMAP=us" > /etc/vconsole.conf
  ```
  + 这会设置系统默认键盘布局为`us`，即美式键盘布局；可以根据个人需求进行修改
- 主机名：
  ```bash
  echo "hostname" > /etc/hostname
  ```
  + 这会设置系统主机名为`hostname`；1~63个字符，只能包含`a-z`、`0-9`以及`-`，不能以`-`开头
- Initramfs：
  + 这一步会生成**initramfs**，不同系统生成**initramfs**的方法不同，请参考官方文档

  > **initramfs**是为了避免将大量驱动程序**硬编码**到内核导致内核过大、用于启动时推断根文件系统位置、以及实现休眠挂起等问题而诞生的；它是一个**临时根文件系统**，在系统启动时加载，用于加载内核模块、挂载根文件系统和查找设备等

- User：
  + 这一步会配置用户，包括创建用户、设置密码等；***一定不要忘记设置Root密码和用户密码***，否则无法进入系统
  ```bash
  # 配置Root密码
  passwd root
  # 创建用户并加入wheel组
  useradd -m -G wheel 用户名
  # 设置用户密码
  passwd 用户名
  ```
  + 配置**sudo**，确保你安装了**sudo！！**：
  ```bash
  # 编辑sudoers文件
  vim /etc/sudoers
  # 取消注释以下行，不要使用%wheel ALL=(ALL) NOOPASSWD: ALL！！
  %wheel ALL=(ALL) ALL
  ```
  + 这会配置**sudo**，允许**wheel**组中的用户使用**sudo**命令；***一定不要使用「%wheel ALL=(ALL) NOOPASSWD: ALL」***，这会导致**wheel**组中的用户可以无密码使用**sudo**命令，存在安全风险

  > **sudo(substitute user, do)**不同于**su**，**sudo**允许用户以临时以其他用户的身份执行命令，而不是**完全切换**到其他用户，管理员可以通过配置`/etc/suoders`来限制用户的权限，防止用户执行危险命令

### 引导程序

- **引导程序(bootloader)** 是负责启动操作系统的软件；在计算机启动阶段，操作系统和加载程序并未在**RAM**中运行，此时计算机会执行**引导ROM**中的程序(eg.**BIOS**、**UEFI**)，该程序会初始化硬件，定位并加载**引导程序**，称为**第一阶段引导**；**引导程序**(eg.**GRUB**、**rEFInd**)加载操作系统并使操作系统初始化，称为**第二阶段引导**
- **引导程序**通常安装在**MBR/ESP**中，**MBR**是计算机启动时首先读取的**512字节**的扇区，**ESP**是**EFI系统分区**，用于存放**EFI引导程序**
- 这里主要讲讲广泛使用的 **GRUB 2(以下简称GRUB)** 的安装配置：
  1. 使用包管理器安装**GRUB**；**UEFI**系统需要额外安装**efibootmgr**；**多系统**需要安装**os-prober**
  2. 写入**GRUB**到**MBR/ESP**，***确保你的分区正确挂载！！***：
    ```bash
    # 安装GRUB到MBR for Legacy BIOS
    grub-install --target=i386-pc /dev/sda
    # 安装GRUB到ESP for UEFI
    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Linux
    ```

    > 其中**target**对应**CPU架构与系统平台**，`/dev/sda`应为主要磁盘，**efi-directory**对应**EFI分区的挂载点**，**bootloader-id**对应**引导程序显示的id（名称）**

  3. 配置**GRUB**：
    ```bash
    # 编辑GRUB配置文件
    vim /etc/default/grub
    ```
    ```bash
    # GRUB配置文件示例
    GRUB_DEFAULT=0 # 默认启动项，0表示第一个启动项
    GRUB_TIMEOUT=5  # 启动等待时间，单位为秒，-1表示无限等待
    GRUB_CMDLINE_LINUX="quiet splash" # 内核参数，quiet表示不输出内核信息，splash表示显示启动画面
    GRUB_DEVICE="设备 eg. /dev/sda" # 系统设备，用于查找根文件系统
    GRUB_DISABLE_OS_PROBER=false # 是否禁用os-prober，false表示不禁用，true表示禁用
    ```
    ```bash
    # 生成GRUB配置文件
    grub-mkconfig -o "grub安装点 eg. /boot/efi/grub/grub.cfg"
    ```
    + 此命令会根据 **/etc/default/grub**文件中的配置生成**GRUB**配置文件，并保存到 **grub安装点**中
  4. 检查一下**GRUB**是否正确安装：
    ```bash
    # 查看GRUB安装点
    ls "grub安装点 eg. /boot/efi/grub"
    ```

> 第二阶段引导硬件已经初步初始化，因此可以实现**图形化**，这意味着你可以通过配置文件自定义**引导程序**的**界面**，如**GRUB**的**主题**、**字体**、**背景**等

### 重启
- 使用`Ctrl+D`组合键退出Chroot环境，使用`reboot`命令重启系统，进入BIOS/UEFI界面，将引导程序设置为第一个启动位置，保存设置并重启系统，进入Linux系统

  > Linux安装完成后**LiveCD**也不要丢弃，它可以作为系统修复的环境，类似**WinPE**
