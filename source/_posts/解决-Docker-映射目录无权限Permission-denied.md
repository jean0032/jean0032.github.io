---
title: 解决 Docker 映射目录无权限Permission denied
date: 2022-09-14 10:46:30
tags: [linux, docker]
categories: Linux
---

### Docker 映射目录时，容器无权限操作导致启动报错

```bash
docker run -v /home/yy/Tools:/Tools -p 8080:8080 hnsac/Tools
```

```shell
# 错误信息
PermissionError: [Errno 13] Permission denied: '/Tools/tools.yaml'
```

### 原因

Centos7默认安全模块selinux禁用了相关权限。

### 解决办法（3种）

#### ①在运行时追加特权 --privileged=true

```bash
docker run -v /home/yy/Tools:/Tools -p 8080:8080 hnsac/Tools --privileged=true
```
#### ②临时关闭selinux

```bash
setenforce 0
```

#### ③把相关挂载目录添加selinux白名单

> chcon [-R] [-t type] [-u user] [-r role] 文件或者目录
> 选项参数： 
>  -R  ：该目录下的所有目录也同时修改； 
>  -t  ：后面接安全性本文的类型字段，例如 httpd_sys_content_t ； 
>  -u  ：后面接身份识别，例如 system_u； 
>  -r  ：后面接角色，例如 system_r

```
chcon -Rt svirt_sandbox_file_t /home/yy/Tools
```
