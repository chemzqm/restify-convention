# Restify API 基本规范

## http 头部

因为我们目前只需要 json 格式的请求和响应，所以客户端会携带 http 请求头：`Accept: application/json`,
后端返回时应携带 http 响应头：`Content-type: application/json`, 方便前端解析响应。

所以非 http 规范的请求头部如非特别需要，不应该出现在 http 请求头中，服务端也不应该在非特别需要情况下
在 http 响应头部中添加非标准的字段。


## 响应

* 响应状态代码应遵守 http 规范，只在错误时返回 200~300 的响应码, 一些常用的 http 响应码如下：

    * 200 "ok"
    * 201 "created"
    * 202 "accepted"
    * 301 "moved permanently"
    * 302 "moved temporarily"
    * 400 "bad request"
    * 401 "unauthorized"
    * 403 "forbidden"
    * 404 "not found"
    * 405 "method not allowed"
    * 406 "not acceptable"
    * 408 "request time-out"
    * 409 "conflict"
    * 413 "request entity too large"
    * 414 "request-uri too large"
    * 500 "internal server error"
    * 501 "not implemented"

* 返回字段应为数字时应在 json 中以数字表示
* 返回字段应为布尔时应在 json 响应中返回 boolean 的 `true` 和 `false`， 而不是字符串 `"true"` 和 `"false"`
* 返回字段应为日期时应返回 ISO 标准格式日期字符串，例如： `2016-03-31T08:40:45.954Z`,
* 返回字段如果是空数组应返回空的数组格式，例如: `{"orders": []}`, 而不是 `{"orders": null}` 或者其他形式
* 返回字段如果是空字符串应返回空的字符串，例如: `{"note": ""}`, 而不是 `{"note": null}` 或者其他形式
* 返回字段名称如没有特别需要，应与后端 model 中使用的名称完全一致, 如果无法一致，必须在文档着重表明

* 标准错误格式，使用统一的错误格式，方便客户端统一解析，目前格式为 `{"error": "错误信息"}`, 如果同时有多条错误信息使用换行符进行
  分隔，例如： `{"error": "错误1\n错误2"}`

## URL 约定

统一资源的不同 CRUD 请求应符合一致的约定，以 order 为例：

* 列出所有 order 的请求： `GET /orders`
* 列出一个 order 的请求： `GET /orders/:id`
* 删除一个 order 的请求： `DELETE /orders/:id`
* 更新 order 全部属性请求： `PUT /orders/:id`
* 修改 order 状态的请求： `PATCH /orders/:id`

实体名称不考虑英文 es 做为复数表示的形式，例如 `glass` 对应 url 为 `/glasss`

## GET 请求

* 参数： 所有参数通过 encode 之后的 querystring 传递
  请求格式举例： `/orders?user_id=5` `/orders/5`

* 响应： GET 响应有两种情况一种是资源列表，一种是单个资源。 
  例如： `/orders` 返回全部订单，响应内应包含 `orders` 做为最底层的数组属性，

    `{"orders":[{"id": 1, "id" : 2}]}`

  _同 url 请求，实体名称不考虑 `es` 做为复数的形式, 统一为后面加 `s`_

  如果 orders 列表为空应返回 `{"orders": []}` 做为响应，此时响应码还应该是 200，客户端对 orders 字段进行判定是否返回的空列表。

  如果是返回单个资源的响应， 例如： `/orders/5`, 响应内应包含 `order` 做为底层嵌套属性

    `{"order": {"id": 1}}`

## POST 请求

POST 请求仅做为资源创建请求使用。

* 参数类型：原则上所有参数通过请求 body 传递。 除非文件上传相关的请求，body 类型只可能是 `application/json` 或者 `application/x-www-form-urlencoded`
  客户端应在 http 请求头 `Content-type` 中表明请求 body 类型， 服务端应根据类型解析出对应 model

* 参数格式：请求时资源名做为底层属性，例如 json 请求：`{"order": {"user_id": 1}}`
* 响应：成功时响应中应包含 `id` 做为资源名的子属性，例如：`{"order": {"id": 2}}`, 其它无关字段不应当返回 (方便调试，节省带宽)。

## DELETE 请求

DELETE 请求仅做为资源删除时使用。

* 参数：原则上 DELETE 请求不应携带出 auth_token 之外的其它参数
* url： 删除资源 id 因该 url pathname 最后体现，例如： `GET /api/v2/space_designers/:id?auth_token=xxxx`
* 响应：成功时返回 200 状态，以及空的 body，失败时依照 http 响应规范返回状态码以及符合标准错误格式的错误信息

## PATCH 请求

PATCH 仅在修改资源部分属性时使用。

* 参数：与 POST 相同
* url： 修改资源 id 应在 url pathname 中体现
* 响应：同 DELETE

## PUT 请求

PUT 仅在修改资源多个属性时使用, 其它行为与 PATCH 相同
