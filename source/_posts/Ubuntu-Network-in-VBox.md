---
title: VBox中Ubuntu在不同连接方式下的网络配置
date: 2016-10-28 13:25:32
tags: VBox,Ubuntu,network
---

## 背景

VirtualBox®网卡现有6种连接方式，每一种连接方式在设置的时候都有一些坑。通常，一个虚拟机上会同时使用多个网卡，这样在设置的时候就会更坑。当然，网络专业的大牛请自动掠过！
下面介绍一些常用的设置方式，以备不时之需。

## 环境

平台Windows 7(64)，所需软件如下：

* VirtualBox (VirtualBox-5.1.8-111374-Win.exe)
* Ubuntu Server 16.04(64)(mini.iso安装)
* Xshell (可选)

## 网络配置

Ubuntu 16.04的网络配置文件

```
/etc/network/interfaces
```

刚装好的系统，如果**网卡1** 的连接方式是**网络地址转换(NAT)**，配置如下：

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto enp0s3
iface enp0s3 inet dhcp
```
其中**enp0s3**是虚拟机**网卡1**在Ubuntu中的设备名，**网卡2**的设备名为**enp0s8**，**网卡3**的设备名为**enp0s9**，**网卡4**的设备名为**enp0s10**。
以下各种连接方式的配置，只需追加到这个配置文件就可以了。

### 网络地址转换(NAT)

这是最常用的一种连接方式，用于虚拟机借道主机的网络连接外网。它最大的特点是虚拟机能访问主机的网络，但是主机却不能访问虚拟机，也就是无法从外部访问虚拟机。
所以用途单一，只需默认设置就可。
如果**网卡3**的连接方式为**网络地址转换(NAT)**，配置如下：

```
auto enp0s9
iface enp0s9 inet dhcp
```

### NAT 网络



### 桥接网卡



### 内部网络



### 仅主机(Host-Only)网络

```
# Host Only Network
auto enp0s8
iface enp0s8 inet static
    address 192.168.56.99
    network 192.168.56.0
    netmask 255.255.255.0
    broadcast 192.168.56.255
```

### 通用驱动




















