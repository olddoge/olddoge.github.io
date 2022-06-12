---
title: 解决 Nginx 下 ThinkPHP 路由失效
date: 2021-11-14 18:59:31
tags: [ThinkPHP, PHP]
categories: PHP
description: Nginx 配置 ThinkPHP 和 Laravel 伪静态
---

在 `Nginx` 对应的站点配置中添加下列配置，`Laravel`框架有此问题也是类似的解决方案

```shell
location / { 
	if (!-e $request_filename) {
		rewrite ^(.*)$ /index.php?s=/$1 last; break; 
	} 
}
```

 