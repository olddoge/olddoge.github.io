---
title: CentOS7 下搭建 LAMP + FreeRadius + Daloradius Web 管理
date: 2021-11-22 20:42:34
tags: [Linux, FreeRadius, Radius]
categories: Linux
description:
---

> FreeRadius 服务官网：[http://freeradius.org/](http://freeradius.org/)
> Daloradius Web 管理页面官网：[https://sourceforge.net/projects/daloradius/](https://sourceforge.net/projects/daloradius/)
<!--more-->
# 创建 radius 数据库

创建 radius 数据库和用户名密码

```shell
mysql -h 127.0.0.1 -u root -p
# 下列在 mysql 命令行中操作
mysql> create database radius;
mysql> grant all on radius.* to radius@"localhost" identified by "radpass";
mysql> flush privileges;
mysql> exit
```

# 安装 FreeRadius

```shell
yum -y install freeradius freeradius-utils freeradius-mysql
systemctl start radiusd.service
systemctl enable radiusd.service
```
查看 Radius 使用的端口，然后添加 Radius 服务到防火墙中；也可以直接关闭防火墙

```shell
cat /usr/lib/firewalld/services/radius.xml
firewall-cmd --state # 查看防火墙状态，启动状态才能添加规则
firewall-cmd --add-service=radius --permanent # 添加 Radius 服务到 firewalld 防火墙中。
firewall-cmd --reload # 让配置的防火墙策略立即生效
firewall-cmd --list-services # 列举区域中启用的服务
```

# 配置 FreeRadius

```shell
cd /etc/raddb/mods-config/sql/main/mysql/
# 备份 setup.sql 配置文件
mv setup.sql setup.sql-backup 
# 过滤有 # 注释符号的信息行
grep -v "#" /etc/raddb/mods-config/sql/main/mysql/setup.sql-backup > /etc/raddb/mods-config/sql/main/mysql/setup.sql 

# 进入数据库命令行操作
mysql -h 127.0.0.1 -u root -p
mysql> source /etc/raddb/mods-config/sql/main/mysql/setup.sql 
mysql> use radius 
mysql> source /etc/raddb/mods-config/sql/main/mysql/schema.sql 
mysql> exit; 
```
导入完成后，可以看到如下的表结构

```shell
+------------------+
| Tables_in_radius |
+------------------+
| nas              |
| radacct          |
| radcheck         |
| radgroupcheck    |
| radgroupreply    |
| radpostauth      |
| radreply         |
| radusergroup     |
+------------------+
```

`MySQL` 中表结构的定义
针对 `FreeRadius3`，数据表的设计和结构定义在下面的文件中：`/etc/raddb/sql/mysql/schema.sql` 
主数据库定义，8个表，包括
`nas # 网络设备表`
`radacct # 计费情况表`
`radcheck # 用户检查信息表`
`radgroupcheck # 用户组检查信息表`
`radgroupreply # 用户组回复信息表`
`radpostauth # 认证后处理信息，可以包括认证请求成功和拒绝的记录。`
`radreply # 用户回复信息表`
`radusergroup # 用户和组关系表 `

为 `/etc/raddb/mods-enabled` 创建软连接（涉及到后续 Radius 验证是否使用数据库）

```shell
ln -s /etc/raddb/mods-available/sql /etc/raddb/mods-enabled/
```
配置 `SQL` 模块 `/raddb/mods-available/sql`，并更改数据库连接参数，以适合自己的配置环境：

```shell 
vi /etc/raddb/mods-available/sql
```
```apacheconf
31 # driver = "rlm_sql_null" # 注释掉这行
32 driver = "rlm_sql_mysql" # 新增这行，表示使用 MySQL 数据库
88 # dialect = "sqlite"  # 注释掉这行
89 dialect = "mysql" # 修改连接类型为 MySQL
90
91 # Connection info:
92 # (这里可以改成root账号) 
93 server = "localhost"
94 port = 3306
95 login = "radius"
96 password = "radpass" # 对应数据库密码
97
98 # Database table configuration for everything except Oracle
99 radius_db = "radius"
247 read_clients = yes
250 client_table = "nas"
其他配置默认无需更改。
```
然后，将 `/etc/raddb/mods-enabled/sql` 所属组更改为 `radiusd`：
```shell
chgrp -h radiusd /etc/raddb/mods-enabled/sql
```

添加启动服务，调整 `FreeRadius` 与 `MariaDB` 的启动顺序，`FreeRadius` 必须在 `MariaDB` 启动之后启动，在 `[Unit]` 部分，增加 `After=mariadb.service`，如下所示：

```shell
systemctl enable radiusd.service
vi /etc/systemd/system/multi-user.target.wants/radiusd.service
```
```apacheconf
[Unit]
Description=FreeRADIUS high performance RADIUS server.
After=syslog.target network.target ipa.service dirsrv.target krb5kdc.service
After=mysqld.service
```
添加客户端（路由器、交换机等设备）连接设置，ip 可以自由更改。

```shell
mv /etc/raddb/clients.conf /etc/raddb/clients.conf-backup
grep -v "#" /etc/raddb/clients.conf-backup > /etc/raddb/clients.conf # 过滤有 # 注释符号的信息行 
vi /etc/raddb/clients.conf
```
```apacheconf
client localhost {
    ipaddr = 127.0.0.1
    proto = *
    secret = testing123
    require_message_authenticator = no
    limit {
        max_connections = 16
        lifetime = 0
        idle_timeout = 30
    }
}
```

需要注意，上面配置的 `127.0.0.1` 主要用于测试，将来真正的客户端要按照下面的信息进行补充：
将来你真正的 `raidus` 客户端，如路由器、交换机等需要在这里配置 `ip` 信息，例如：

```apacheconf
# 这里的 x.x.x.x 就是交换机或者路由器等设备的IP地址
client x.x.x.x {
    ipaddr = x.x.x.x # 交换机或者路由器等设备的IP地址
    secret = xxxxxxxxxx # 自己定义的交换机或者路由器等设备和 radius 服务器的连接密码
    # require_message_authenticator = no # 这个可以注释掉，不用管
    
    client 0.0.0.0/0 {
        secret = testing123
    }
}
```
可以执行 `cat /var/log/radius/radius.log` 去查看 `radius` 服务的日志看有没有错误。

# 安装 FreeRADIUS 管理界面 Daloradius

进入 `Apache` 网站根目录，下载 `daloradius` 源文件

```shell
yum install -y php-pear-DB
cd /var/www/html/
wget https://github.com/lirantal/daloradius/archive/master.zip
unzip master.zip
mv daloradius-master/ daloradius
```
下载 `daloradius-0.9-9.tar.gz`，解压后合并到 `daloradius` 文件夹中

```shell
wget http://liquidtelecom.dl.sourceforge.net/project/daloradius/daloradius/daloradius0.9-9/daloradius-0.9-9.tar.gz
tar -zxvf daloradius-0.9-9.tar.gz
mv daloradius-0.9-9 daloradius
```
进入 `daloradius` 目录，导入 `daloradius` 数据库

```shell
cd daloradius
mysql -h 127.0.0.1 -u root -p radius < contrib/db/fr2-mysql-daloradius-and-freeradius.sql
mysql -h 127.0.0.1 -u root -p radius < contrib/db/mysql-daloradius.sql
```

复制一份 `daloradius.conf.php.sample` 为 `daloradius.conf.php`

```shell
mv /var/www/html/daloradius/library/daloradius.conf.php.sample /var/www/html/daloradius/library/daloradius.conf.php
```
设置 `daloradius` 目录用户组和用户，设置 `daloradius.conf.php` 权限

```shell
chown -R apache:apache /var/www/html/daloradius/
chmod 664 /var/www/html/daloradius/library/daloradius.conf.php
```

设置 `daloradius` 数据库连接信息，打开 `daloradius.conf.php` 文件，修改 `CONFIG_DB_USER`，`CONFIG_DB_PASS`，`CONFIG_DB_NAME`。

```shell
vi /var/www/html/daloradius/library/daloradius.conf.php
```
找到下列变量进行修改

```php
// （28-33行）
$configValues['CONFIG_DB_ENGINE'] = 'mysql';
$configValues['CONFIG_DB_HOST'] = '127.0.0.1';
$configValues['CONFIG_DB_PORT'] = '3306'; // 连接 mysql 数据库的端口
$configValues['CONFIG_DB_USER'] = 'root'; // 连接 mysql 数据库的账户
$configValues['CONFIG_DB_PASS'] = ' ';  // 连接 mysql 数据库账号的密码
$configValues['CONFIG_DB_NAME'] = 'radius'; // 连接 mysql 的 radius 数据库
// 下面还有几个 daloradius 的 bug，默认配置中有几个文件路径和我们导入的不一样，把它改过来：
$configValues['CONFIG_FILE_RADIUS_PROXY'] = '/etc/raddb/proxy.conf'; //（68行）
$configValues['CONFIG_PATH_DALO_VARIABLE_DATA'] = '/var/www/html/daloradius/var'; // （70行）
$configValues['CONFIG_MAINT_TEST_USER_RADIUSSECRET'] = 'testing123'; // （88行） # 注意这条，要和 /etc/raddb/clients.conf 文件设置的secret = xxxxxxxxxx 值一样。
```

配置完成后保存退出
重启 `radius`、`mysql`、`http` 服务

```shell
systemctl restart radiusd
systemctl restart mysqld
systemctl restart httpd
```
如果提示： `Warning: radiusd.service changed on disk. Run ‘systemctl daemon-reload’ to reload units.` 执行以下的命令，没有上面提示就忽略此步
```shell
systemctl daemon-reload # 重新加载守护进程
systemctl restart radiusd
```
现在打开浏览器访问 `http://your ip address//` 就可以看到 `daloradius` 的界面了，默认登录的用户名和密码分别为 `username: administrator`, `password: radius`
至此 `LAMP` + `FreeRadius` + `Daloradius Web` 管理界面已经安装成功，下面是 Web 界面汉化教程。

# Daloradius 中文版设置
进入 `Daloradius` 文件目录,修改 `config-lang.php`,添加中文选项：

```shell
vi /var/www/html/daloradius/config-lang.php
# Simplified Chinese （79行）
```

# RADIUS 操作命令

1. 启动 `RADIUS` 服务(调试模式)

```shell
systemctl stop radiusd
radiusd -X
```   

2. 配置文件添加账号

```shell
vi /etc/raddb/users
#steve Cleartext-Password := "testing"（这个注释去掉）
```

3. 允许外部访问

```shell
vi /etc/raddb/clients.conf
```

```apacheconf
client 192.168.0.0/24 { # 地址根据实际情况去配置
    secret = testing123
}
```
4. 远程客户端登录 `RADIUS` 服务器

```shell
radtest steve testing 192.168.70.116 1812 testing123 
```
访问成功会收到 `Access-Accept` 的提示

