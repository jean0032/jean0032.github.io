---
title: Linux&Windows时间统一
date: 2022-06-30 10:38:30
tags: [linux, windows]
categories: Linux
---

### 时间不一致的原因

windows与liunx看待系统硬件时间的方式是不一样的。windows把计算机硬件时间当作地方时（local time），即东八区时间。但是linux把计算机硬件时间当作世界统一时间（UTC），所以linux系统时间会在硬件时间基础上增加电脑设置的时区数（东八区就加上8）。所以linux时间设置正确，相对应的windows时间就会慢8小时。

### ubuntu系统

把ubuntu时间更新到计算机硬件时间上，在ubuntu的终端上输入如下代码：

```shell
# 更新ubuntu的系统时间
sudo apt-get update
sudo apt-get install ntpdate
sudo ntpdate time.windows.com
# 将时间更新到硬件上
sudo hwclock --localtime --systohc
```

然后重新进入Windows，发现时间恢复正常了

### windows系统

让windows把计算机硬件时间当作UTC，在命令提示符（需要管理员权限）下输入：

```powershell
Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
```
