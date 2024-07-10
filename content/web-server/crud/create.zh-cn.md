---
title: 创建消息
type: docs
weight: 2
---

在搭建好控制器的框架后，让我们来为它添加一些内容。首先，添加一个在数据库中创建一条消息的简单逻辑。

想要和数据库交互，我们可以使用 `GoFrame` 提供的 `g.DB` 方法来创建模型或运行一些 sql。也可以使用之前自动生成的数据访问对象 `dao` 来满足我们的需求。

{{< callout type="warning" >}}
由 `g.DB` 和 `dao` 创建出的模型是不同的。`g.DB` 创建出的模型并非 `链式安全`，原始模型会被链式调用所影响。而 `dao` 创建出的模型是 `链式安全` 的，你可以多次使用它而不会改变原始模型。在具体使用时请注意这一差别。
{{< /callout>}}

## 保存数据

在我们之前生成的控制器文件中：

{{< tabs items="dao,g.DB" >}}
{{< tab >}}
```go {filename="internal/controller/message/message_v1_create_message.go",hl_lines=[2]}
func (c *ControllerV1) CreateMessage(ctx context.Context, req *v1.CreateMessageReq) (res *v1.CreateMessageRes, err error) {
	dao.Messages.Ctx(ctx).Save(req)
	return
}
```
{{< /tab >}}
{{< tab >}}
```go {filename="internal/controller/message/message_v1_create_message.go",hl_lines=[2]}
func (c *ControllerV1) CreateMessage(ctx context.Context, req *v1.CreateMessageReq) (res *v1.CreateMessageRes, err error) {
	g.DB().Model("messages").Save(req)
	return
}
```
{{< /tab >}}
{{< /tabs >}}

这里使用的 `dao` 和 `g.DB` 功能类似，只是为了创建模型并保存一些数据。你可以在[这里](https://pkg.go.dev/github.com/gogf/gf/v2/database/gdb#Model)找到 `Model` 的所有可用方法。

{{< callout type="warning" >}}
大多数情况下可以直接使用 `Save()` 方法向数据库中插入数据。但对于某些数据库，例如 `clickhouse`， `Save()` 方法是无法使用的，不过你可以使用 `Insert()` 来替代。在[这里](https://goframe.org/pages/viewpage.action?pageId=1114699)查看不同数据库支持的不同方法。
{{< /callout>}}

## 完善响应

然后我们可以在响应中添加一些信息。

```go {filename="internal/controller/message/message_v1_create_message.go"}
func (c *ControllerV1) CreateMessage(ctx context.Context, req *v1.CreateMessageReq) (res *v1.CreateMessageRes, err error) {
	dao.Messages.Ctx(ctx).Data(req).Save()
	res = &v1.CreateMessageRes{
		Code:    0,
		Data:    nil,
		Message: "Success",
	}
	return
}
```

看起来逻辑已经完整了，但实际上我们还缺少一些错误处理。`Save` 方法在发生问题时会返回一个错误。所以，让我们在发生错误时添加一些额外的信息吧。

{{< tabs items="dao,g.DB" >}}
{{< tab >}}
```go {filename="internal/controller/message/message_v1_create_message.go",hl_lines=[2]}
func (c *ControllerV1) CreateMessage(ctx context.Context, req *v1.CreateMessageReq) (res *v1.CreateMessageRes, err error) {
	_, err = dao.Messages.Ctx(ctx).Data(req).Save()
	res = &v1.CreateMessageRes{
		Code:    0,
		Data:    nil,
		Message: "Success",
	}
	if err != nil {
		res.Code = 1
		res.Message = "Database error happened"
	}
	return
}
```
{{< /tab >}}
{{< tab >}}
```go {filename="internal/controller/message/message_v1_create_message.go",hl_lines=[2]}
func (c *ControllerV1) CreateMessage(ctx context.Context, req *v1.CreateMessageReq) (res *v1.CreateMessageRes, err error) {
	_, err = g.DB().Model("messages").Save(req)
	res = &v1.CreateMessageRes{
		Code:    0,
		Data:    nil,
		Message: "Success",
	}
	if err != nil {
		res.Code = 1
		res.Message = "Database error happened"
	}
	return
}
```
{{< /tab >}}
{{< /tabs >}}

## 测试

现在你已经成功地完成了创建消息的功能，让我们来试一试它。

{{< tabs items="Postman,curl" >}}
{{< tab >}}
POST `http://localhost:8000/messages`

```json
{
  "user_uid": "0000000000",
  "content": "This is my first message."
}
```
{{< /tab >}}
{{< tab >}}
```bash
curl -X POST -H "Content-Type: application/json" -d '{"user_uid":"0000000000","content":"This is my first message."}' "http://localhost:8000/messages"
```
{{< /tab >}}
{{< /tabs >}}

你将会看到调用它的响应，似乎调用这个 API 本身没有问题，但是响应看起来非常奇怪：

```json
{
	"code": 0,
	"message": "",
	"data": {
		"code": 0,
		"message": "Success",
		"data": null
	}
}
```

别担心，还记得之前我们是怎么注册路由的吗？

```go {filename="internal/cmd/cmd.go",hl_lines=[2]}
s.Group("your route", func(group *ghttp.RouterGroup) {
	group.Middleware(ghttp.MiddlewareHandlerResponse)
	group.Bind(
		message.NewV1(),
	)
})
```

在注册路由时，我们添加了一个中间件 `ghttp.MiddlewareHandlerResponse` 来处理响应。它包含了一些逻辑来使用我们的数据生成最终的响应，进而导致了这一情况。

在下一章中，让我们创建自己的中间件来处理响应，并为我们的路由添加一些额外的功能。
