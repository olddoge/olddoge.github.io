---
title: Linux 常用命令
date: 2021-11-14 19:00:56
tags: [Linux]
categories: Linux
---



### 解压缩
```shell

# 压缩
tar -zcvf [压缩包名字].tgz [要压缩的文件或路径]
# 解压
tar -zxvf [文件名]

```

### 软连接
```shell
# 创建软连接
ln  -s  [源文件或目录]  [目标文件或目录]
# 修改软连接
ln –snf  [新的源文件或目录]  [目标文件或目录]
```

### 检查是否安装了某个软件
```shell
yum list installed | grep "软件名或者包名"
```

### 列出目录文件
```shell
ls -a #列出目录所有文件，包含以.开始的隐藏文件
ls -A #列出除.及..的其它文件
ls -l #除了文件名之外，还将文件的权限、所有者、文件大小等信息详细列出来
ls -t #以文件修改时间排序
ls -S #以文件大小排序
ls -h #以易读大小显示
```

### 创建文件夹
```shell
# 创建文件夹t
mkdir t
# 在 tmp 目录下创建路径为 test/t1/t 的目录，若不存在，则创建：
mkdir -p /tmp/test/t1/t
```

### cat 命令
```shell
# 一次显示整个文件
cat filename
# 从键盘创建一个文件 只能创建新文件，不能编辑已有文件
cat > filename
# 将几个文件合并为一个文件
cat file1 file2 > file
```
### df 命令
```shell
df -a # 全部文件系统列表
df -h # 以方便阅读的方式显示信息
df -l # 只显示本地磁盘
df -T # 列出文件系统类型
```

### 查看当前运行的进程状态
```shell
ps -A # 显示所有进程
ps -a # 显示同一终端下所有进程
ps c # 显示进程真实名称
ps e # 显示环境变量
ps f # 显示进程间的关系
ps r # 显示当前终端运行的进程
ps -aux # 显示所有包含其它使用的进程
```

