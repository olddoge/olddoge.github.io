---
title: hexo 部署到云服务器
date: 2022-04-23 19:40:24
tags: [Hexo, JS, 教程]
categories: 指南
---

云服务器购买、git 安装、hexo 安装等基础部分略，只整理一些我部署时遇到的问题


# 1. 创建仓库

首先创建一个空仓，这个仓库会存储`hexo`的相关代码（改动记录等）

```shell
git init --bare [你的仓库名称]
```
进入这个仓库目录的`hooks`文件夹下，创建一个新文件，名称为 `post-receive`

```shell
vim [你的仓库路径]/hooks/post-receive
```
写入以下内容
```shell
git --work-tree=[你的博客站点路径，hexo 生成的 html 文件会存储到这里] --git-dir=[你的仓库路径] checkout -f
```
保存后退出，赋予权限

```shell
chmod +x [你的仓库路径]/hooks/post-receive
```

# 2. 配置中填写部署的地址

在 `_config.yml`中填写自己服务器仓库地址

```yaml
deploy:
  branch: master
  type:  git
  repo:  ssh://[用户名]@[服务器地址]:[服务器端口]/[仓库路径] # 此处举例而已，适用于特定端口号的服务器
```

# 3. 最后

将`nginx`或`Apache`的站点路径指向对应的博客目录即可完成访问