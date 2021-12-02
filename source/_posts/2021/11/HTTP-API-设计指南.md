---
title: HTTP API 设计指南
date: 2021-11-16 11:37:12
tags: [API, HTTP, RESTful, 设计指南]
categories: 指南
description: HTTP API 设计指南
---


# 基础

> 基础部分概述了本指南其余部分所依据的设计原则。

## 隔离关键点

通过隔离请求和响应生命周期中不同部分的关注点，在设计时保持简单。保持简单的规则可以让更多的注意力集中在更大和更难的问题上。

请求和响应用以解决特定资源或集合。使用路径（path）来表示身份，使用正文（body）来传输内容，使用头（header）来传达元数据。 查询参数（query param）也可以在边缘情况下用作传递标头信息的手段，但首选头信息，因为它们更灵活并且可以传达更多样的信息。

## 需要安全连接

需要使用 TLS 安全连接才能访问所有的 API，没有特例。试图弄清楚或解释什么时候可以使用 TLS 以及什么时候不能使用是不值得的。一切都需要并且只需要 TLS。

理想情况下，只需通过不响应 http 或端口 80 的请求来拒绝任何非 TLS 请求，以避免任何不安全的数据交换。在无法做到这一点的环境中，请使用403 Forbidden.

不鼓励重定向，因为它们允许含混/不良客户端行为而不会提供任何明确的收益。依赖重定向的客户端会增加服务器流量并使 TLS 变得无用，因为在第一次调用期间敏感数据已经暴露。

## 在 Accepts 标头中要求版本控制

版本控制和版本之间的转换可能是设计和维护 API 的更具挑战性的一个方面。因此，最好从一开始就制定一些机制来预防这种情况。

为了防止对用户造成意外、破坏性更改，最好要求为所有请求指定一个版本。应避免使用默认版本，因为它们在未来很难更改。

最好在标头中提供版本规范以及其他元数据，使用Accept带有自定义内容类型的标头，例如：

```http
Accept: application/vnd.heroku+json; version=3
```

## 支持缓存的 ETag

在所有响应中包含一个 `ETag` 标头，标识返回资源的特定版本。 这允许用户缓存资源并使用 `If-None-Match` 标头中使用具有此值的请求来确定是否应该更新缓存。

## 为自省提供请求 ID

在每个 API 响应中包含一个 `Request-Id` 标头，填充一个 UUID 值。 通过在客户端、服务器和任何支持服务上记录这些值，它提供了一种跟踪、诊断和调试请求的机制。

## 在具有范围的请求之间划分大响应

应使用 `Range` 标头指定何时有更多数据可用以及如何检索数据，从而将大型响应拆分为多个请求。

# 请求

> 请求部分概述了API请求的模式。

## 接受请求正文中的序列化 JSON

接受序列化JSON在 `PUT/PATCH/POST` 请求机构中，代替或除了的 `form-encoded` 的数据。 这创建了与 JSON 序列化响应主体的对称性，例如：

```shell
$ curl -X POST https://service.com/apps \
    -H "Content-Type: application/json" \
    -d '{"name": "demoapp"}'

{
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "name": "demoapp",
  "owner": {
    "email": "username@example.com",
    "id": "01234567-89ab-cdef-0123-456789abcdef"
  },
  ...
}
```

## 资源名称

使用复数形式的资源名称，除非相关资源是系统内的单例（例如，系统的整体状态可能是 `/status`）。这使它在您引用特定资源的方式上保持一致。

## 操作

首选不需要特殊操作的端点配置。在需要指定操作的情况下，用 `actions` 前缀清楚地描述它们：

```http
/resources/:resource/actions/:action
```
例如停止特定的运行：

```http
/runs/{run_id}/actions/stop
```
还应尽量减少对集合的操作。在需要时，他们应该使用顶级操作描述来避免命名空间冲突并清楚地显示操作范围：

```http
/actions/:action/resources
```
例如重新启动所有服务器：

```http
/actions/restart/servers
```

## 使用一致的路径格式

### 小写路径和属性

使用小写和破折号分隔的路径名，与主机名对齐，例如：

```
service-api.com/users
service-api.com/app-setups
```
小写属性也是如此，但使用下划线分隔符，以便在 JavaScript 中输入属性名称时无需引号，例如：

```
service_class: "first"
```

### 为方便起见，支持非id间接引用

在某些情况下，最终用户可能不方便提供 ID 来标识资源。例如，用户可能会想到 Heroku 的应用程序名称，但该应用程序可能由 UUID 标识。 在这些情况下，您可能希望同时接受 id 或名称，例如：

```shell
$ curl https://service.com/apps/{app_id_or_name}
$ curl https://service.com/apps/97addcf0-c182
$ curl https://service.com/apps/www-prod
```

不要只接受名称而排除 ID。

### 最小化路径嵌套

在具有嵌套父/子资源关系的数据模型中，路径可能会深度嵌套，例如：

```
/orgs/{org_id}/apps/{app_id}/dynos/{dyno_id}
```
通过优先在根路径上定位资源来限制嵌套深度。使用嵌套来指示范围集合。例如，对于上面的 dyno 属于一个应用程序属于一个组织的情况：

```
/orgs/{org_id}
/orgs/{org_id}/apps
/apps/{app_id}
/apps/{app_id}/dynos
/dynos/{dyno_id}
```

## 响应

响应部分提供了 API 响应模式的概述。

### 返回适当的状态代码

为每个响应返回适当的 HTTP 状态代码。成功的响应应根据本指南进行编码：

- `200`：请求成功了`GET`，`POST`，`DELETE`，或 `PATCH` 调用同步完成，或 `PUT` 调用同步更新现有资源
- `201`：请求成功 `POST`，或 `PUT` 同步创建新资源的调用。提供指向新创建资源的"位置"标头也是最佳实践。 这在 `POST` 上下文中特别有用，因为新资源将具有与原始请求不同的 `URL`
- `202`：接受请求的 `POST`，`PUT`，`DELETE`，或 `PATCH` 通话将被异步处理
- `206`：请求成功 `GET`，但只返回了部分响应：见上面的范围
注意认证和授权错误码的使用：

- `401 Unauthorized`：请求失败，因为用户未通过身份验证
- `403 Forbidden`：请求失败，因为用户无权访问特定资源
出现错误时返回合适的代码以提供附加信息：

- `422 Unprocessable Entity`：您的请求已被理解，但包含无效参数
- `429 Too Many Requests`： 你被限速了，稍后重试
- `500 Internal Server Error`：服务器出现问题，检查状态站点和/或报告问题

### 在可用的情况下提供完整的资源

尽可能在响应中提供完整的资源表示（即具有所有属性的对象）。始终提供 200 和 201 响应的完整资源，包括 `PUT/PATCH` 和 `DELETE` 请求，例如：

```shell
$ curl -X DELETE \
  https://service.com/apps/1f9b/domains/0fd4

HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
...
{
  "created_at": "2012-01-01T12:00:00Z",
  "hostname": "subdomain.example.com",
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "updated_at": "2012-01-01T12:00:00Z"
}
```

202 响应将不包括完整的资源表示，例如：

```shell
$ curl -X DELETE \
  https://service.com/apps/1f9b/dynos/05bd

HTTP/1.1 202 Accepted
Content-Type: application/json;charset=utf-8
...
{}
```

### 提供资源 (UU)ID

`id` 默认给每个资源一个属性。除非您有充分的理由不使用 UUID，否则请使用 UUID。不要使用在服务实例或服务中的其他资源之间不是全局唯一的 ID，尤其是自动递增的 ID。

以小写8-4-4-4-12格式呈现 UUID ，例如：

```
"id": "01234567-89ab-cdef-0123-456789abcdef"
```

### 提供标准时间戳

提供 `created_at` 和 `updated_at` 时间戳在默认情况下，如：

```json
{
// ...
"created_at": "2012-01-01T12:00:00Z",
"updated_at": "2012-01-01T13:00:00Z",
// ...
}
```
这些时间戳对于某些资源可能没有意义，在这种情况下可以省略它们。

### 提供标准响应类型

本文档描述了每种 JSON 基本数据类型的可接受值。

#### 字符串

- 可接受的值：
  - 字符串
  - null

例如：

```json
[
  {
    "description": "very descriptive description."
  },
  {
    "description": null
  },
]
```

#### 布尔值

- 可接受的值：
  - true
  - false

例如：

```json
[
  {
    "provisioned_licenses": true
  },
  {
    "provisioned_licenses": false
  },
]

```

#### 数字

- 可接受的值：
  - 数字
  - null
  
注意：一些 JSON 解析器会将精度超过 15 位小数的数字作为字符串返回。 如果您需要大于 15 位小数的精度，请始终为该值返回一个字符串。如果不是，请将这些字符串转换为数字，以便 API 的使用者始终知道期望的值类型。

例如：

```json
[
  {
    "average": 27.123
  },
  {
    "average": 12.123456789012
  },
]
```

#### 数组

- 可接受的值：
  - 数组

注意：返回空数组而不是NULL当数组中没有值时。

例如：

```json
[
  {
    "child_ids": [1, 2, 3, 4],
  },
  {
    "child_ids": [],
  }
]
```

#### 对象

- 可接受的值：
  - 对象
  -null

例如：

```json
[
  {
    "name": "service-production",
    "owner": {
      "id": "5d8201b0..."
     }
  },
  {
    "name": "service-staging",
    "owner": null
  }
]
```

### 使用 ISO8601 格式的 UTC 时间

仅接受和返回 UTC 时间。ISO8601 格式的渲染时间，例如：

```
"finished_at": "2012-01-01T12:00:00Z"
```

### 嵌套外键关系

使用嵌套对象序列化外键引用，例如：

```json
{
  "name": "service-production",
  "owner": {
    "id": "5d8201b0..."
  },
  // ...
}
```

而不是例如：

```json
{
  "name": "service-production",
  "owner_id": "5d8201b0...",
  // ...
}
```

这种方法可以内联有关相关资源的更多信息，而无需更改响应的结构或引入更多顶级响应字段，例如：

```json
{
  "name": "service-production",
  "owner": {
    "id": "5d8201b0...",
    "email": "alice@heroku.com"
  },
  // ...
}
```

嵌套外键关系时，请使用完整记录或仅使用外键。提供字段的子集可能会导致意外和混乱，使不同操作和端点之间更有可能出现不一致。

为避免不一致和混淆，请序列化：

- 仅外键 - 可以查找完整记录的值，例如`id`，`slug`，`email`。
- 完整记录, 所有字段（这将是"嵌入记录"）

### 生成结构化错误

针对错误生成一致、结构化的响应主体。包括一个机器可读的错误 `id`，一个人类可读的错误 `message`，以及可选的url指向客户端关于错误和如何解决它的更多信息，例如：

```
HTTP/1.1 429 Too Many Requests
```

```json
{
  "id":      "rate_limit",
  "message": "Account reached its API rate limit.",
  "url":     "https://docs.service.com/rate-limits"
}
```

记录您的错误格式和 `id` 客户可能遇到的可能错误。

### 显示速率限制状态

来自客户端的速率限制请求，以保护服务的健康并为其他客户端保持高服务质量。 您可以使用 [令牌桶算法](http://en.wikipedia.org/wiki/Token_bucket) 来量化请求限制。

返回 `RateLimit-Remaining` 响应标头中每个请求的剩余请求令牌数。

### 在所有响应中保持 JSON 缩小

额外的空白为请求增加了不必要的响应大小，许多供人类消费的客户端会自动“美化”JSON 输出。最好保持 JSON 响应最小化，例如：

```json
{"beta":false,"email":"alice@heroku.com","id":"01234567-89ab-cdef-0123-456789abcdef","last_login":"2012-01-01T12:00:00Z","created_at":"2012-01-01T12:00:00Z","updated_at":"2012-01-01T12:00:00Z"}
```

而不是例如：

```json
{
  "beta": false,
  "email": "alice@heroku.com",
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "last_login": "2012-01-01T12:00:00Z",
  "created_at": "2012-01-01T12:00:00Z",
  "updated_at": "2012-01-01T12:00:00Z"
}
```

您可以考虑通过查询参数（例如 `?pretty=true`）或通过 `Accept` 标头参数（例如 `Accept: application/vnd.heroku+json; version=3; indent=4;`）为客户端提供一种可选的方式来检索更详细的响应。

# 工件

工件部分描述了我们用来管理和讨论 API 设计和模式的物理对象。

## 提供机器可读的 JSON 模式

提供机器可读的模式来准确指定您的 API。使用 [prmd](https://github.com/interagent/prmd) 来管理您的架构，并确保它使用了 `prmd verify` 验证。

## 提供人类可读的文档

提供人类可读的文档，客户开发人员可以使用这些文档来理解您的 API。

如果您如上所述使用 prmd 创建架构，您可以使用 `prmd doc` 轻松地为所有端点生成 Markdown 文档。

除了端点详细信息外，还提供包含以下信息的 API 概述：

- 身份验证，包括获取和使用身份验证令牌。
- API 稳定性和版本控制，包括如何选择所需的 API 版本。
- 常见的请求和响应头。
- 错误的序列化格式。
- 将 API 与不同语言的客户端一起使用的示例。

## 提供可执行示例

提供可执行示例，用户可以直接在其终端中键入以查看 API 调用。在最大程度上，这些示例应该逐字可用，以最大限度地减少用户尝试 API 所需的工作量，例如：

```shell
$ export TOKEN=... # acquire from dashboard
$ curl -is https://$TOKEN@service.com/users
```

## 描述稳定性

根据 API 的成熟度和稳定性来描述您的 API 或其各种端点的稳定性，例如使用原型/开发/生产标志。

一旦您的 API 被声明为生产就绪且稳定，请不要在该 API 版本中进行向后不兼容的更改。如果您需要进行向后不兼容的更改，请创建一个版本号递增的新 API。