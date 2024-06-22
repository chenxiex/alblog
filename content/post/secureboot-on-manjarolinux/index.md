---
title: "在全盘加密的Manjaro Linux上配置安全启动(Secure Boot)"
date: 2023-01-03T22:10:46+08:00
categories: 
    - 笔记
tags:
    - Linux
    - 软件安全
image: images/cover.webp
---
## 本文已过期。GRUB的一次更新要求GRUB使用的字体也必须签名，这让GRUB配置安全启动变得麻烦。如果要配置Secure Boot，推荐换用systemd-boot或rEFInd等引导器。对于这些引导器，Arch Wiki的教程已非常详细。例如，我现在采用的systemd-boot+sbctl的方案，可参考[https://wiki.archlinuxcn.org/wiki/UEFI/%E5%AE%89%E5%85%A8%E5%90%AF%E5%8A%A8#sbctl](https://wiki.archlinuxcn.org/wiki/UEFI/%E5%AE%89%E5%85%A8%E5%90%AF%E5%8A%A8#sbctl)
## 前言
本文主要介绍使用shim+MOK+grub2实现安全启动全盘加密的Manjaro Linux。

首先说一下我的环境，我的电脑是宏碁spin5，Manjaro是使用的全盘安装，btrfs格式分区，勾选全盘加密。

在开始以下工作之前，建议先将secureboot恢复到出厂设置。
## 分析
首先当然要看看[Arch Wiki](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#top-page)。以下内容也大量借鉴了这篇wiki。

这里提到要发挥secure boot安全功能的建议：
1. 设置一个强的硬件设置密码。（一般来说，在bios里面设置）
2. 不使用默认的厂商密钥和第三方密钥。根据Archwiki的警告，有变砖风险，这里就不折腾了。
3. uefi直接启动内核，包括microcode和initramfs，不使用启动加载器。这主要是缩短启动信任链的长度，减少被攻击的环节。这个方案推荐和第2条一起食用，这样比较方便，这里也不折腾了。
4. 启用全盘加密。避免能碰到你硬件的人乱动硬盘内容。
5. 使用TPM加强secure boot。

然后简单看看实现secureboot的方案。
1. 首先就是使用自己的密钥，优点是所有情况都在自己掌握之中，你可以完全控制签名哪些启动项来启动，非常安全。缺点是容易变砖，另外你需要对每个启动项签名，对于windows你也要自己配置。这里不采用这种方案，因为怕变砖。
2. 其次是使用已经经过微软签名的启动加载器。这样的加载器有两个，一个是preload，一个是shim。preload只能使用hash来验证启动项目，并且在preload阶段使用它的hash tool来导入hash。这一来比较麻烦，每次启动加载器或者内核更新都要重新导入一遍hash，hash多了还要自己清理，二来preload阶段全盘加密没有解开，hash tool也访问不到内核镜像。所以放弃这个方案，使用shim。shim可以用hash，也可以用MOK，我们这里介绍的是shim+MOK的方案。

## 正文
### 释义
下文中，\$esp指的是你的efi分区位置。请在所有提到\$esp的地方自觉替换。
下文中大多数命令需要root权限执行。
### 软件安装
你需要安装以下软件：
1. [shim-signed](https://aur.archlinux.org/packages/shim-signed/) (aur)
2. [sbsigntools](https://archlinux.org/packages/?name=sbsigntools)

### 设置shim
（注：本节部分有误。bootx64.efi是systemd-boot的默认位置。）
备份bootx64.efi。这一般认为是电脑的默认启动项，我们用shim来代替他。
```bash
mv $esp/EFI/boot/bootx64.efi $esp/EFI/boot/bootx64.efi.bak
```
将shim复制到原来bootx64.efi的位置
```bash
cp /usr/share/shim-signed/shimx64.efi $esp/EFI/BOOT/BOOTx64.EFI
cp /usr/share/shim-signed/mmx64.efi $esp/EFI/BOOT/
```
最后，创建一个新的nvram入口指向shim
```bash
efibootmgr --unicode --disk /dev/$disk --part $Y --create --label "Shim" --loader /EFI/BOOT/BOOTx64.EFI
```
这里的\$disk请替换成你的efi分区所在磁盘，大多数人应该是sda或者sdb，但是像我就是nvme0n1。\$Y请替换成你的efi在这块磁盘上是第几个分区。这些可以通过`lsblk`命令得到。

### 设置grub
接下来，我们要对grub进行设置。要使grub能在secureboot模式下启动，grub必须集成了能让它读取到vmlinuz和initramfs的所有模块。你可以参考[ubuntu的grub生成脚本](https://git.launchpad.net/~ubuntu-core-dev/grub/+git/ubuntu/tree/debian/build-efi-images?h=debian/2.06-2ubuntu12)来选择你所需要的模块。然后，将这些模块记录成环境变量使用。

作为参考，我是这么处理的。新建`/etc/grub_modules`文件，内容如下：
```bash
CD_MODULES="
	all_video
	boot
	btrfs
	cat
	chain
	configfile
	echo
	efifwsetup
	efinet
	ext2
	fat
	font
	gettext
	gfxmenu
	gfxterm
	gfxterm_background
	gzio
	halt
	help
	hfsplus
	iso9660
	jpeg
	keystatus
	loadenv
	loopback
	linux
	ls
	lsefi
	lsefimmap
	lsefisystab
	lssal
	memdisk
	minicmd
	normal
	ntfs
	part_apple
	part_msdos
	part_gpt
	password_pbkdf2
	png
	probe
	reboot
	regexp
	search
	search_fs_uuid
	search_fs_file
	search_label
	sleep
	smbios
	squash4
	test
	true
	video
	xfs
	zfs
	zfscrypt
	zfsinfo
	cpuid
	play
	tpm
	"
GRUB_MODULES="$CD_MODULES
	cryptodisk
	gcry_arcfour
	gcry_blowfish
	gcry_camellia
	gcry_cast5
	gcry_crc
	gcry_des
	gcry_dsa
	gcry_idea
	gcry_md4
	gcry_md5
	gcry_rfc2268
	gcry_rijndael
	gcry_rmd160
	gcry_rsa
	gcry_seed
	gcry_serpent
	gcry_sha1
	gcry_sha256
	gcry_sha512
	gcry_tiger
	gcry_twofish
	gcry_whirlpool
	luks
	lvm
	mdraid09
	mdraid1x
	raid5rec
	raid6rec
	"
```
之后source这个文件即可。

>>> 注意：如果你在grub启动后遭遇`unkown tpm error`问题，请删除`tpm`模块。

除了这些模块，你的grubx64.efi还需要有一个sbat小节。以下是完整的生成命令：
```bash
source /etc/grub_modules
grub-install --target=x86_64-efi --efi-directory=$esp --modules="${GRUB_MODULES}" --sbat /usr/share/grub/sbat.csv
```

### 签名
然后你需要创建MOK，也就是机器所有者密钥。这里建议你选择一个合适的工作目录进行生成。我选择了创建一个/etc/MOK目录。生成密钥并且签名。
```bash
mkdir -p /etc/MOK
cd /etc/MOK
openssl req -newkey rsa:4096 -nodes -keyout MOK.key -new -x509 -sha256 -days 3650 -subj "/CN=my Machine Owner Key/" -out MOK.crt
openssl x509 -outform DER -in MOK.crt -out MOK.cer
sbsign --key MOK.key --cert MOK.crt --output /boot/$vmlinuz /boot/$vmlinuz
sbsign --key MOK.key --cert MOK.crt --output $esp/EFI/Manjaro/grubx64.efi $esp/EFI/Manjaro/grubx64.efi
```
这里的`$vmlinuz`请替换成你自己的linux内核镜像文件名。你可以用
```bash
ls /boot | grep 'vmlinuz'
```
得到你需要签名的文件的完整列表。

这里的`$esp/EFI/Manjaro`是Manjaro的grub默认配置文件生成grub的位置。这个位置可能有以下几种情况：
- 在没有配置文件的情况下执行`grub-install`，将会生成在`$esp/EFI/grub`
- 在通过`grub-mkconfig`等命令创建了grub配置文件的情况下，这个位置由`/etc/default/grub`中的`GRUB_DISTRIBUTOR`指定。例如在Manjaro中，这个值默认是`manjaro`，所以最终生成目录在`$esp/EFI/Manjaro`
请仔细确认你的grub生成位置。下文中提到`$esp/EFI/Manjaro`的地方，请自行替换。

签名完成后，将生成的`MOK.cer`复制到$esp/目录，为后续导入做准备。
### 完成
最后，将grub复制到shim同目录下。
```bash
cp $esp/EFI/Manjaro/grubx64.efi $esp/EFI/boot/grubx64.efi
```
然后重启电脑，启动项选shim进入shim，这时shim会有一个错误提示界面。不要慌，这是因为你的MOK文件还未导入MOK list。按页面提示进入MokManager，然后选择Enroll key，选择刚刚复制过来的`MOK.cer`，按提示导入这个密钥即可。
### 后续工作
虽然现在你的电脑已经支持secureboot了，但是对grub和对vmlinuz的签名是一次性的，一旦这两个软件更新，你就需要重新签名。这显然非常不方便。为此，我们可以设置两个pacman钩子，在这两个软件更新的时候自动进行签名。因为每个人终端环境不一样，比如我就无法直接使用archwiki上记载的hook。所以此处仅供参考。需要注意的是，以下命令并不检查签名情况，在已经签名的情况下依然会附加另一个签名，可能导致内核和grub不断增大。但以一般人更新的频率来看，应该不会有太大影响。

创建`/usr/local/bin/sign_kernel_for_secureboot.sh`，内容如下：
```bash
#! /bin/sh
/usr/bin/find /boot/ -maxdepth 1 -name 'vmlinuz-*' -exec /usr/bin/sh -c '/usr/bin/sbsign --key /etc/MOK/MOK.key --cert /etc/MOK/MOK.crt --output {} {};' \;
```
并且为其赋予可执行权限：
```bash
sudo chmod +x /usr/local/bin/sign_kernel_for_secureboot.sh
```
创建`/etc/pacman.d/hooks/999-sign_kernel_for_secureboot.hook`，内容如下：
```bash
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = linux
Target = linux-lts
Target = linux-hardened
Target = linux-zen
Target = linux515

[Action]
Description = Signing kernel with Machine Owner Key for Secure Boot
When = PostTransaction
Exec = /usr/local/bin/sign_kernel_for_secureboot.sh
Depends = sbsigntools
Depends = findutils
```
这里的`$linuxX`自行替换成你的linux内核包名。例如对于manjaro的5.15内核，这里应该替换为linux515。


然后是对grub进行签名的钩子。

新建 `/usr/local/bin/sign_grub_for_secureboot.sh` 内容如下：
```bash
#! /bin/bash
source /etc/grub_modules
grub-install --target=x86_64-efi --efi-directory=$esp --modules="${GRUB_MODULES}" --sbat=/usr/share/grub/sbat.csv
sbsign --key /etc/MOK/MOK.key --cert /etc/MOK/MOK.crt --output $esp/EFI/Manjaro/grubx64.efi $esp/EFI/Manjaro/grubx64.efi
cp $esp/EFI/Manjaro/grubx64.efi $esp/EFI/boot/grubx64.efi
```
并且为其赋予可执行权限。
```bash
sudo chmod +x /usr/local/bin/sign_grub_for_secureboot.sh
```
新建 `/etc/pacman.d/hooks/999-sign_grub_for_secureboot.hook` 内容如下：
```bash
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = grub

[Action]
Description = Signing kernel with Machine Owner Key for Secure Boot
When = PostTransaction
Exec = /usr/local/bin/sign_grub_for_secureboot.sh
Depends = sbsigntools
Depends = grub
```
完成。