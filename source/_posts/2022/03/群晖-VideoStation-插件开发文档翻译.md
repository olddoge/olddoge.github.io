---
title: 群晖 Video Station 插件开发文档翻译
date: 2022-03-12 21:07:00
tags: [群晖,Video Station, 翻译]
categories: 指南
description:
---

> 群晖 Video Station 插件官方开发文档翻译

# 1. 介绍

## 1.1 关于视频信息插件

从 DSM 7.0 的 Video Station 3.0.0 和 DSM 6.0 的 2.5.0 开始，您可以上传自己开发的视频信息索引插件
使用视频信息插件功能检索电影和电视节目的视频信息。
本文件规定了信息检索工作流程、文件要求、包装
视频信息插件的说明和测试详细信息。 
<!--more-->

[下载示例代码](https://global.download.synology.com/download/Addons/VideoStation/com.synology.TMDBExample.zip)

# 2. 信息搜索流程

要在 Video Station 中运行视频信息插件，主要步骤如下：

## 2.1 触发视频信息搜索

单击 Video Station 中的 "从视频信息插件搜索" 按钮后，PluginSearch API 将发送请求以触发视频信息搜索。 

## 2.2 转换 INFO 文件

Video Station 检查插件状态并检索插件 ID 和 INFO 文件中的入口文件路径 

## 2.3 运行插件

Video Station 找到入口文件 (loader.sh) 并以 "nobody" 权限运行它。 一旦信息搜索完成后，Video Station 将解析插件响应并将其保存到数据库 

![1](/pictures/post/2022/03/1.png)

# 3. 文件要求

## 3.1 INFO

INFO 文件提供插件 ID、支持的视频类型、入口文件位置和测试验证插件的示例。 内容必须使用 JSON 格式的 UTF-8 编码，通常如下所示

```json
{
  "id": "com.synology.TMDBExample",
  "description": "",
  "version": "1.0",
  "site": "http://www.themoviedb.org/",
  "entry_file": "loader.sh",
  "type": [
    "movie",
    "tvshow"
  ],
  "language": [
    "enu"
  ],
  "test_example": {
    "movie": {
      "title": "Harry Potter",
      "original_available": "2001-11-16"
    },
    "tvshow": {
      "title": "Game of Thrones",
      "original_available": "2011-04-17"
    },
    "tvshow_episode": {
      "title": "Game of Thrones",
      "original_available": "2011-04-17",
      "season": 1,
      "episode": 1
    }
  }
}
```

**表1. INFO 文件的内容**

|     key      |      类型       |                                                                                                                                                                                                    描述                                                                                                                                                                                                    | 是否必须 |
| :----------: | :-------------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :------: |
|      id      |     string      |                                                                                                                                                                 唯一的插件ID，应该和插件一样,作为文件夹名称。如果插件ID重复，插件不能上传                                                                                                                                                                  |   必须   |
|  entry_file  |     string      |                                                                                                                                                                                     入口文件loader.sh的相对文件路径。                                                                                                                                                                                      |   必须   |
|     type     | array of string |                                                                                                                                                             插件支持的视频类型，必须是以下之一：['movie']、['tvshow'] 或 ['movie', 'tvshow']。                                                                                                                                                             |   必须   |
|   version    |     string      |                                                                                                                                                                                                插件的版本号                                                                                                                                                                                                |   可选   |
| description  |     string      |                                                                                                                                                                                               插件的描述信息                                                                                                                                                                                               |   可选   |
|     site     |     string      |                                                                                                                                                                                                插件的源地址                                                                                                                                                                                                |   可选   |
|   language   | array of string |                                                                                                                                                                                    插件支持的语言 (e.g. ['cht', 'enu'])                                                                                                                                                                                    |   可选   |
| test_example |   JSON object   | 此值可确保您的插件在您使用时可用上传或测试插件的连接。 这是一个测试_电影的例子："movie": {"title": "Harry Potter", "original_available": "2001-11-16"}，如果插件支持电视剧类型的视频，这里提供一个相关类型的例子："tvshow": {"title": "Game of Thrones", "original_available": "2011-04-17"},"tvshow_episode": {"title": "Game of Thrones", "original_available": "2011-04-17", "season": 1, "episode": 1} |   必须   |

## 3.2 loader.sh

Video Station 将执行 loader.sh 以检索视频信息。 您可以使用 PHP 或 Python 3.x 搜索算法并在 loader.sh 中运行。 Video Station 将运行您的具有以下参数的插件： 

```sh
/bin/bash loader.sh --type movie --lang enu --input "{\"title\":\"Toy Story\", \"original_available\": \"1995-11-22\"}" --limit 1 --allowguess false
```

下表解释了 Video Station 传递的参数的详细信息

**表2. 发送给 loader.sh 的参数说明**

|    参数    |    类型     |                                                                  描述                                                                  | 是否必须 |
| :--------: | :---------: | :------------------------------------------------------------------------------------------------------------------------------------: | :------: |
|   input    | JSON object |  查询输入必须是 JSON 对象，包括以下内容：title（必填）和 original_available（可选）。搜索电视剧集的信息时，必须包含 episode 和 season  |   必须   |
|    lang    |   string    | 首选语言（必须是以下任何一种：chs、cht, csy, dan, enu, fre, ger, hun, ita, jpn, krn, nld, nor, plk, ptb, ptg、rus、spn、sve、trk、tha) |   必须   |
|    type    |   string    |                                    查询的视频类型, 必须是下列的一种: movie, tvshow, tvshow_episode                                     |   必须   |
|   limit    |     int     |                                                            允许的最大结果数                                                            |   必须   |
| allowguess |   boolean   |                                                如果 title guessing 功能支持的时候才可用                                                |   可选   |


> 1. 如果电视剧的 season 值为 0，意味着这是季的特别篇（e.g. 日剧的SP）。如果输入的参数没有 tvshow_episode (集的信息)，意味着将获得 season 的全部集信息
> 2. 标题猜测功能应该由插件实现。 可查看示例代码的 `searchinc.py` 的 `get_guessing_names` 方法 

# 4. 插件响应

所有插件响应都应编码为 JSON 对象。 搜索结果插件由键 "success" 表示，其值可以是真/假（布尔值）取决于请求的状态

## 4.1 成功响应

成功的响应将包含 "success" 的键，值为 "true", "result" 的值是 JSON 数组，内容因视频类型而异（例如，电影、电视节目、电视，显示剧集）

### 4.1.1 电影

电影的检索结果将包括以下属性：

```json
{
  "success": true,
  "result": [
    {
      "title": "Toy Story",
      "tagline": "",
      "original_available": "1995-10-30",
      "original_title": "Toy Story",
      "summary": "Led by Woody, Andy's toys live happily in his room until Andy's birthday brings Buzz Lightyear onto the scene. Afraid of losing his place in Andy's heart, Woody plots against Buzz. But when circumstances separate Buzz and Woody from their owner, the duo eventually learns to put aside their differences.",
      "certificate": "G",
      "genre": [
        "Animation",
        "Adventure",
        "Family",
        "Comedy"
      ],
      "actor": [
        "Tom Hanks"
      ],
      "director": [
        "John Lasseter"
      ],
      "writer": [
        "Andrew Stanton"
      ],
      "extra": {
        "com.synology.TMDBExample": {
          "rating": {
            "com.synology.TMDBExample": 7.9
          },
          "poster": [
            "https://image.tmdb.org/t/p/w500/uXDfjJbdP4ijW5hWSBrPrlKpxab.jpg"
          ],
          "backdrop": [
            "https://image.tmdb.org/t/p/original/3Rfvhy1Nl6sSGJwyjb0QiZzZYlB.jpg"
          ]
        }
      }
    }
  ]
}
```

### 4.1.2 电视剧

电视剧的搜索结果将包含以下属性：

```json
{
  "success": true,
  "result": [
    {
      "title": "Elementary",
      "original_available": "2012-09-27",
      "original_title": "Elementary",
      "summary": "A modern-day drama about a crime-solving duo that cracks the NYPD's most impossible cases. Following his fall from grace in London and a stint in rehab, eccentric Sherlock escapes to Manhattan where his wealthy father forces him to live with his worst nightmare - a sober companion, Dr. Watson.",
      "extra": {
        "com.synology.TMDBExample": {
          "poster": [
            "https://image.tmdb.org/t/p/w500/q9dObe29W4bDpgzUfOOH3ZnzDbR.jpg"
          ],
          "backdrop": [
            "https://image.tmdb.org/t/p/original/7sJrNKwzyJWnFPFpDL9wnZ859LZ.jpg"
          ]
        }
      }
    }
  ]
}
```

### 4.1.3 电视剧的集信息

电视节目特定剧集的搜索结果将包括以下属性：

```json
{
  "success": true,
  "result": [
    {
      "title": "Elementary",
      "tagline": "Pilot",
      "original_available": "2012-09-27",
      "summary": "Detective Sherlock Holmes, along with his sober companion, Dr. Joan Watson, uses his uncanny ability to read people and analyze crimes to assist the NYPD on some of their more difficult cases.",
      "certificate": "TV-14",
      "genre": [
        "Drama",
        "Mystery",
        "Crime"
      ],
      "actor": [
        "Jonny Lee Miller",
        "Lucy Liu"
      ],
      "director": [
        "Michael Cuesta"
      ],
      "writer": [
        "Robert Doherty"
      ],
      "season": 1,
      "episode": 1,
      "extra": {
        "com.synology.TheMovieDb": {
          "tvshow": {
            "title": "Elementary",
            "original_available": "2012-09-27",
            "original_title": "Elementary",
            "summary": "A modern-day drama about a crime-solving duo that cracks the NYPD's most impossible cases. Following his fall from grace in London and a stint in rehab, eccentric Sherlock escapes to Manhattan where his wealthy father forces him to live with his worst nightmare - a sober companion, Dr. Watson.",
            "extra": {
              "com.synology.TMDBExample": {
                "poster": [
                  "https://image.tmdb.org/t/p/w500/q9dObe29W4bDpgzUfOOH3ZnzDbR.jpg"
                ],
                "backdrop": [
                  "https://image.tmdb.org/t/p/original/7sJrNKwzyJWnFPFpDL9wnZ859LZ.jpg"
                ]
              }
            }
          },
          "poster": [
            "https://image.tmdb.org/t/p/w500/14PEsYWrZGYaQONPxQHaHjqufK5.jpg"
          ],
          "rating": {
            "com.synology.TheMovieDb": 7.4
          }
        }
      }
    }
  ]
}
```

## 4.2 错误响应

当 API 请求由于某些搜索错误而失败时，响应将包含键 "success"，值为 "false"，错误代码指示导致失败的条件

```json
{"success":false,"error_code":1003}
```

根据发生的错误类型，显示以下状态代码


| 错误码 |                                   描述                                    |
| :----: | :-----------------------------------------------------------------------: |
|  1003  | 该代码表示搜索失败。 例如，插件可能无法连接到源网站或无法检索请求的数据。 |
|  1004  |    该代码表示无法解析响应结果。 这可能是由于缺少必填信息或其他意外错误    |


# 5. 打包你的插件

应与文件夹名称和您的插件 ID 相同。 Video Station 将提取带有 tar 的 .tar 和带有 7z 的 .zip

```sh
System# tar --no-xattrs -cvf com.synology.TMDBExample.tar com.synology.TMDBExample
System# 7z a com.synology.TMDBExample.zip com.synology.TMDBExample
```

# 6. 测试你的插件

在开发阶段，您可以使用插件测试器检查您的插件是否有效

## 6.1 快速测试

此方法允许您测试插件的搜索功能。 测试你的插件功能齐全，请参阅完整测试部分。

1. 在插件测试器所在的位置安装 Video Station。

2. 使用 File Station 将插件文件上传到 Synology NAS。

3. 使用 nobody 权限输入以下命令来测试您的插件。

```sh
sudo -u nobody  /var/packages/VideoStation/target/plugins/syno_plugin_tester/loader.sh --type movie --lang enu --input "{\"title\":\"{movie title}\", \"original_available\":\"2001-11-16\"}" --limit 1 --path "${entry file path of your search plugin}" --pluginid $your plugin id}
```

4. 如果插件验证通过，会收到成功响应：`{"success": true}`


5. 否则，`success` 的值为 `false`，并带有错误代码指示导致故障的条件

```json
{"success": false, "error_code": 1004, "msg": "execute plugin fail"}
```

**下面是插件测试器的参数说明**

|   参数   |    类型     |                                                                           描述                                                                           | 是否强制 |
| :------: | :---------: | :------------------------------------------------------------------------------------------------------------------------------------------------------: | :------: |
|  input   | JSON object | input 字段必须是 JSON object 并且包含下列内容: title (必填) 和 original_available (可选).  如果搜索的是 tvshow_episode, input 必须包含 episode 和 season |   必须   |
|   lang   |   string    |          语言选项必须是下列中的一种 : chs, cht, csy, dan, enu, fre, ger, hun, ita, jpn, krn, nld, nor, plk, ptb, ptg, rus, spn, sve, trk, tha.           |   必须   |
|   type   |   string    |                                             查询视频的类型，必须是下列的一种: movie, tvshow, tvshow_episode.                                             |   必须   |
|  limit   |     int     |                                                                 插件返回的最大查询结果数                                                                 |   必须   |
|   path   |   string    |                                                                    入口文件的绝对路径                                                                    |   必须   |
| pluginid |   string    |                                                                          插件ID                                                                          |   必须   |

## 6.2 完整测试

完整测试你的插件， 你可以上传到 Video Station 使用 Chrome DevTools 来检查你插件的状态
