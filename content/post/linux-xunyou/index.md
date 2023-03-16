---
title: "在Linux上使用迅游加速器"
date: 2023-03-12T23:19:32+08:00
categories:
    - 笔记
tags:
    - linux
    - 游戏
image: images/cover.webp
---
**注意：本文所述方法需要迅游SVIP方能使用**
## 前言
proton等技术的发展让linux的游戏能力越来越好了。然而对于国内的游戏玩家来说，还有一个大问题就是网络问题。目前国内的各个加速器都没有原生linux客户端。幸运的是，迅游加速器的主机加速功能支持通过l2tp和pptp vpn进行连接。这给linux玩家带来了机会。本文主要介绍在linux上通过配置l2tp vpn连接迅游加速器。

本文中系统使用的网络管理工具是networkmanager，这是被ubuntu、deepin、manjaro等主流系统所接受的网络管理工具。如果你使用的是其它网络管理工具，请自行查阅相关文档。

本文会涉及到的Linux发行版包括Arch/Manjaro，Ubuntu，Deepin。对于其它发行版，操作大同小异。

## 软件准备
### Arch/Manjaro
[networkmanager-l2tp](https://archlinux.org/packages/community/x86_64/networkmanager-l2tp/)
```bash
sudo pacman -Syu networkmanager-l2tp
```

### Ubuntu
```bash
sudo apt install network-manager-l2tp
```
如果你使用的是gnome桌面环境，还可以安装
```bash
sudo apt install network-manager-l2tp-gnome
```
来使用gnome的图形界面来设置l2tp vpn

### GUI
很多主流桌面环境在它们的设置中集成了networkmanager的设置。如果你的桌面环境没有提供此功能，你仍可通过`nm-connection-editor`来使用GUI配置networkmanager。**ubuntu已预装该软件**，对于Arch，它在extra仓库中，可以通过pacman安装：
[nm-connection-editor](https://archlinux.org/packages/?name=nm-connection-editor)

```bash
sudo pacman -Syu nm-connection-editor
```

## VPN配置
### 获取迅游节点信息
首先你需要有一个迅游SVIP帐号。之后打开[https://pay.xunyou.com/u/#host](https://pay.xunyou.com/u/#host)，访问迅游个人中心的**主机加速**页面。迅游提供了港服、日服、美服、欧服四个服务器的主机加速节点。我们一一对应地创建4个vpn，这样只要连接到不同的vpn，就能连接到不同的节点。
### Gnome图形界面
打开gnome设置，网络，VPN，点击加号，选择“第二层隧道协议(L2TP)“，名称取自己喜欢的，在”网关“一栏填入从迅游那里得到的服务器地址，用户名和密码栏照写。密码栏右侧小图标可以点击，建议改成仅为当前用户存储密码。然后点右上角添加即可。如图：
![第一步](images/1.webp)

![第二步](images/2.webp)

![第三步](images/3.webp)

之后直接在gnome网络设置界面打开开关即可连接到对应的vpn。也可以在gnome右上角控制栏里面找到切换开关。需要注意的是，如果你要切换到另一个节点，你需要先手动关闭前一个vpn连接再开启新的vpn连接，否则会报错。

### deepin(DDE)图形界面
Deepin预装了l2tp的vpn支持，并在它的网络设置界面提供了方便的设置选项。打开deepin的设置（控制中心），点击网络，vpn，点击加号，选择l2tp，名称取一个自己喜欢的，建议关闭自动连接，然后网关填入从迅游那里得到的服务器地址，用户名和密码照写，其它选项不变，保存即可。如图：
![第一步](images/4.webp)
![第二部](images/5.webp)

### 其它图形界面
对于kde等桌面环境，可参考上面两段进行配置。如果桌面环境没有提供相关设置项，可通过`nm-connection-editor`进行配置。
首先以管理员用户启动`nm-connection-editor`（非管理员用户会无法保存密码，即使仅为当前用户保存也一样。这是`nm-connection-editor`的bug）
```bash
sudo nm-connection-editor
```
然后点击左下角的+号，选择l2tp，然后填入名称（随意），网关（即迅游处获得的服务器地址），用户名和密码（从迅游处获得）。保存即可。然后用你喜欢的方式连接到对应vpn就行（例如用nmtui或者nm-applet）

## 总结
虽然这样只有4个节点，远远不如迅游客户端那么灵活。但是能达到加速的效果，而且与wine或者虚拟机方案比起来没有性能损耗，连接也非常方便，只要启用vpn即可。实测Apex在使用香港和日本节点加速的时候都能达到<50ms的延迟，相比于直连进步巨大，完全可玩。希望以后能有原生支持Linux的加速器吧。