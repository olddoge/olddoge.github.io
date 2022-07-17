---
title: 最新Jetbrains IDE激活教程
date: 2022-07-17 18:21:00
tags: [教程, 破解]
categories: 软件
description: 继 IDE Eval Reset 失效后,今天我们分享一个最新的jetbrains全家桶及phpstorm激活方法,并且支持全系列,全版本可更新使用并且有效。
---

> 最新jetbrains全家桶激活方法

原理是我们主要通过代码搜索其他授权服务器进行永久激活激活。

* 通过censys

地址：[https://search.censys.io/](https://search.censys.io/)

主要用到的代码

```
services.http.response.headers.location: account.jetbrains.com/fls-auth
```

我们复制上面用到的代码 `services.http.response.headers.location: account.jetbrains.com/fls-auth` 进入 `censys` 进行搜索。

![01](/pictures/post/2022/07/01.png)

可以看到出现了很多对应跳转到 jetbrains 的服务器 IP 和网址,我们随便点击一个看下状态是不是 `302` 只有 `302` 的才能 正常使用 。

![02](/pictures/post/2022/07/02.png)

然后我们复制域名或者 IP 到 jetbrains 全家桶进行激活,比如我们使用第一个 `49.234.70.205` 复制到 `License server`

![03](/pictures/post/2022/07/03.png)

可以看到已经连接到 jetbrains 授权服务器成功了,然后我们点击 `ACTIVATE` 进行启动就可以了。

![04](/pictures/post/2022/07/04.png)


jetbrains激活原理：

通过以上方式激活 jetbrains 全家桶 主要是用到了 爬取网站服务 这一类的 搜索引擎 实现的通过 搜索引擎 我们找到全世界的 jetbrains 授权服务器 进行激活。

注意事项

- 每个服务器IP承载激活的数量优先, 如果激活时候提示失败,可以多换几个.