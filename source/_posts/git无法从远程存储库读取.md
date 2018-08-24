---

title:  git无法从远程存储库读取
date: 2018-07-09
categories:
- git
tags:
- git

---

# git无法从远程存储库读取



## 生成SSH Key

`ssh-keygen -t rsa -C  "mo_dishuai@126.com"`

## 找到刚生成Key值所在目录

`~/.ssh` 目录 包含三个文件，id_rsa、id_rsa.pub、known_hosts



## 登录个人GitHub 账号，添加SSH and GPG keys

打开生成好的文件，`id_rsa.pub` 复制文件中的内容添加到 SSH key
