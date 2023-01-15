---
title: "Manjaro Linux默认LUKS全盘加密解密慢，启动慢"
date: 2023-01-15T12:22:26+08:00
categories: 
    - 笔记
tags:
    - linux
    - 软件安全
image: images/cover.webp
---
## 前言
我的manjaro安装的时候选择了全盘安装，勾选luks加密。加密虽好，但是我发现这里有个问题，manjaro启动的时候解密时间非常的长，需要10s左右。这显然不正常。经过搜索，我发现这个问题很可能跟LUKS的iter-time这个参数有关[https://bbs.archlinux.org/viewtopic.php?id=217193](https://bbs.archlinux.org/viewtopic.php?id=217193)。以下是archwiki里面关于iter-time参数的介绍：
>Number of milliseconds to spend with PBKDF2 passphrase processing. Release 1.7.0 changed defaults from 1000 to 2000 to "try to keep PBKDF2 iteration count still high enough and also still acceptable for users."[2]. This option is only relevant for LUKS operations that set or change passphrases, such as luksFormat or luksAddKey. Specifying 0 as parameter selects the compiled-in default.. 

大体上就是LUKS等待PBKDF2解密密钥的时间。默认是2000ms，这个数字对于新的机器来说其实有点太高了，现代cpu大多不需要这么长的解密时间。所以我们可以适当缩短这个时间。

## 正文
参考[https://unix.stackexchange.com/questions/690138/how-to-modify-iter-time-on-a-existing-luks-partition](https://unix.stackexchange.com/questions/690138/how-to-modify-iter-time-on-a-existing-luks-partition),我们可以修改这个iter-time。使用以下命令：
```bash
sudo cryptsetup luksChangeKey <device> --iter-time <time in ms>
```
`<device>`填写LUKS加密的分区，可以用`lsblk`查看。`<time in ms>`填写以毫秒为单位的iter-time，我填了300，也有看到有人填60的。取决于你的cpu。这还会要求你修改密码，如果你不希望修改，你可以直接填写原来的密码。注意，**这个命令需要执行两次**，原因我下面讲。

表面上看，只需要执行一次命令，修改一次就可以了。然而实际上还是有问题。manjaro的全盘加密默认注册了两个key slot。slot 0是通过我们设置的密码加密的，而slot 0解锁之后会解密一个密钥文件，通过这个密钥再来解密slot 1，进而解密整个硬盘。上面的命令会修改slot 0，但是修改后却会将其放在slot 2的位置。LUKS是按顺序尝试解锁的，这会导致LUKS先尝试解锁slot 1，解锁失败后再尝试slot 2，白白浪费了时间。怎么办呢？很简单，我们再执行一次这个命令。这个命令是将修改后的key放在最小的空slot。第一次执行命令时，slot 0和slot 1都被占用，所以新key放在了slot 2。这时slot 0才被清除。所以第二次执行命令时，slot 1和slot 2都被占用，新key就会被放回slot 0。这样就能达成我们的目的了。

## 结语
经过以上调整，我的开机解密时间缩短到了原来的1/10不到，大大提高了开机速度，让我觉得之前的开机都是在浪费生命XD。另外也了解到了不少关于LUKS的知识。可见折腾好处多多，能用就行的思想要不得😛。