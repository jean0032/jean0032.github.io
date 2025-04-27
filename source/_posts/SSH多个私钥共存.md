---
title: SSH多个私钥共存
date: 2022-03-19 15:20:44
tags: [linux, ssh]
categories: Linux
---
在操作多个git仓库或者ssh时，ssh私钥存在冲突，这时可以通过*config*管理多个私钥及对应地址。
#### 首先，生成密钥时可以通过指定不同名称后缀的形式，避免覆盖之前的密钥：
```  bash
ssh-keygen -t rsa -f ~/.ssh/id_rsa.work -C "Key for Work stuff"
ssh-keygen -t rsa -f ~/.ssh/id_rsa.github -C "Key for GitHub stuff"
```
#### 创建ssh配置文件*config*，并修改文件权限：
```  bash
touch ~/.ssh/config
chmod 600 ~/.ssh/config
```
#### 修改*config*文件内容：
```  bash
Host *.workdomain.com
    IdentityFile ~/.ssh/id_rsa.work
    User lee

Host github.com
    IdentityFile ~/.ssh/id_rsa.github
    User git
```
#### 修改完毕后，再操作git仓库或者ssh时，就会按对应地址自动选择私钥。
