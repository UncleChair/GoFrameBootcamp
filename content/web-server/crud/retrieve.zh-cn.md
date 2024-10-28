---
title: 获取消息
type: docs
weight: 4
---

你已经可以通过请求 `create message` API 来向数据库中插入记录了。不过我们还需要一些能在响应中返回消息的方法。让我们按照 `RESTful` 的标准实现它。

## 获取一条记录

### 创建 API 结构体

首先，让我们添加一个通过 `id` 来获取消息的 API：

```go {filename="api/message/v1/show.go"}
package v1

import "github.com/gogf/gf/v2/frame/g"

type Message struct {
	Id      int    `json:"id"`
	UserUid string `json:"user_uid" des:"消息发送人ID" eg:"0000000000"`
	Content string `json:"content" des:"消息内容" eg:"This is my first message."`
}

type GetMessageReq struct {
	g.Meta `path:"/:id" tags:"Message" method:"get" sum:"获取一条消息"`
}

type GetMessageRes struct {
	g.Meta  `mime:"application/json"`
	Code    int      `json:"code" v:"required" des:"状态码" eg:"0"`
	Message string   `json:"message" v:"required" des:"状态消息" eg:"Success"`
	Data    *Message `json:"data"`
}
```

{{< callout type="info" >}}
由于 Go 的 `internal` 文件不能在它的外部使用，我们在这里定义了一个 `Message` 结构体来表示消息的信息。这在生成 OpenAPI 文档时非常方便，但却会增加项目复杂度。你也可以使用 `interface{}` 类型的 `Data` 字段，并在控制器中填入相应的数据，但是这样 OpenAPI 的文档将会缺少详细的返回数据结构。
{{< /callout >}}

### 添加控制器 logic

使用 `gf gen ctrl` 命令生成控制器，[这里](http://localhost:8000/swagger#tag/Message/paths/~1messages~1:id/get)是它的文档页面。

在控制器中添加获取一条消息的逻辑：
```go {filename="internal\controller\message\message_v1_get_message.go"}
func (c *ControllerV1) GetMessage(ctx context.Context, req *v1.GetMessageReq) (res *v1.GetMessageRes, err error) {
	id := g.RequestFromCtx(ctx).GetRouter("id").Int()
	g.Dump(id)
	var message *v1.Message
	dao.Messages.Ctx(ctx).Where(g.Map{
		dao.Messages.Columns().Id: id,
	}).Scan(&message)
	res = &v1.GetMessageRes{
		Code:    0,
		Data:    nil,
		Message: "Success",
	}
	if message != nil {
		res.Data = message
	} else {
		res.Code = 1
		res.Message = "No message found"
	}
	return
}
```

还记得我们设置的 `get` API 路径是 `/:id` 吗？现在我们可以从路由中获取消息的id。这里使用的`g.Dump(id)` 是为了输出 id 到标准输出（命令行），它是一个 `GoFrame` 提供的便捷函数。

然后我们可以使用数据访问对象 `dao` 来根据 `id` 获取消息。这里使用的 scan 函数可以自动将查询结果转换为 API 中定义的 `Message` 结构体。你也可以使用其他方法例如 `One()` 来获取结果并将它们填入响应。

{{< callout type="info" >}}
尽管 `Scan()` 非常好用，在使用时你仍需小心。它使用了 `反射` 来转换结构体并且会消耗一定的资源。如果你需要进行复杂的结构体转换，我们推荐你自行编写 [`UnmarshalValue`](https://goframe.org/pages/viewpage.action?pageId=1114226) 来提升性能表现。
{{< /callout >}}

这里的所有响应都只会有相同的 HTTP 状态码 `200`。如果你希望在错误时返回不同的 HTTP 状态码，可以在错误处理时添加：

```go
g.RequestFromCtx(ctx).Response.Status = 404
```

它会把最终的 HTTP 状态码设置为 `404`，你也可以在响应处理中间件中做类似的错误处理。

{{< callout type="info" >}}
在 `v2.8.0` 版本之前，如果使用了不同的 HTTP 状态码，你需要手动修改 OpenAPI 文档。可以使用 `g.Server().GetOpenApi()` 来获取 OpenAPI 对象，然后在 web 服务启动后修改它。而在 `v2.8.0` 版本后，你可以使用 `status` 标签和 `ResponseStatusMap` 方法来设置不同的 HTTP 状态码。
{{< /callout >}}

### 测试

因为已经注册了 `message` 相关的路由，这里我们可以直接测试：

{{< tabs items="Postman,curl" >}}
{{< tab >}}
GET `http://localhost:8000/messages/1` 并使用你的 token
{{< /tab >}}
{{< tab >}}
```bash
curl -X GET "http://localhost:8000/messages/1?token=<your token>"
```
{{< /tab >}}
{{< /tabs >}}

你会得到如下响应：

```json
{
    "code": 0,
    "message": "Success",
    "data": {
        "id": 1,
        "user_uid": "0000000000",
        "content": "This is my first message."
    }
}
```

{{< callout type="info" >}}
如果你的 API 返回了 null，代表之前创建消息的部分存在问题。你可以回到创建消息的部分，并仔细检查哪里有误。
{{< /callout >}}

## 获取多条记录

获取多条记录也非常简单，尝试自行创建 API 并添加如下结构体到你的 API 定义中：

```go
type Messages []*Message
```

同样，你也可以使用 `Scan()` 来获取多条记录：

```go
var messages v1.Messages
dao.Messages.Ctx(ctx).Scan(&messages)
```

测试一下，你会得到类似下面的响应：

```json
{
    "code": 0,
    "message": "Success",
    "data": [
        {
            "id": 1,
            "user_uid": "0000000000",
            "content": "This is my first message."
        }
    ]
}
```

## 获取关联（实验功能）

和其他框架不同的是，`GoFrame` 并没有提供 `hasOne` 或其他类似的关联，因为框架的主旨是保持数据结构简单易用。不过获取关联也很容易，只需要使用 `GoFrame` 提供的 `with` 函数。

回到 `Message` API 的定义，让我们先添加一个 `User` 结构体：

```go {filename="api/message/v1/show.go"}
type User struct {
	g.Meta   `orm:"table:users"`
	Uid      string `json:"uid" des:"User uid" eg:"0000000000"`
	Username string `json:"username" des:"Username" eg:"admin"`
}
```

我们使用 `g.Mate` 在 `User` 结构体中添加 orm 标签，然后在消息结构体中添加关联：

```go {filename="api/message/v1/show.go"}
type Message struct {
	Id      int    `json:"id"`
	UserUid string `json:"user_uid" des:"Message sender ID" eg:"0000000000"`
	Content string `json:"content" des:"Message content" eg:"This is my first message."`
	User    *User  `orm:"with:uid=user_uid" json:"user" des:"Message sender"`
}
```

最后，在 `Model` 操作中使用关联时，你需要在方法链中添加一个 `with`：

```go {filename="internal/controller/message/message_v1_get_message.go"}
dao.Messages.Ctx(ctx).WithAll().Where(g.Map{
	dao.Messages.Columns().Id: id,
}).Scan(&message)
```

现在尝试从原来的 api 再次获取一条消息，你会在响应数据中看到附加的用户信息。

```json
{
    "code": 0,
    "message": "Success",
    "data": {
        "id": 1,
        "user_uid": "0000000000",
        "content": "This is my first message.",
        "user": {
            "uid": "0000000000",
            "username": "admin"
        }
    }
}
```

{{< callout type="info" >}}
在方法链中，`with` 还有别的能够选择特定关联的方法，你可以在[这里](https://goframe.org/pages/viewpage.action?pageId=7297190)找到它们。并且由于这是一项实验性的功能，目前只有获取数据的相关方法可以使用。
{{< /callout >}}
