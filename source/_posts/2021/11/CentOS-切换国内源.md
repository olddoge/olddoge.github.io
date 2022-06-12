---
title: CentOS 切换国内源
date: 2021-11-14 18:58:30
tags: [Linux]
categories: Linux
description: CentOS 切换国内源的方法
---

### 备份现有源
```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

### 下载新的 CentOS-Base.repo 到 /etc/yum.repos.d/
```shell
# CentOS 6
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo
# 或者
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo


# CentOS 7
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
# 或者
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

# CentOS 8
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
# 或者
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
```

### 缓存刷新
```shell
yum makecache
```

附：[常用的开源镜像](https://developer.aliyun.com/mirror/)