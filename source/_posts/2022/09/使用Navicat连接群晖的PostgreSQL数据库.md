---
title: 使用 Navicat 连接群晖的 PostgreSQL 数据库
date: 2022-09-12 19:03:23
tags: [PostgreSQL, 教程]
categories: 指南
description: 如何使用 Navicat 连接群晖（Synology）中的 PostgreSQL 数据库
---


群晖的类似 Moments、Audio station、Video Station 等套件都是使用群晖内置的一个 PostgreSQL 数据库，记录一下连接这个数据库的方法。温馨提示： 群晖多个套件均依赖此数据库，没有一定基础的同学，请做好备份，自行评估操作风险。

# 创建数据库用户并授权

打开ssh 连接并群晖获取 root 权限，切换到 posgres 用户，进入 psql 交互命令行，创建新用户并授权：

```shell
# 切换到postgres用户
su - postgres

# 进入posql交互
psql
```

```shell
# 创建用户
CREATE USER 用户名 PASSWORD '密码';

# 授权所有权限
ALTER USER 用户名 WITH SUPERUSER CREATEDB;
ALTER USER 用户名 WITH CREATEDB;
ALTER USER 用户名 WITH CREATEROLE;
ALTER USER 用户名 WITH REPLICATION;

# 查看用户列表 看是否执行成功
\du

# 退出psql
\q
```

```shell
# 退出postgres用户，回到root下
exit
```

修改 `pg_hba.conf`，将新建的用户授权登陆：

```shell
vi /etc/postgresql/pg_hba.conf
```

原始内容为：

```conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD

local   all             postgres                                peer map=pg_root
local   all             all                                     peer
```

我们新增一行，`host all 用户名 127.0.0.1/0 md5`，修改之后内容为：

```conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             用户名           127.0.0.1/0             md5
local   all             postgres                                peer map=pg_root
local   all             all                                     peer
```

重新载入配置文件:

```shell
su -l postgres -c "exec /usr/bin/pg_ctl reload"
```

# 修改群晖 sshd 配置，允许端口转发

不进行此步操作，后面使用 `ssh` 隧道连接数据库的时候，终端会出现类似 `channel 4: open failed: administratively prohibited: open failed` 的报错。

```shell
vi /etc/ssh/sshd_config
```

在约 `86` 行，找到`AllowTcpForwarding no` 将其改为 `AllowTcpForwarding yes`,保存退出。
直接登陆网页的 DSM 控制台，依次进入 控制面板——终端机和SNMP——终端机，将“启动SSH功能”取消勾选，应用之后，重新勾选并应用，即可重启 sshd 服务。

# 使用 Navicat 连接数据库
新建——PostgreSQL，连接名随意，主机填写`127.0.0.1`，端口填写 PostgreSQL 监听的端口 5432，初始数据库填写 `postgres`，用户名、密码为前面新建数据库用户设置的，在 “SSH” 页选择使用 SSH 通道，IP、用户名、密码使用群晖的 ssh 连接信息即可。

　　使用 macOS 的同学如果 Navicat 自带的 SSH 通道兼容性有问题的，可以本地使用SSH隧道进行端口转发：

```shell
# 1234为本地监听的端口，5432 为群晖上postgreSQL的监听端口
# root@192.168.1.2 -p 22 表示Alliot的群晖用户名为root，IP为192.168.1.2，sshd端口为22
# ssh -L 5432:127.0.0.1:1234 root@192.168.1.2 -p 22 (我是用这句成功的)
ssh -L 5432:127.0.0.1:5432 root@192.168.1.2 -p 22

# 查看macOS本地监听的端口，出现1234端口的监听则OK
netstat -AaLlnW
```
之后 Navicat 连接时，端口填 5432 。

# 删除用户，回收权限

操作完成后，如果需要回收权限，恢复到原样。只需要删除我们新增的那行 `pg_hba.conf` 配置文件的内容。进入 psql 交互命令中，执行：

```mysql
drop user 用户名;
```

即可。