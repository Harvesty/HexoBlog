---
title: 让 VBox 虚拟机在后台运行
date: 2016-10-24 20:11:46
tags: VBox
---

## 背景

在开发环境下，多数时候虚拟机作为服务器使用，是不需要显示界面的。所以就想让它在后台运行，一方面，不显示界面运行时，虚拟机占用的系统资源较少；另一方面，解放了任务栏的一个位置。

## 环境

主机Windows，VirtualBox版本VirtualBox-5.1.8-111374-Win.exe。

## 操作

打开VirtualBox管理器，在需要后台运行的虚拟机上鼠标右击，选择*创建桌面快捷方式*。
在桌面找到快捷方式，右击*属性*，打开属性编辑窗口。
在目标下可以看到类似如下的内容：

```
"C:\Program Files\Oracle\VirtualBox\VirtualBox.exe" --comment "CentOS" --startvm "e03eee5f-692f-43c4-967d-9b2b8ac5c582"
```

修改为：

```
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" startvm "e03eee5f-692f-43c4-967d-9b2b8ac5c582" --type headless
```

这样，每次双击这个快捷方式，虚拟机就在后台自动运行了。