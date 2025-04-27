---
title: Magisk Root Android
date: 2023-01-30 9:32:10
tags: [android, root]
categories: Android
---

### 前言

此操作前提为已经解锁BootLoader的前提下，解锁BL不同机型流程不同，甚至某些机型目前常规方法无法解锁。

### 安装Magisk

尽量从github中的[Magisk项目地址](https://github.com/topjohnwu/Magisk/releases)中下载最新的Magisk*.apk并安装。

### 修补boot.img

准备好当前手机系统的固件包，机型与版本必须完全一致。解压缩固件包得到其根目录下的boot.img,在Magisk程序中的**安装**界面点击*选择并修补一个文件*，然后找到boot.img。运行完毕会得到一个magisk_patched*.img。

### 正式Root

打开开发者模式，连接USB, PC上打开命令行。
```SHELL
# 取回修补好的magisk_patched*.img
adb pull /storage/emulated/0/Download/magisk_patched*.img
# 重启手机至引导模式
adb reboot bootloader
# 将magisk_patched*.img写入boot分区
fastboot flash boot magisk_patched*.img
```
重启手机，至此Root操作结束。

